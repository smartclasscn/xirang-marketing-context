# DBPrisma 运维手册

**版本**: v1.2 | **日期**: 2026-06-10 | **适用版本**: DBPrisma ≥ 1.0  
**受众**: 系统运维工程师、值班 SRE、NOC 工程师

---

## ⚠️ CLI 调用模型（重要，请先阅读）

DBPrisma 的命令分为两类，混淆将导致操作无效甚至误操作：

| 类型 | 可直接在 Shell 中单次运行的命令 |
|------|---------------------------------|
| **独立子命令**（Shell 直接执行） | `start`、`checkconfig`（别名 `cc`）、`setconfig`（别名 `sc`）、`service`、`--help`、`--version`；其中 `checkconfig`/`setconfig`/`service` 必须为第一个参数（位于选项之前），`start` 选项可前可后 |
| **交互会话命令**（仅限 XR:> 提示符内） | `status`、`stop`、`reload`、`restart`、`shutdown`、`backends`、`conns`、`kick`、`drain`、`pause`、`resume`、`flush`、`capture`、`display`、`loglevel`、`history`、`useradd`、`userdel`、`userlist`、`setpwd`、`token rotate` 等所有其他命令 |

**关键说明**：
- 运行 `./DBPrisma status` 会报 `unknown command 'status'`（`status` 既非子命令、又非存在的配置文件），**不会**显示状态。
- 运行 `./DBPrisma stop` 同理报 `unknown command 'stop'`，**不会**停止服务。
- 所有操作命令必须在 **XR:> 交互会话**中执行（通过 `./DBPrisma` 启动交互模式；守护进程模式请用 `./DBPrisma start --no-interactive`，注意必须带 `start`，否则代理不会启动且进程随即退出）。
- 本手册中的 `XR:> <cmd>` 格式表示在交互提示符下输入的命令；`./DBPrisma <cmd>` 格式仅用于独立子命令。
- `DBPrisma stop` 和 `DBPrisma reload` 也**不是**有效的独立 Shell 命令，不能在 systemd `ExecStop=`/`ExecReload=` 中使用。

---

## 目录

1. [日常检查清单](#1-日常检查清单)
2. [启动、停止、重启操作](#2-启动停止重启操作)
3. [热更配置（不停服）](#3-热更配置不停服)
4. [连接管理操作](#4-连接管理操作)
5. [监控与状态查询](#5-监控与状态查询)
6. [日志管理](#6-日志管理)
7. [后端数据库运维协同](#7-后端数据库运维协同)
8. [故障排查手册](#8-故障排查手册)
9. [紧急操作手册（SOE）](#9-紧急操作手册soe)
10. [变更操作规程](#10-变更操作规程)
11. [监控告警配置参考](#11-监控告警配置参考)
12. [运维命令速查表](#12-运维命令速查表)
13. [授权与 License 管理](#13-授权与-license-管理)

---

## 1. 日常检查清单

### 1.1 每日检查（建议每天 9:00 执行）

```bash
# 4. 检查日志中是否有 ERROR/WARN
grep -E "^.{23}(ERROR|WARN)" logs/dbprisma.log | tail -50

# 5. 检查 Admin HTTP 可达性
curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health | python -m json.tool

# 6. 检查磁盘空间（日志磁盘）
df -h /opt/dbprisma/logs
```

以下命令须在 **XR:> 会话**中执行（启动 `./DBPrisma` 后输入）：

```
# 1. 确认代理运行状态
XR:> status

# 2. 查看后端健康状态
XR:> backends

# 3. 检查活跃连接数是否异常
XR:> conns
```

### 1.2 每周检查

- 检查日志文件轮转是否正常（`logs/` 目录文件数和大小）
- 检查 `backups/` 配置备份文件数量，清理过期备份
- 复查 Prometheus `/metrics` 中 `connection_errors_total` 趋势
- 验证 TLS 证书有效期（距到期 < 30 天需提前轮换）

### 1.3 每月检查

- 轮换 Admin HTTP Bearer Token：在 XR:> 会话中执行 `token rotate`
- 核查代理用户列表是否有废弃账号：在 XR:> 会话中执行 `userlist`
- 核查 IP 黑白名单是否需要更新
- 对照 `docs/user/configuration-reference.md` 核查运行配置

---

## 2. 启动、停止、重启操作

### 2.1 正常启动

```bash
# 前台启动（进入 XR:> 提示符）
./DBPrisma

# 守护进程模式（无交互式提示符；SIGTERM 或 XR:> stop 命令退出）
./DBPrisma start --no-interactive --config config/conf.json

# systemd（推荐生产部署）
systemctl start dbprisma

# Windows 服务
.\DBPrisma.exe service start
# 或
Start-Service DBPrisma
```

> **授权前置**：启动时会先经过 License 闸门，控制台/日志会打印 `[license] ...` 一行：
> - `[license] licensed to ...` 或 `[license] TRIAL - ...` → 放行，继续启动；
> - `[license] Startup blocked: ...` + `Startup aborted by license gate.` → **不会启动**。
> 详见 [§13 授权与 License 管理](#13-授权与-license-管理)。

**启动后验证**（在 XR:> 会话中执行）：
```
XR:> status          # 期望输出: RUNNING
XR:> backends        # 期望后端状态: UP
```

或通过 Admin HTTP API：
```bash
curl http://127.0.0.1:8080/health -H "Authorization: Bearer $TOKEN"
```

### 2.2 优雅停止

在 XR:> 会话中执行：
```
# 优雅停止（等待连接处理完，最长 graceful_drain_ms 毫秒）
XR:> stop
```

或通过 OS 服务管理器：
```bash
# systemd（发送 SIGTERM，触发优雅关闭）
systemctl stop dbprisma

# Windows 服务
.\DBPrisma.exe service stop
```

优雅停止流程：
1. 停止接受新连接
2. 等待现有连接完成当前事务（最长 `txn_drain_ms` ms）
3. 强制关闭剩余连接（最长 `stop_deadline_ms` ms）
4. 清理资源退出

### 2.3 优雅重启（零停机滚动重启）

在 XR:> 会话中执行：
```
# 方式1：直接重启命令（stop + 预检 + 预热启动）
XR:> restart
```

> **注意**：`systemctl reload dbprisma` **无效**——代理不响应 SIGHUP 作为重新加载信号（systemd 单元不含 `ExecReload=`）。热更配置请用 `XR:> reload`，进程级重启请用 `XR:> restart` 或 systemctl restart。

配合负载均衡的滚动重启（在 XR:> 会话中）：
```
# 步骤1：标记指定后端为排水（停止接受新连接）
XR:> drain <server_id>

# 步骤2：等待 conns 降为 0 或业务低峰后停止
XR:> stop
```
```bash
# 步骤3：替换可执行文件或修改配置，然后重新启动
./DBPrisma start --no-interactive --config config/conf.json
```

### 2.4 强制停止（紧急）

在 XR:> 会话中执行：
```
# 优雅停止（带倒计时，默认等待现有连接完成事务）
XR:> shutdown

# 立即强制停止（跳过排水等待）
XR:> shutdown 0 force
```

```bash
# 或直接 kill（最后手段）
kill -15 $(cat /var/run/dbprisma.pid)
```

> **注意**：`shutdown` 默认是**优雅停止**（等待连接排空，受 `graceful_drain_ms` / `stop_deadline_ms` 控制）。`shutdown 0 force` 才是立即强停，可能导致正在执行的事务回滚，生产环境仅在紧急情况使用。

---

## 3. 热更配置（不停服）

### 3.1 单项配置热更

在 XR:> 会话中执行：
```
# 运行时动态修改日志级别（立即生效，无需停服）
XR:> loglevel DEBUG
XR:> loglevel INFO     # 恢复
```

修改安全配置（`security.*` 不支持热更，须停服 → 修改文件 → 重启）：
```
# 步骤1：在 XR:> 会话中停止服务
XR:> stop
```
```bash
# 步骤2：手动编辑 conf.json 的 ip_denylist / sql_firewall_enabled 等字段
# 或使用交互式向导（会校验配置）：
./DBPrisma setconfig security
# 步骤3：重新启动
./DBPrisma start --no-interactive --config config/conf.json
```

### 3.2 整体配置热更

```bash
# 步骤1：备份当前配置
cp conf/conf.json conf/conf.json.manual-$(date +%Y%m%d-%H%M%S)

# 步骤2：修改 conf.json

# 步骤3：验证配置（独立子命令，可直接在 Shell 中运行）
./DBPrisma checkconfig
```

在 XR:> 会话中执行：
```
# 步骤4：热更（仅对可热更字段生效，不可热更字段须 stop → start）
XR:> reload
```

**热更后验证**：
```bash
# 查看当前磁盘配置（独立子命令）
./DBPrisma setconfig show logging
./DBPrisma setconfig show proxy
```

在 XR:> 会话中执行：
```
# 查看运行时状态
XR:> status
```

### 3.3 热更注意事项

- 修改 `proxy.worker_threads`、`servers[].listen_port`、`tls.*` 等字段需要重启才能生效
- `reload` 命令会跳过需要重启的字段（显示 `SKIP: RESTART_REQUIRED`），不会报错
- 若 `config_manage.rollback_on_reload_fail = true`，热更失败会自动回滚

---

## 4. 连接管理操作

### 4.1 查看连接详情

在 XR:> 会话中执行：
```
# 显示所有活跃连接
XR:> conns

# 仅显示某客户端 IP 的连接（可选过滤）
XR:> conns 192.168.1.100

# 输出示例
# ID    CLIENT_IP       USER        DB       STATE     DURATION
# 1001  192.168.1.100   appuser     mydb     EXECUTING  0.5s
# 1002  192.168.1.101   readuser    mydb     IDLE        120s
```

### 4.2 踢出问题连接

`kick` 命令可按客户端 IP，或按 `conns` 列表行号强制断开连接。在 XR:> 会话中执行：
```
# 断开指定 IP 的所有连接（port=0 表示所有端口）
XR:> kick 192.168.1.100

# 断开指定 IP 的指定端口
XR:> kick 192.168.1.100 54321

# 按 conns 列表的第 N 行（# 列）断开单条连接
XR:> conns          # 先查看，记下目标行的 # 序号
XR:> kick id 3      # 断开第 3 行连接（实时解析为其 IP:端口）
```

> **注意**：`kick` 不接受 `--user` / `--ip` 标志。参数为客户端 IP（可选端口），或 `id <n>`。`id <n>` 的 N 取自**最近一次** `conns` 输出；列表变动后需重新 `conns` 确认。

### 4.3 暂停/恢复接受连接

在 XR:> 会话中执行：
```
# 暂停接受新连接（不断开现有连接）
XR:> pause
# 场景：后端维护前暂停，避免新连接建立

# 恢复接受新连接
XR:> resume
```

### 4.4 排水（Drain）操作

在 XR:> 会话中执行：
```
# 标记指定后端为排水（停止向其路由新连接，现有连接自然结束）
XR:> drain <server_id>
# 场景：滚动升级、计划维护；server_id 为 servers[].id 的值

# 取消排水，恢复接受新连接
XR:> drain cancel <server_id>
```

```bash
# 监控 drain 进度（通过 Admin HTTP）
watch -n 2 'curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health | python -m json.tool'
```

---

## 5. 监控与状态查询

### 5.1 代理整体状态

在 XR:> 会话中执行：
```
XR:> status

# 期望输出示例
# Status:       RUNNING
# Uptime:       3d 2h 15m
# Connections:  Active=42  Total=152340
# Workers:      8 threads
# Memory:       RSS=128MB
```

### 5.2 后端健康状态

在 XR:> 会话中执行：
```
XR:> backends

# 输出示例
# ID         HOST              PORT   STATUS  WEIGHT  POOL  LATENCY
# primary    mysql-primary     3306   UP      100     10/50  1.2ms
# replica1   mysql-replica1    3306   UP      50      5/20   2.1ms
# replica2   mysql-replica2    3306   DOWN    50      0/0    -
```

后端状态说明：
- `UP`：健康，正常接受请求
- `DOWN`：不可用，探测失败超过阈值
- `HALF_OPEN`：熔断恢复探测中
- `DISABLED`：管理员手动禁用（`servers[].enabled = false`）

### 5.3 Admin HTTP 指标

```bash
# 获取 Prometheus 格式指标
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/metrics

# 关键指标
# dbprisma_connections_active         当前活跃连接数
# dbprisma_connections_total          历史总连接数
# dbprisma_backend_status{id="..."}   后端状态（1=UP, 0=DOWN）
# dbprisma_backend_latency_ms{id="..."} 后端延迟
# dbprisma_query_duration_seconds     查询耗时分布
# dbprisma_errors_total               错误总计数
# dbprisma_cb_state{id="..."}         熔断器状态
```

### 5.4 状态一次性显示

在 XR:> 会话中执行：
```
# 一次性读取 conf.json 和代理用户列表并显示（非持续刷新）
XR:> display
```

> **注意**：`display` 是**一次性读取**磁盘上的 `conf.json` + 代理用户列表并输出，不会持续刷新。

---

## 6. 日志管理

### 6.1 日志级别

| 级别 | 适用场景 |
|------|----------|
| `DEBUG` | 开发调试、问题定位（**勿在生产长期开启，性能影响大**） |
| `INFO` | 正常生产运行（推荐） |
| `WARN` | 需关注的异常，但不影响服务 |
| `ERROR` | 影响服务的错误，需立即处理 |

在 XR:> 会话中执行：
```
# 在线切换日志级别（立即生效，无需重启）
XR:> loglevel DEBUG
XR:> loglevel INFO
XR:> loglevel WARN
```

### 6.2 日志文件管理

配置参考：
```json
"logging": {
  "file": "logs/dbprisma.log",
  "level": "INFO",
  "max_size_mb": 100,
  "rotate_count": 7
}
```

- `max_size_mb`：单个日志文件最大大小，超过后轮转
- `rotate_count`：保留的历史日志文件数量

在 XR:> 会话中执行：
```
# 刷新后端预热连接池（重建 warmup pool；省略 server_id 则刷新全部）
XR:> flush [server_id]
```

```bash
# 查看最新日志
tail -f logs/dbprisma.log

# 过滤错误日志
grep "ERROR" logs/dbprisma.log | tail -100

# 查看慢查询日志（若 log_config.slow_query_file 已配置）
tail -f logs/slow_query.log
```

### 6.3 SQL 流量捕获（调试用）

`capture` 命令进入**阻塞捕获模式**（当前窗口显示实时 SQL 流）。在 XR:> 会话中执行：
```
# 进入实时 SQL 捕获（阻塞模式；P=暂停/恢复打印，E=退出捕获）
XR:> capture [screen|json|all]

# 按后端过滤
XR:> capture -s 192.168.1.10:3306

# 按客户端过滤
XR:> capture -c 10.0.0.5:0

# 按 SQL 类型过滤（DDL,DML,DQL,DCL,TCL）
XR:> capture -t DML,DDL
```

> **注意**：
> - `capture` **没有** `start`/`stop` 子命令；进入后按 **P** 暂停/恢复，按 **E** 退出捕获模式。
> - `capture_redact_literals = true` 时，SQL 参数值会被 `?` 替换（脱敏）。
> - capture 开启时会降低性能，仅用于短期问题定位。

---

## 7. 后端数据库运维协同

### 7.1 后端计划维护（不停代理服务）

在 XR:> 会话中执行后，再执行独立命令：
```
# 步骤1：停止服务
XR:> stop
```
```bash
# 步骤1（续）：通过 setconfig server <id> 向导禁用该后端
./DBPrisma setconfig server 1     # 交互式向导，将 enabled 改为 false
# 重新启动
./DBPrisma start --no-interactive --config config/conf.json
```
```
# 步骤2：等待该后端连接池清空（在 XR:> 会话中查看）
XR:> backends    # 查看 POOL 列变为 0
```

```
# 步骤3：对该 MySQL 实例做维护操作（略）

# 步骤4：维护完成后重新启用
XR:> stop
```
```bash
./DBPrisma setconfig server 1     # 交互式向导，将 enabled 改回 true
./DBPrisma start --no-interactive --config config/conf.json
```
```
# 步骤5：验证后端恢复
XR:> backends
```

### 7.2 后端主从切换

> 注：`auto_failover`（自动故障转移）当前为**计划中/未实现**（TD-012 清查的保留键，仅解析与 `checkconfig` 校验，引擎侧不切换）。主库故障请按下述**手动切换**流程处理。

主库故障时手动切换：

在 XR:> 会话中执行：
```
# 查看当前路由状态
XR:> backends

# 若自动故障转移未触发，手动将新主库标记为 primary
# 方法：修改 conf.json 中 servers 的 primary 字段后 reload
XR:> reload
```

### 7.3 后端探测与熔断

在 XR:> 会话中执行：
```
# 查看熔断器状态
XR:> status    # 输出中包含 CB 状态

# 后端状态探测配置
# resilience.probe_interval_sec：健康探测间隔
# resilience.probe_failure_threshold：连续失败多少次触发熔断
# resilience.probe_recovery_threshold：连续成功多少次关闭熔断
# circuit_breaker.error_rate_threshold_pct：错误率阈值
```

### 7.4 防止后端 MySQL 封禁代理 IP（`max_connect_errors` / Error 1129）

> **背景**：MySQL 会按**来源 IP** 累计「连接建立后、认证完成前」的失败次数（`max_connect_errors`，默认 100）。超过阈值后，该 IP 被封锁，返回：
>
> `Error 1129: Host 'x.x.x.x' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'`
>
> DBPrisma 连后端时，所有连接对 MySQL 而言都来自**代理机同一 IP**。压测、错误凭据探测、频繁重启/暖机失败等场景容易在代理 IP 上累积计数，导致**整台代理对后端不可用**。
>
> 注意：这与 DBPrisma 自身对客户端的 IP 封禁（`security.auth_fail_max` → 客户端见 1129）是**两套机制**，勿混淆。

> **代理侧已根治（无需额外配置）**：自 2026-06 起，代理对后端的两个「认证完成前断开」来源均已在代码层消除——(1) 认证代理（scheme A）的健康检查不再用「裸 TCP 连接后立即关闭」的探测（改由已认证 `COM_PING`/真实业务连接近期上报即跳过裸探测）；(2) **裸 TCP 预热池彻底退役**（曾对后端建立未认证连接后丢弃，是 `ERROR 1129` 的主要残留源，尤其 scheme B）——现 `warmup_tcp_connections` 已弃用，方案 A+明文后端折算为认证预热，其余场景用冷连接。因此代理**不再向后端产生「认证前断开」的连接错误**。下列后端配置作为**纵深防御**仍然推荐。

**按方案选择（先看这里）：**

| 方案 | 代理是否持有后端账号 | 首选手段 | 说明 |
|------|----------------------|----------|------|
| **scheme A（认证代理）** | 是（`conf.json` `servers[].user`） | 代理已根治 + 方向一/方向三 | 用固定后端专用账号，可精准授予 `CONNECTION_ADMIN`（方向三） |
| **scheme B（透明代理）** | **否**（客户端凭据直传后端） | **方向二（`host_cache_size=0`）** | 代理无凭据，无法做认证级探测；`CONNECTION_ADMIN` **不适用**（每个客户端账号各异），只能在后端关闭封禁机制 |

#### 7.4.1 方向一：调大后端 `max_connect_errors`（推荐，所有 MySQL 版本）

在**每台后端 MySQL** 上调高阈值，降低误封概率。生产环境建议设为较大值（如 `1000000`），并在变更单中记录。

**临时生效（重启 MySQL 后失效）：**

```sql
-- 在 MySQL 上执行（需 SUPER 或 SYSTEM_VARIABLES_ADMIN）
SET GLOBAL max_connect_errors = 1000000;

-- 验证
SHOW GLOBAL VARIABLES LIKE 'max_connect_errors';
```

**永久生效（`my.cnf` / `my.ini`，改后重启 MySQL）：**

```ini
[mysqld]
max_connect_errors = 1000000
```

**已封禁时的应急解封：**

```bash
# 在能连上该 MySQL 的机器上执行（通常从 DBA 跳板机，而非已被封的代理机）
mysqladmin -h <backend_host> -P <port> -u <admin_user> -p flush-hosts
```

> **运维建议**：将 `max_connect_errors` 纳入后端 MySQL 基线配置；代理上线前在每台后端核查一次。压测/长 soak 前从**未被封禁的跳板机**执行 `flush-hosts`，勿依赖代理机自身执行（代理 IP 被封时 `mysqladmin` 也会失败）。

#### 7.4.2 方向二：禁用后端 host 缓存（`host_cache_size=0`，**scheme B 首选 / 最彻底**）

后端封禁计数保存在 MySQL 的 **host 缓存**里；把缓存容量设为 `0` 会**关闭 host 缓存，连带禁用 host 封禁机制**——`Error 1129 (ER_HOST_IS_BLOCKED)` 从此**永不再发**，无论代理产生多少「认证前断开」。这正是 **MySQL 官方推荐的「前置代理 / 负载均衡」场景**配置：所有连接都来自代理同一 IP，按 IP 封禁本就无意义且有害（一个坏客户端会连累全部代理流量）。

同时建议开启 `skip_name_resolve`，免去反向 DNS 解析失败造成的连接错误，并加快连接建立。

**临时生效（运行时即时生效，重启 MySQL 后失效）：**

```sql
-- 需 SUPER 或 SYSTEM_VARIABLES_ADMIN
SET GLOBAL host_cache_size = 0;

-- 验证（应为 0）
SHOW GLOBAL VARIABLES LIKE 'host_cache_size';
```

> `skip_name_resolve` 为只读变量，**不能**用 `SET GLOBAL` 在线修改，只能写 `my.cnf` 后重启。

**永久生效（`my.cnf` / `my.ini`，改后重启 MySQL）：**

```ini
[mysqld]
host_cache_size  = 0
skip_name_resolve = ON
```

> **适用性**：所有 MySQL 版本（5.7 / 8.0 / 8.4 / 9.x）均支持。对 **scheme B（透明代理）是首选**，因为代理不持后端账号、无法走方向三；对 scheme A 亦可作为方向一/三之外的兜底，三者不冲突。
> **权衡**：禁用 host 缓存后，后端不再缓存主机名解析与 per-host 统计；在「代理前置」架构下这恰好是期望行为（无副作用）。若后端**同时**还直接面向大量非代理客户端，则评估后再决定（可改用方向一把 `max_connect_errors` 调到极大值替代）。

#### 7.4.3 方向三：授予后端账号 `CONNECTION_ADMIN`（MySQL 8.0.26+，**仅 scheme A**）

MySQL 8.0.26 起，持有 `CONNECTION_ADMIN` 全局权限的账号，其连接**不受** `max_connect_errors` 限制。适合将 DBPrisma 在 `conf.json` 中配置的**后端专用账号**设为豁免，与方向一可叠加使用。

**前提**：后端 MySQL ≥ 8.0.26；`conf.json` 中 `servers[].user` 使用专用账号（勿与临时测试账号混用）。

**授权示例（在每台后端执行，按实际账号名替换）：**

```sql
-- 查看当前后端账号（与 conf.json servers[].user 一致）
-- 示例账号：dbprisma_backend

GRANT CONNECTION_ADMIN ON *.* TO 'dbprisma_backend'@'%';
FLUSH PRIVILEGES;

-- 验证（应含 CONNECTION_ADMIN）
SHOW GRANTS FOR 'dbprisma_backend'@'%';
```

**与 conf.json 的对应关系：**

| conf.json 字段 | 说明 |
|----------------|------|
| `servers[].user` | 应使用已授予 `CONNECTION_ADMIN` 的后端专用账号 |
| `servers[].password` | 明文或 `ENC2:` 加密；生产建议用 XR:> `passwd`（REPL 运行时命令）或 Shell 子命令 `DBPrisma setconfig srvpwd <id>` 生成密文 |
| `servers[].host` / `port` | 每台后端均需单独执行 `GRANT`（多实例部署时逐台操作） |

> **注意**：`CONNECTION_ADMIN` 为较高权限，仅授予代理专用后端账号，勿授予应用直连账号。MySQL 5.7 / 8.0.25 及以下无此权限，请改用方向一或方向二。
> **scheme B 不适用**：透明代理不持有固定后端账号（客户端各自的凭据直传后端），无法对「代理账号」授权，请改用**方向二（`host_cache_size=0`）**。

#### 7.4.4 上线与压测检查清单

通用：
- [ ] 已区分客户端 1129 与后端 1129：前者查 `security.auth_fail_*` 与代理日志；后者查后端 host 封禁与代理机 IP 是否被封
- [ ] 长 soak / 全量回归前已从跳板机 `flush-hosts`（若此前有压测失败累积）

**scheme A（认证代理）**：
- [ ] 已确认代理为 2026-06 之后版本（裸 TCP 探测已根治；`XR:> backends` 正常）
- [ ] 每台后端 `max_connect_errors` 已按基线调高（方向一）
- [ ] （可选，MySQL 8.0.26+）后端专用账号已授予 `CONNECTION_ADMIN`（方向三）
- [ ] `conf.json` 后端 `user`/`password` 与授权账号一致，暖机可成功

**scheme B（透明代理）**：
- [ ] 每台后端已设 `host_cache_size = 0`（方向二，首选）
- [ ] 每台后端已设 `skip_name_resolve = ON`
- [ ] `servers[].warmup_tcp_connections` 已弃用：裸 TCP 预热已退役（曾是 `ERROR 1129` 根因）。透明代理下该值被忽略并改用冷连接，无需再手动置 0；如仍配置 >0，`checkconfig` 会给出弃用 WARN

> **`ERROR 1129 (Host 'x' is blocked because of many connection errors)` 根因已修复**：旧版「裸 TCP 预热池」对后端建立未认证连接后丢弃，被 MySQL 计入 host 级连接错误并最终拉黑代理。现已退役裸 TCP 预热——认证代理 + 明文后端改用「认证预热」（`warmup_db_connections`，或由 `warmup_tcp_connections` 折算），其余场景用冷连接。残留封禁仍可用 `mysqladmin flush-hosts` 或 `FLUSH HOSTS;` 临时解除。

---

## 8. 故障排查手册

### 8.1 故障排查总流程

```
故障发生
  │
  ├── XR:> status            → 确认代理是否运行（XR:> 会话命令）
  ├── XR:> backends          → 确认后端是否可达（XR:> 会话命令）
  ├── grep ERROR logs/...    → 查看最新错误日志（Shell 命令）
  └── curl /health           → 确认 Admin HTTP 正常（Shell 命令）

  ┌── 代理未运行 ──────────────> 重启代理
  ├── 后端 DOWN ─────────────> 排查后端 MySQL
  ├── 连接耗尽 ──────────────> 调整 max_clients / 清理空闲连接
  └── 高延迟 ────────────────> 查看慢查询日志 / 后端负载
```

### 8.2 客户端无法连接代理

**现象**：`mysql: ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (111)`

排查步骤：
```bash
# 1. 确认代理进程是否运行（Shell 命令）
ps -ef | grep DBPrisma

# 2. 确认端口是否监听
netstat -tlnp | grep 3307        # Linux
netstat -an | findstr 3307       # Windows

# 3. 检查日志启动错误
tail -50 logs/dbprisma.log

# 4. 检查防火墙
iptables -L INPUT -n             # Linux
netsh advfirewall show all       # Windows
```

若代理运行中，可在 XR:> 会话中执行：
```
XR:> status
```

### 8.3 认证失败 Access Denied

**现象**：`ERROR 1045 (28000): Access denied for user 'xxx'@'xxx'`

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 确认代理用户存在
XR:> userlist

# 2. 若用户不存在，添加用户（交互式密码提示，不回显）
XR:> useradd xxx

# 3. 若存在但密码错误（交互式密码提示，不回显）
XR:> setpwd xxx
```
```bash
# 4. 检查 IP 是否被封禁
grep "ban\|blocked\|denied" logs/dbprisma.log | tail -20

# 5. 若 IP 被封禁，需等待 auth_fail_ban_ms 毫秒后自动解封
# 或重启代理（会清空封禁状态）
```

### 8.4 连接数耗尽

**现象**：`ERROR 1040 (08004): Too many connections`

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 查看当前连接数
XR:> conns

# 3. 踢出长时间空闲连接（找出 STATE=IDLE 且 DURATION 很长的连接，然后踢出）
XR:> conns
XR:> kick <client_ip> [port]
```
```bash
# 2. 检查 max_connections_per_ip 是否触发
grep "max_connections_per_ip" logs/dbprisma.log | tail -10

# 5. 检查应用连接池是否有泄漏
```
```
# 4. 提升 max_clients（须停服 → setconfig proxy 向导修改 → 重启）
XR:> stop
```
```bash
./DBPrisma setconfig proxy && ./DBPrisma start --no-interactive --config config/conf.json
```

### 8.5 SQL 被防火墙拦截

**现象**：客户端收到 `MySQL Error 1148: SQL blocked by DBPrisma firewall`

**原理**：防火墙对以下 SQL 执行大小写不敏感子串匹配：
- `COM_QUERY` 及 `COM_STMT_PREPARE` — 检查原始 SQL 文本
- `COM_STMT_EXECUTE` — **双重检查**：① 合成记录文本；② 原始 PREPARE SQL（来自 `fw_stmt_map`）

**注意**：`reject` 模式下只拒绝该条命令并返回 1148 错误，**连接不会断开**，应用可继续发送其他 SQL。

排查步骤：
```bash
# 1. 确认 SQL 防火墙是否开启及当前模式（查看磁盘配置，独立子命令）
./DBPrisma setconfig show security

# 2. 查看被拦截的 SQL 和命中规则
#    主日志（当 sql_firewall_log_blocked=true 时）
grep "FIREWALL_BLOCK" logs/dbprisma.log | tail -20
#    或专用防火墙审计日志（当 firewall_log_file 已配置时）
tail -50 logs/firewall.log
```

若是误拦截，须停服 → setconfig security 向导修改 action → 重启：
```
# 3. 在 XR:> 会话中停止服务
XR:> stop
```
```bash
# 通过 setconfig 修改（security.* 须重启生效）
./DBPrisma setconfig security    # 向导中将 sql_firewall_action 改为 log_only
./DBPrisma start --no-interactive --config config/conf.json

# 4. 调整 sql_block_patterns 后切回 reject（同样须 restart）
./DBPrisma setconfig security    # 向导中将 sql_firewall_action 改回 reject
./DBPrisma start --no-interactive --config config/conf.json

# 5. 若需彻底关闭（生产须记录原因和时间窗口）
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改为 false
./DBPrisma start --no-interactive --config config/conf.json
```

> **提示**：`log_config.firewall_log_file` 写入独立审计流；`log_config.firewall_log_file`/`firewall_log_max_size_mb`/`firewall_log_rotate_count` **支持 `reload` 热更**（路径变更时自动重开文件）；**`security.*`（规则/action/enabled）须 stop→start**（不支持 `reload`）。完整剧本见 [技术 Runbook §7A](../technical/09-operations-runbook.md#7a-事件响应剧本-playbooks)、[迁移指南 §7A](../technical/18-migration-upgrade-guide.md#7a-关闭或回滚-sql-防火墙)。

### 8.6 后端连接失败

**现象**：`Backend connection failed: 192.168.1.10:3306`，或客户端/日志中出现：

`Error 1129: Host '<proxy_ip>' is blocked because of many connection errors`

> 若报文含 `flush-hosts`，且被封 IP 为**代理机地址**，属于后端 MySQL `max_connect_errors` 封禁，见 [§7.4](#74-防止后端-mysql-封禁代理-ipmax_connect_errors--error-1129)。若 1129 出现在**客户端 IP** 且与认证失败相关，则为 DBPrisma 自身 IP 封禁，见 [§8.3](#83-认证失败-access-denied)。

排查步骤：
```bash
# 1. 直接测试后端连通性（在代理机上执行；若代理 IP 已被封，此步会复现 1129）
mysql -h 192.168.1.10 -P 3306 -u dbprisma -p

# 2. 若为后端 1129：从跳板机解封并落实 §7.4 预防措施
mysqladmin -h 192.168.1.10 -P 3306 -u <admin> -p flush-hosts
# 长期：调高 max_connect_errors（§7.4.1）；MySQL 8.0.26+ 可为后端账号授予 CONNECTION_ADMIN（§7.4.2）

# 4. 检查日志中的后端错误
grep "backend\|connect\|1129" logs/dbprisma.log | grep ERROR | tail -20
```

在 XR:> 会话中执行：
```
# 3. 检查后端 MySQL 状态
XR:> backends    # 查看具体后端状态

# 5. 检查熔断器是否触发（状态为 OPEN）
XR:> status      # 查看 CB 状态

# 6. 若熔断器 OPEN，等待 open_duration_ms 后自动进入 HALF_OPEN 探测
```

### 8.7 高延迟 / 慢查询

排查步骤：
```bash
# 1. 查看慢查询日志
tail -100 logs/slow_query.log
```

在 XR:> 会话中执行：
```
# 2. 开启 DEBUG 日志短暂抓取
XR:> loglevel DEBUG
# 等待几分钟
XR:> loglevel INFO

# 3. 开启 SQL 捕获（进入阻塞捕获模式；P=暂停，E=退出）
XR:> capture

# 4. 查看后端延迟
XR:> backends     # 查看 LATENCY 列
```

```sql
-- 5. 检查后端 MySQL 负载（在 MySQL 端执行）
SHOW FULL PROCESSLIST;
SHOW STATUS LIKE 'Threads%';
```

### 8.8 内存持续增长

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 查看代理内存使用
XR:> status       # 查看 RSS

# 3. 检查是否有大量 IDLE 连接未清理（idle_timeout_sec 是否合理）
XR:> conns
```
```bash
# 2. 查看连接是否有泄漏
grep "session_leak\|LEAK" logs/dbprisma.log | tail -20
# 配置 resilience.session_leak_detect = true 可自动检测
```

---

## 9. 紧急操作手册（SOE）

### 9.1 全量封禁某 IP 段（安全事件）

在 XR:> 会话中执行：
```
# 先踢出该 IP 已有连接（kick 接受 IP 字符串）
XR:> kick 10.20.30.40

# 停服
XR:> stop
```
```bash
# 封禁 IP 或 CIDR（如 203.0.113.0/24）；security.* 须停服 → 修改 → 重启才生效
# 手动在 conf.json security.ip_denylist 数组中追加 "10.20.30.40"
# （或使用 setconfig security 向导，数组字段须手动编辑 conf.json）
./DBPrisma setconfig security
./DBPrisma start --no-interactive --config config/conf.json

# 记录操作日志
echo "$(date) Banned IP 10.20.30.40 by operator $(whoami)" >> /opt/dbprisma/logs/ops-audit.log
```

### 9.2 紧急关闭 SQL 防火墙（误拦截生产 SQL）

在 XR:> 会话中停服，然后用独立命令修改配置：
```
# 1. 停服
XR:> stop
```
```bash
# 通过 setconfig security 向导关闭（或改 log_only 保留审计）
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改为 false
                                 # 或将 sql_firewall_action 改为 log_only

# 2. 重启使安全配置生效
./DBPrisma start --no-interactive --config config/conf.json

# 3. 记录变更原因与时间窗口；通知安全团队
# 4. 修复规则后重开并 restart
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改回 true
./DBPrisma start --no-interactive --config config/conf.json
```

### 9.3 代理进程失响（假死）

```bash
# 1. 确认进程存活但不响应（Shell 命令）
ps -ef | grep DBPrisma

# 2. 查看是否有线程卡死
cat /proc/$(pgrep DBPrisma)/status | grep Threads
```

在 XR:> 会话中执行（若进程能响应）：
```
# 3. 尝试优雅重启
XR:> restart
```

```bash
# 4. 若重启失败，强制 kill 后重启
kill -9 $(pgrep DBPrisma)
./DBPrisma start --no-interactive --config config/conf.json

# 5. 启用 Watchdog 防止再次发生（在 conf.json 中配置）
# watchdog.enabled = true
# watchdog.action = "restart"
```

### 9.4 配置误操作回滚

```bash
# 方法1：使用自动备份回滚（备份与 conf.json 同目录；命名 conf.json.bak.YYYYMMDD_HHMMSS）
ls -lt config/conf.json.bak.*
cp config/conf.json.bak.20260605_093000 config/conf.json
```

在 XR:> 会话中执行：
```
# 热更（仅对可热更字段生效）
XR:> reload
```

```bash
# 方法2：使用手动备份
cp conf/conf.json.manual-20260605-093000 conf/conf.json
```

在 XR:> 会话中执行：
```
XR:> reload
```

### 9.5 磁盘空间告急（日志暴涨）

```bash
# 立即压缩旧日志
gzip logs/dbprisma.log.2 logs/dbprisma.log.3

# 清理超期日志（保留最近 3 天）
find logs/ -name "*.log.*" -mtime +3 -delete
```

在 XR:> 会话中执行：
```
# 刷新后端预热连接池
XR:> flush

# 临时降低日志级别
XR:> loglevel WARN
```

---

## 10. 变更操作规程

### 10.1 变更操作原则

1. **先备份后操作**：任何配置变更前备份 `conf.json`
2. **先 `checkconfig` 后 `reload`**：验证通过再热更
3. **灰度测试**：重大变更先在测试环境验证
4. **操作记录**：所有变更操作记录到运维日志
5. **回滚预案**：操作前确认回滚方案

### 10.2 标准变更流程

```
1. 申请变更时间窗口（通知相关业务方）
2. 备份当前配置：cp conf.json conf.json.backup-$(date +%Y%m%d)
3. 修改配置文件
4. 验证（独立子命令）：./DBPrisma checkconfig
5. 执行热更（XR:> 会话）：XR:> reload
6. 验证（XR:> 会话）：XR:> status 然后 XR:> backends
7. 观察 5-10 分钟（检查错误日志、业务指标）
8. 填写变更记录
```

### 10.3 紧急变更流程

```
1. 通知直属负责人
2. 备份当前配置
3. 执行最小变更（仅修改必要项）
4. 立即验证
5. 变更完成后 30 分钟内补填变更记录
```

---

## 11. 监控告警配置参考

### 11.1 推荐告警规则（Prometheus/Alertmanager）

```yaml
# 代理宕机
- alert: DBPrismaDown
  expr: up{job="dbprisma"} == 0
  for: 30s
  labels:
    severity: critical
  annotations:
    summary: "DBPrisma proxy is down"

# 后端 DOWN
- alert: DBPrismaBackendDown
  expr: dbprisma_backend_status == 0
  for: 1m
  labels:
    severity: warning

# 活跃连接数超阈值
- alert: DBPrismaConnectionHigh
  expr: dbprisma_connections_active > 800
  for: 2m
  labels:
    severity: warning

# 错误率过高
- alert: DBPrismaErrorRateHigh
  expr: rate(dbprisma_errors_total[5m]) > 10
  for: 2m
  labels:
    severity: critical

# 后端延迟过高
- alert: DBPrismaBackendLatencyHigh
  expr: dbprisma_backend_latency_ms > 100
  for: 3m
  labels:
    severity: warning
```

### 11.2 健康检查集成（Nginx upstream）

```nginx
upstream dbprisma {
    server 127.0.0.1:3307 max_fails=3 fail_timeout=30s;
}

# 健康检查（需 nginx_upstream_check_module）
check interval=3000 rise=2 fall=3 timeout=1000 type=http;
check_http_send "GET /health HTTP/1.0\r\nAuthorization: Bearer TOKEN\r\n\r\n";
check_http_expect_alive http_2xx;
```

---

## 12. 运维命令速查表

> **说明**：标注「XR:>」的命令须在交互会话中执行；标注「Shell」的可直接从终端调用。

| 场景 | 命令 | 类型 |
|------|------|------|
| 查看代理状态 | `XR:> status` | XR:> |
| 查看后端健康 | `XR:> backends` | XR:> |
| 查看活跃连接 | `XR:> conns` | XR:> |
| 踢出连接 | `XR:> kick <client_ip> [port]` | XR:> |
| 热更配置 | `XR:> reload` | XR:> |
| 修改单项配置（磁盘） | `./DBPrisma setconfig <verb> [args]`（如 `setconfig srvpwd 1`） | Shell |
| 修改日志级别 | `XR:> loglevel <level>` | XR:> |
| 封禁 IP | 停服 → 手动编辑 conf.json `security.ip_denylist` → 重启 | Shell |
| 暂停新连接 | `XR:> pause` | XR:> |
| 恢复新连接 | `XR:> resume` | XR:> |
| 排水（drain） | `XR:> drain <server_id>` | XR:> |
| 优雅重启 | `XR:> restart` | XR:> |
| 优雅停止 | `XR:> stop` | XR:> |
| 带倒计时停止 | `XR:> shutdown [秒] [force\|cancel]` | XR:> |
| 验证配置 | `./DBPrisma checkconfig` 或 `./DBPrisma cc` | Shell |
| 轮换 Token | `XR:> token rotate` | XR:> |
| 刷新预热池 | `XR:> flush [server_id]` | XR:> |
| SQL 捕获（阻塞模式；P=暂停，E=退出） | `XR:> capture [opts] [screen\|json\|all]` | XR:> |
| 后台 SQL 捕获（启动时） | `./DBPrisma start --bg-capture` | Shell |
| 状态一次性显示（读磁盘） | `XR:> display` | XR:> |
| 查看代理用户 | `XR:> userlist` | XR:> |
| 添加代理用户 | `XR:> useradd <username>`（交互式密码提示） | XR:> |
| Admin 健康检查 | `curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health` | Shell |
| Admin 指标 | `curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/metrics` | Shell |

---

## 13. 授权与 License 管理

DBPrisma 启动时由 License 闸门（`DBPrisma/license_gate.c`）控制是否放行，逻辑完全离线，不联网。

### 13.1 判定规则

| 情况 | 结果 | 行为 |
|------|------|------|
| **config 目录无 `license.dat`** | 试用 | 自首次运行起 30 天内放行；到期后拒绝启动 |
| `license.dat` 验签通过、`key_id` 与内嵌 `VENDOR_KEY_ID` 一致、绑定本机、已到 `activate_utc`、未过期（或在 `grace_days` 宽限期内） | 授权 | 放行 |
| `license.dat` 签名无效 / 文件损坏 / 格式错误 / `key_id` 与内嵌公钥不匹配 | 拒绝 | **报错中止，不回落试用** |
| `license.dat` 绑定到其它机器 | 拒绝 | 报错中止 |
| `license.dat` 未到 `activate_utc`（预签发尚未生效） | 拒绝 | 报错中止 |
| `license.dat` 已过期且超出 `grace_days` 宽限期 | 拒绝 | 报错中止 |
| 试用状态相对系统时钟回拨 | 拒绝 | 报错中止 |

> 关键约定：**只要存在 `license.dat`，它就必须有效**；只有「完全没有该文件」时才走 30 天试用。宽限期内仍放行，但 `Detail` 会提示已进入 grace period。

### 13.2 相关文件与位置

| 文件 | 位置 | 说明 |
|------|------|------|
| `license.dat` | **config 目录**（与 `conf.json` 同级） | 厂商签发的授权文件，由客户放置 |
| `dbprisma.trial.state` | **run 目录** | 试用起点（机器绑定 AES-GCM 封存，**勿删**，删除会重置 30 天） |

> 启动时控制台/日志会打印一行 `[license] ...` 表明当前授权状态与剩余天数。
> 守护进程/服务模式下每小时复检一次，license/试用到期会自动停止服务。

**到期前提示（`license.warn_days`）**：当 license **或**试用剩余天数 ≤ `license.warn_days`（默认 14 天，见 [配置参考](configuration-reference.md)）时，启动会打印 `[license] License/Trial expires in N day(s); renew ...`，守护模式下每小时复检也会重复该提醒；`status verbose` 的 `Days left` 在此阈值内转黄。设 `warn_days=0` 关闭提醒。此提示对**正式 license 与试用同时生效**（对称）。

**监控指标**：`/metrics`（及命令管道 `metrics` op）输出两项授权指标，便于到期前自动告警：
> - `dbprisma_license_state`：`2`=授权、`1`=试用、`0`=拒绝、`-1`=未知
> - `dbprisma_license_days_left`：剩余整天数（`-1`=未知）

示例告警规则：`dbprisma_license_state >= 1 and dbprisma_license_days_left <= 14`。

**随时查看授权状态**：在 `XR:>` 会话执行 `status verbose`，输出末尾的 `---- License ----` 段会显示 `Mode`（LICENSED / TRIAL / DENIED）、`Days left`（剩余天数）与 `Detail`（如 `licensed to <组织>, expires ...` 或 `trial: N day(s) left`）。这是除启动行外随时可查的入口（不会重置试用）。

### 13.3 申请与部署流程

```
客户: licreq --print-fingerprint（可选，查看本机指纹）
客户: licreq 生成 license.request（含本机机器指纹与订单 SN；月数由 SN 解析）→ 核对生成摘要或 licreq --show <file> → 发给厂商
厂商: licissue show 审核（核对 machine_fp；可加 --art/--all 打印 randomart）
      → licissue issue --private-key ... --public-key ... 签发 license.dat（可 --months / --days / --activate-utc / --grace-days；台账 issued-ledger.jsonl）
      → licissue inspect -i license.dat 复核（可选）
客户: 将 license.dat 放入 config 目录 → 重启 DBPrisma
```

**客户侧常用命令**：

```bash
licreq --print-fingerprint              # 默认：单行 hex，便于脚本捕获
licreq --print-fingerprint --art        # 仅 randomart 图案（人工核验）
licreq --print-fingerprint --all        # hex + 图案
licreq -o license.request               # 交互式生成申请（成功后会打印请求摘要）
licreq --show license.request           # 解密并回显已生成的申请文件
# 机器指纹：SHA256(CPU序列号 | hostname | salt)，与 license.dat 绑定字段同源
```

**厂商侧签发示例**（须同时提供私钥与公钥）：

```bash
licissue show -i license.request
licissue issue -i license.request -o license.dat \
  --private-key vendor.key --public-key vendor.pub -y
# 按天签发（覆盖申请月数）：--days 28
licissue inspect -i license.dat         # 查看序列号、四时间字段、宽限天数等
```

**跨平台私钥（portable key）**：自 2026-06-22 起，`licissue keygen` 生成的私钥
（`DBPVK1`，150 字节 = `d‖X‖Y`）与公钥一样是跨平台格式，**同一个私钥文件可在
Windows 与 Linux 上签发**，签出的 `license.dat` 在所有 DBPrisma（任意 OS/架构）上
均可验证。公钥与 `key_id` 不变，**无需重新 keygen、无需重编 DBPrisma**。

历史上 Windows 私钥是 BCrypt blob（158 字节）、Linux 是裸标量（54 字节），互不通用。
两种旧格式仍可被新 `licissue` 读取（向后兼容）；将旧私钥一次性迁移为跨平台格式：

```bash
licissue convert-key -i vendor.old.key -o vendor.key   # 公钥/ key_id 不变
```

> 迁移 Windows 旧 blob 在任意平台均可；迁移 Linux 旧裸标量需在 Linux 上执行
> （需用曲线运算推导公钥点）。

详见 `tools/licreq/README.md`、`tools/licissue/README.md`。

### 13.4 启动失败排障（`[license] Startup blocked: ...`）

| 提示 | 原因 | 处置 |
|------|------|------|
| `trial period has ended` | 30 天试用到期且无 license | 申请并放入有效 `license.dat` |
| `trial state invalid or tampered` | `run/dbprisma.trial.state` 被改动/损坏/清空，或机器身份变化（CPU 标识、主机名变化、整目录跨机拷贝）导致机器绑定 AES-GCM 解封失败 | 正解：放入有效 `license.dat`（不再走试用）。开发/测试环境如确需重置试用：删除 `run/dbprisma.trial.state` 后重启（**等于重置 30 天**，受授权策略约束） |
| `license.dat is invalid or corrupt` | 文件被改动/损坏，或 `key_id` 与内嵌公钥不匹配（密钥轮换后旧证） | 重新获取与当前产品版本公钥匹配的 `license.dat`；确认已 `keygen --emit-header` 并重编译 |
| `license.dat is bound to a different machine` | license 绑定的机器指纹与本机不符（换机/重装/CPU 或主机名变化） | 本机 `licreq --print-fingerprint` 与 `licissue show` 中的 `machine_fp` 对比；不符则重新 `licreq` 申请并换发 |
| `license not active until ...` | 预签发的 license 尚未到 `activate_utc` | 等到生效日，或请厂商重签（调整 `--activate-utc`） |
| `license expired at ...` | license 到期且超出 `grace_days` 宽限期 | 续期换发新的 `license.dat` |
| `system clock rolled back` | 系统时钟相对试用起点回拨 | 校正系统时间（NTP） |

> 机器指纹来源与 ENC2 密钥同源（`SHA256(CPU序列号|hostname|salt)`）。更换 CPU、主机名或重装可能导致指纹变化、需换发 license。

---

*配置详情参阅 [配置参考手册](configuration-reference.md)*  
*安装部署参阅 [管理员手册](admin-manual.md)*  
*安全加固参阅 [安全加固指南](security-hardening-guide.md)*  
*授权工具参阅 [licreq](../../tools/licreq/README.md) / [licissue](../../tools/licissue/README.md)*
