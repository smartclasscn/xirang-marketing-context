# conf.json 配置项参考手册

| 版本 | r19 | 日期 | 2026-07-13 |

> 部署与证书、mysql 客户端用法见 [configuration-guide.md](configuration-guide.md)。  
> 配置文件路径：`bin/<Platform>/<arch>/config/conf.json`
> 格式：标准 JSON，所有字段均可省略，省略时使用括号内的默认值。
> 解析器共识别 **162** 个配置键（含 `version`、`servers[]` 每元素字段计 1、数组计 1）。
> 其中 157 个为标量叶子键（`json_get_`*），5 个为数组/集合键（`proxy.admin_allow_ips`、
> `security.ip_allowlist`、`security.ip_denylist`、`security.sql_block_patterns`、`servers` 数组）。
> 标量叶子键含 `paths` 段的 3 个运行期目录键（`run_dir`/`log_dir`/`capture_dir`，见 §十九-A）。

---

## 〇、实现状态总览（重要）

> DBPrisma 会解析下列键，但它们当前**未被运行时消费**或被取代。`checkconfig` 与
> `start` 在检测到 conf.json **显式设置**这些键时会打印 `[WARN] config: <段>.<键> ...`，
> 以避免"以为生效、实则空转"的误解。详见 `docs/technical/14-risk-technical-debt-register.md`（TD-010）。


| 段.键                                                                                                              | 状态         | 说明                                                         |
| ---------------------------------------------------------------------------------------------------------------- | ---------- | ---------------------------------------------------------- |
| `resilience.keepalive_idle_sec` / `keepalive_interval_sec` / `keepalive_count`                                   | **已生效**     | `backend_tcp_keepalive=true` 时下发：Linux `TCP_KEEPIDLE/INTVL/CNT`；Windows `SIO_KEEPALIVE_VALS`（idle/interval，count 由 OS 固定） |
| `resilience.retry_on_lost_connection` / `retry_idempotent_only` / `max_retry_count` / `retry_delay_ms`           | 保留         | 失败重试未实现                                                    |
| `resilience.auto_failover` / `failover_target_id` / `failover_restore_on_recovery` / `failover_restore_delay_ms` | 部分生效     | `auto_failover`+`failover_target_id` 已用于**新连接**重定向至健康后端；`failover_restore_*` 仍为保留（不自动切回 primary 配置） |
| `resilience.session_leak_action`                                                                                 | 仅 `log`     | `log` 已实现；`force_close` **有意不实现**（TD-006 安全决策：强制释放卡死会话会绕过 `io_inflight` 门控引入 UAF，风险>收益） |
| `mode.rw_ddl_to_primary` / `rw_dcl_to_primary` / `rw_transaction_detection`                                      | 保留(TD-008) | 非 DQL 与事务已硬编码走主库；该值未读取                                     |
| `mode.mirror_enabled` / `mirror_server_id` / `mirror_ratio_pct` / `mirror_async` / `mirror_ddl`                  | 计划中(TD-005) | 镜像流量未实现（解析+`checkconfig` 校验已具备，引擎侧旁路复制待实现）                     |
| `security.hide_server_version`                                                                                   | 常开加固（无开关） | 合成握手**始终**上报通用 MySQL 版本号；该字段仅为兼容保留，置 `true/false` 均无差别       |
| `security.strip_client_attributes`                                                                               | 常开加固（无开关） | 客户端 `CLIENT_CONNECT_ATTRS` **始终**被剥离（代理不转发属性载荷）；该字段仅为兼容保留     |
| `security.max_packet_size_bytes`                                                                                 | **已生效**    | >0 时作为单条客户端命令更严格的字节上限（超出即拒绝/断开）；0=沿用 `proxy.max_reassemble_bytes`；须重启 |
| `circuit_breaker.fallback_policy`                                                                                | **已生效**     | `reject` OPEN 态快速失败；`queue` 在连接建立处按连接超时预算有界排队等待熔断器恢复，超时再拒  |
| `log_config.log_include_sql`                                                                                     | **已生效**    | `true` 时每条捕获语句写主日志（`SQL session=... sql=...`，遵循 `capture_redact_literals`）；须重启 |
| `config_manage.max_backup_count`                                                                                 | **已生效**    | `reload` 与 `setconfig` 写盘均创建 `conf.json.bak.`* 并按上限裁剪旧备份   |
| `notify.syslog`                                                                                                  | **已生效**    | `true` 时（仅 Linux）事件写入 syslog；`false` 不写                    |
| `notify.win_event_log`                                                                                           | 平台受限       | 仅 Windows 生效                                               |
| `tls.cipher_list` / `prefer_server_ciphers` / `session_ticket_enabled`                                           | 平台受限       | 仅 OpenSSL/Linux 生效；Windows SChannel 忽略                     |


> **本版新增已生效项**：`tls.min_tls_version`（两平台均生效；OpenSSL 路径下 `cipher_list`/
> `prefer_server_ciphers`/`session_ticket_enabled` 同时生效）、`security.long_query_warn_ms`
> （超过阈值在主日志打 `[WARN] long query ...`）。

---

## 顶层结构

```json
{
  "version":          "0.0.1",
  "proxy":            { ... },
  "servers":          [ { ... }, { ... } ],
  "mode":             { ... },
  "resilience":       { ... },
  "logging":          { ... },
  "security":         { ... },
  "capture":          { ... },
  "circuit_breaker":  { ... },
  "watchdog":         { ... },
  "tls":              { ... },
  "log_config":       { ... },
  "config_manage":    { ... },
  "notify":           { ... },
  "paths":            { ... },
  "command_pipe":     { ... },
  "license":          { ... }
}
```

> `notify` 段（N-06，OS 即时通知，含去抖）：`win_event_log`（仅 Windows）、`syslog`（仅 Linux）、
> `debounce_sec`、`on_backend_error`、`on_circuit_breaker`、`on_firewall_block`、`on_auth_fail`、
> `on_config_reload`。`syslog` 为 Linux 后端开关，`win_event_log` 为 Windows 后端开关，各按平台生效。

> `command_pipe` 段（带外管理控制通道，可选，默认禁用）：通过 Windows 命名管道 / Linux AF_UNIX
> 套接字让运维客户端**远程登录并执行管理子命令**（`start`/`stop`/`status`/`reload`/`capture` 等），
> 鉴权用户来自独立的 `admin_users.json`。详见 §二十二。


| 顶层字段      | 类型  | 默认值  | 说明                                                                                                                                                                                                                                 |
| --------- | --- | ---- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `version` | 字符串 | （必填） | 配置文件版本，**必须为三段 semver**（如 `"0.0.1"`，即 `PA_CONFIG_FILE_VERSION` / `PA_VERSION_SEMVER`）。须与引擎 `**PA_VERSION_STRING` 的前三段**一致（引擎完整版如 `0.0.1.568` 含构建号，**勿写入配置**）。省略、写四段或不匹配时 `**checkconfig` / `start` 失败**；旧整数 `config_version` 字段将报错 |


---

## 一、`proxy` 段（全局代理参数）

```json
"proxy": {
  "worker_threads":          0,
  "max_clients":             1024,
  "max_reassemble_bytes":    16777216,
  "connect_timeout_ms":      5000,
  "idle_timeout_sec":        0,
  "capture_redact_literals": true,
  "allow_legacy_password":   false
}
```

> **注：** 项目附带的 `bin/<Platform>/<arch>/config/conf.json` 模板将 `connect_timeout_ms` 设为 `1000`、`idle_timeout_sec` 设为 `300`（更激进的失败检测与回收策略，便于测试）；上方示例显示的是 **省略字段时的真实回退默认**，与代码 `config_loader.c` 一致。


| 配置项                        | 类型    | 默认值         | 取值范围                 | 说明                                                                                                                                                                                           |
| -------------------------- | ----- | ----------- | -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `worker_threads`           | 整数    | `0`         | `0` 或 `1`–`64`       | IOCP/epoll 工作线程数。`0` = 自动（CPU 核数 × 2）；超出范围 **启动失败**                                                                                                                                          |
| `max_clients`              | 整数    | `1024`      | `1` ~ `1024`         | 最大并发客户端连接数（上限 `PA_MAX_CLIENTS=1024`），超出自动截断                                                                                                                                                  |
| `connect_timeout_ms`       | 整数    | `5000`      | `> 0`                | 连接后端 MySQL 的超时时间（毫秒）；`<=0` 时启动失败                                                                                                                                                             |
| `idle_timeout_sec`         | 整数    | `0`         | `>= 0`               | 客户端空闲超时（秒）。`0` = 永不超时；`< 0` 时启动失败                                                                                                                                                            |
| `max_reassemble_bytes`     | 整数    | `16777216`  | `65536` ~ `1073741824` | 每个会话、每个方向（c2s / s2c）**堆上** MySQL 粘包重组缓冲的 **增长上限（天花板）**（字节）。缓冲并非一次性按此值预分配，而是从小额初值（约 16 KiB）起 **按需几何增长**，最高至本上限。默认 16 MiB；一条逻辑语句可跨多个 16 MiB 协议包（续传包），因此允许配置大于 16 MiB 的值，上限 **1 GiB**（对齐 MySQL 服务端 `max_allowed_packet` 硬上限）。超出 1 GiB 或低于 64 KiB 时自动钳制并打印 `[WARN]`。若配置大于 16 MiB，后端 `max_allowed_packet` 须不小于该值方可实际转发同等大小的语句 |
| `capture_redact_literals`  | 布尔    | 见下          | —                    | SQL 捕获与影子 WARN 日志的字面量脱敏。含 `proxy` 段且省略本字段时默认 `**true`**；整文件无 `proxy` 段时为 `**false**`                                                                                                         |
| `allow_legacy_password`    | 布尔    | `false`     | —                    | 是否参与 `mysql_old_password`（scramble323）。生产建议保持默认 `**false**`；仅对遗留 MySQL 4.1 前客户端才需 `true`；支持 `**reload**`                                                                                     |
| `admin_listen_ip`          | 字符串   | `127.0.0.1` | —                    | Admin 绑定地址；改此项需 **重启**                                                                                                                                                                       |
| `admin_listen_port`        | 整数    | `0`         | `0`–`65535`          | Admin 端口；`**0` = 关闭**                                                                                                                                                                        |
| `admin_bearer_token`       | 字符串   | `""`        | —                    | 非空时所有端点须 Bearer；支持 `ENC2:`；可 **reload**                                                                                                                                                       |
| `admin_allow_ips`          | 字符串数组 | `[]`        | —                    | 非空时仅列出的客户端 IP 可访问 Admin；可 **reload**                                                                                                                                                         |
| `admin_request_timeout_ms` | 整数    | `5000`      | —                    | Admin 单连接读超时                                                                                                                                                                                 |
| `admin_max_header_bytes`   | 整数    | `8192`      | `1024`–`16383`       | Admin HTTP 请求头接收缓冲上限；超出 16383 时自动截断至 16383                                                                                                                                                   |
| `admin_tls_pfx_file`       | 字符串   | `""`        | —                    | Admin HTTPS 服务端证书（PFX/PKCS#12），相对 conf.json 目录；配置后该端口启用 HTTPS；改此项需 **重启**                                                                                                                    |
| `admin_tls_pfx_password`   | 字符串   | `""`        | —                    | PFX 口令；支持 `ENC2:`（AES-256-GCM 加密存储）；改此项需 **重启**                                                                                                                                               |
| `admin_tls_cert_file`      | 字符串   | `""`        | —                    | Admin HTTPS 证书（PEM），与 `admin_tls_key_file` 成对使用（`pfx` 优先级更高）；改此项需 **重启**                                                                                                                     |
| `admin_tls_key_file`       | 字符串   | `""`        | —                    | Admin HTTPS 私钥（PEM）；改此项需 **重启**                                                                                                                                                              |


**Admin 端点（HTTP 与 HTTPS 完全一致，只读监控）：** `GET /health`（JSON）、`GET /metrics`（Prometheus 文本）、`GET /events`（最近 64 条事件 JSON）、`GET /audit/search?q=<关键词>&event=<类型>&user=<用户>&tool=<工具>&limit=<N>&offset=<M>`（审计 NDJSON 子串检索；`limit` 默认 20、最大 100；`offset` 分页跳过前 N 条匹配；响应含 `returned`/`has_more`）。其它路径返回 `404`，非 `GET` 返回 `405`。

**HTTP / HTTPS 双协议（同端口自动识别）：** Admin 端口对每个连接按首字节自动判别协议（TLS 记录以 `0x16` 开头）——

- 配置了证书（`admin_tls_pfx_file` 或 `admin_tls_cert_file`+`admin_tls_key_file`）→ 同端口**同时**接受 HTTP（明文）与 HTTPS（加密）；
- 未配置证书 → 仅 HTTP；
- 配置了证书但加载失败 → **启动失败（fail-closed）**，绝不静默降级为明文。

默认仅本机；绑定 `0.0.0.0` 时必须配置 `admin_bearer_token`，且强烈建议启用 HTTPS。

**大包转发（方案 A 命令阶段）**：代理在客户端侧按 `max_reassemble_bytes` 重组 MySQL 包后整包转发至后端；单条 `COM_QUERY` 应小于该上限且不超过后端 `max_allowed_packet`。SQL 捕获仍最多记录 **4096** 字节（`PA_MAX_SQL_LEN`）。

**内存提示**：重组缓冲为 **按需增长**——每个会话的 c2s / s2c 缓冲从小额初值（约 16 KiB/方向）起分配，仅在转发的包确需更大空间时几何增长（doubling），上限为 `max_reassemble_bytes`（另有可选 P1 TLS 解密暂存缓冲，初值约 64 KiB、同样增长至该上限）。因此常见的小查询连接稳态仅占数十 KiB，而非急切预留满额；只有真正处理接近上限的大包时单方向才会逼近 `max_reassemble_bytes`。最坏情况上界仍为约 `2 × max_reassemble_bytes`/会话（双向各一块），即 `max_clients=1024` 且 `max_reassemble_bytes=16777216` 时理论峰值约 **32 GiB**，但该上界仅在所有连接同时处理满额大包时才会触及，正常负载下实际占用远低于此。生产环境若同时面临高并发与大包，仍建议按需调低 `max_reassemble_bytes` 或 `max_clients`。本值为 **非 reload 字段**：`PA_Reload` 检测到其变更时会记录 `restart required` 并保持旧上限，需 **重启** 方可生效。

> **glibc 内存归还（Linux 运维提示）**：在 Linux 上，高并发 connect/close 压测会观察到代理 RSS 在前若干分钟内缓慢上升后**趋于平台期**（如单核 2 GB 测试机上双 TLS 30 万次连接后稳定在约 247–286 MB）。经 AddressSanitizer/LeakSanitizer 在双 TLS（`backend_ssl_mode=required`）路径下驱动 3000 次连接验证，**无任何按连接（per-connection）内存泄漏**（仅 2 处进程生命周期内的一次性启动分配，属正常 still-reachable）。该 RSS 微涨是 **glibc malloc 多 arena 的内存保留/碎片**（freed 内存未即时归还 OS），并非泄漏。内存敏感的小内存主机可在服务环境中设置 `MALLOC_ARENA_MAX=2`（并可配 `MALLOC_TRIM_THRESHOLD_=131072`）显著抑制该现象（实测 15 分钟软测试 RSS 增幅由约 82 MB 降至约 27 MB 且尾部完全平稳）。默认未在代码内强制限制 arena 数，以免影响多核主机的 malloc 并发吞吐——此为可选运维调优项。

### 1.1 `proxy.startup` 子段（启动行为）

```json
"proxy": {
  "startup": {
    "wait_warmup_before_accept":   true,
    "warmup_ready_timeout_ms":     30000,
    "warmup_ready_min_pct":        50,
    "backend_preflight_tcp":       true,
    "backend_preflight_timeout_ms": 0
  }
}
```


| 配置项                            | 类型  | 默认值     | 说明                                                 |
| ------------------------------ | --- | ------- | -------------------------------------------------- |
| `wait_warmup_before_accept`    | 布尔  | `true`  | `true` 时代理等待预热连接池就绪后才开始接受客户端连接                     |
| `warmup_ready_timeout_ms`      | 整数  | `30000` | 等待预热就绪的最长时间（毫秒）；超时后无论就绪率如何均继续启动                    |
| `warmup_ready_min_pct`         | 整数  | `50`    | 预热就绪率阈值（0–100%）；达到此比例后即认为预热完成，允许开始接受连接             |
| `backend_preflight_tcp`        | 布尔  | `true`  | 启动时对每个启用后端做一次 TCP 连接探测；失败仅记录 WARN，不阻止启动            |
| `backend_preflight_timeout_ms` | 整数  | `0`     | Preflight 探测连接超时（毫秒）；`0` = 继承 `connect_timeout_ms` |


### 1.2 `proxy.shutdown` 子段（关闭行为）

```json
"proxy": {
  "shutdown": {
    "graceful_drain_ms":       15000,
    "txn_drain_ms":            60000,
    "stop_deadline_ms":        180000,
    "pool_empty_wait_ms":      15000,
    "pool_empty_wait_final_ms": 5000,
    "io_join_initial_ms":      45000,
    "io_join_extended_ms":     120000,
    "rw_relay_stop_join_ms":   45000,
    "reject_errno":            1317,
    "reject_message":          "Server shutdown in progress",
    "allow_ping":              true
  }
}
```


| 配置项                        | 类型  | 默认值                             | 说明                                    |
| -------------------------- | --- | ------------------------------- | ------------------------------------- |
| `graceful_drain_ms`        | 整数  | `15000`                         | 优雅关闭排空阶段时长（毫秒）；期间不再接受新连接，等待现有连接完成     |
| `txn_drain_ms`             | 整数  | `60000`                         | 事务排空阶段最长等待（毫秒）；尽量等待活跃事务提交或回滚          |
| `stop_deadline_ms`         | 整数  | `180000`                        | 整体停止截止时限（毫秒）；超时后强制终止；`0` = 无截止（不推荐）   |
| `pool_empty_wait_ms`       | 整数  | `15000`                         | 连接池排空等待（毫秒）                           |
| `pool_empty_wait_final_ms` | 整数  | `5000`                          | 连接池最终排空等待（毫秒）                         |
| `io_join_initial_ms`       | 整数  | `45000`                         | IO 工作线程第一阶段 join 等待（毫秒）               |
| `io_join_extended_ms`      | 整数  | `120000`                        | IO 工作线程第二阶段 join 等待（毫秒）               |
| `rw_relay_stop_join_ms`    | 整数  | `45000`                         | 读写分离 relay 线程 stop 等待（毫秒）             |
| `reject_errno`             | 整数  | `1317`                          | 排空期间对新建连接返回的 MySQL 错误码                |
| `reject_message`           | 字符串 | `"Server shutdown in progress"` | 排空期间对客户端返回的错误消息                       |
| `allow_ping`               | 布尔  | `true`                          | 排空期间是否允许 `COM_PING`（`true` 可保持健康检查通过） |


> `startup` 和 `shutdown` 所有字段均支持 `**reload**`。

---

## 二、`servers` 段（数组，最多 16 个，由 `PA_MAX_SERVERS` 定义）

每个元素代表一个代理监听实例。`**scheme` 字段必须显式配置**，决定代理工作模式。

```json
"servers": [
  {
    "id":                      1,
    "host":                    "127.0.0.1",
    "port":                    3306,
    "listen_ip":               "0.0.0.0",
    "listen_port":             33006,
    "weight":                  1,
    "enabled":                 true,
    "user":                    "",
    "password":                "",
    "warmup_tcp_connections":  0,
    "warmup_db_connections":   0
  }
]
```

### 2.1 基础字段


| 配置项           | 类型  | 默认值           | 说明                                                                                                                   |
| ------------- | --- | ------------- | -------------------------------------------------------------------------------------------------------------------- |
| `id`          | 整数  | 自动递增          | 服务标识，仅用于日志输出区分，无功能影响                                                                                                 |
| `host`        | 字符串 | `"127.0.0.1"` | 后端 MySQL 服务器 IP 地址                                                                                                   |
| `port`        | 整数  | `3306`        | 后端 MySQL 端口                                                                                                          |
| `listen_ip`   | 字符串 | `"0.0.0.0"`   | 代理监听 IP。`"0.0.0.0"` 表示监听所有网卡                                                                                         |
| `listen_port` | 整数  | `3307`        | 代理监听端口（客户端连此端口）                                                                                                      |
| `weight`      | 整数  | `1`           | `>= 0`                                                                                                               |
| `enabled`     | 布尔  | `true`        | `false` 时此实例不启动监听                                                                                                    |
| `primary`     | 布尔  | `false`       | 仅 `mode.strategy` 为 `dispatch` 时有效：固定该实例为分发主后端；全组至多一个；不配置时仍按轮询选主                                                     |
| `role`        | 字符串 | `""`          | 仅 `mode.strategy` 为 `read_write` 时有效：`"write"`（恰好 1 个 enabled）或 `"read"`（≥1 个 enabled）；其它 strategy 下设置会 **WARN 并忽略** |


### 2.2 代理模式字段（★ 关键）

`**scheme` 字段是必填项，明确指定代理工作模式：**


| `scheme` 值      | 代理模式               | 说明                                 |
| --------------- | ------------------ | ---------------------------------- |
| `"auth"`        | **认证代理**（Scheme A） | 代理拦截认证，客户端使用 `proxy_users.json` 账号 |
| `"transparent"` | **透明代理**（Scheme B） | 客户端认证原样透传给后端 MySQL                 |


> **注意：** `scheme` 为必填字段，省略或填写其他值会导致**启动失败**。不再支持通过 `user` 字段推断代理模式。


| 配置项        | 类型  | 默认值  | 说明                                                              |
| ---------- | --- | ---- | --------------------------------------------------------------- |
| `scheme`   | 字符串 | —    | **必填**。工作模式：`"auth"`（认证代理）/ `"transparent"`（透明代理）；省略或填写其他值时启动失败 |
| `user`     | 字符串 | `""` | 后端 MySQL 账号。认证代理时为连接后端的凭据                                       |
| `password` | 字符串 | `""` | 后端 MySQL 密码。支持明文或 `ENC2:` 加密格式（见下方说明）                            |


**密码加密格式：**
在 XR:> 提示符下使用 `passwd`（交互式，推荐）生成加密字符串，或用 Shell 子命令 `DBPrisma setconfig srvpwd <id>`（直接写盘修改指定后端的加密密码）。
加密使用 AES-256-GCM（`ENC2:` 前缀，含 IV + 认证标签，机器绑定）：

```json
"password": "ENC2:rJ3kLmP9..."
```

> **旧 `ENC:`（AES-256-CBC）格式已移除**：不再生成，也不再被识别为密文。若配置中残留 `ENC:...` 串，将被**当作字面量明文**直接使用（导致认证失败）。请用 `passwd` 重新生成 `ENC2:` 串。

### 2.3 TLS 终结字段（Scheme A 专属）


| 配置项                   | 类型  | 默认值          | 说明                                                                                            |
| --------------------- | --- | ------------ | --------------------------------------------------------------------------------------------- |
| `tls_termination`     | 布尔  | `false`      | 仅 `scheme = "auth"`（方案 A）有效。`true` = 代理在 P1（客户端侧）终结 TLS。                                      |
| `tls_p1_pfx_file`     | 字符串 | `""`         | PFX/P12 服务端证书（**优先**）。路径相对 `conf.json` 所在目录。                                                  |
| `tls_p1_pfx_password` | 字符串 | `""`         | PFX 密码；支持 `ENC2:` 加密存储。**最大 128 字节**（包含 `ENC2:` 前缀和 NUL；超长会被截断，PFX 解码将失败）                       |
| `tls_p1_cert_file`    | 字符串 | `""`         | PEM 证书（与 `tls_p1_key_file` 成对）。                                                               |
| `tls_p1_key_file`     | 字符串 | `""`         | PEM 私钥（`BEGIN RSA PRIVATE KEY` 或 PKCS#8 `BEGIN PRIVATE KEY`；Windows 上 EC 等 exotic 密钥仍建议 PFX）。 |
| `tls_p1_cert_cn`      | 字符串 | `""`         | 未配置证书文件时，自签名回退证书的 CN（默认 `DBPrisma`）。                                                          |
| `backend_ssl_mode`    | 字符串 | `"disabled"` | P2（代理 → MySQL）TLS 策略，镜像 MySQL 客户端 `--ssl-mode` 四档语义。                                          |
| `tls_p2_ca_file`      | 字符串 | `""`         | P2 自定义 CA **文件**（**仅** `verify_ca` / `verify_identity`）。PEM（可多证）或 DER；路径相对 `conf.json` 所在目录。  |
| `tls_p2_ca_path`      | 字符串 | `""`         | P2 自定义 CA **目录**（**仅** verify 模式）。目录内 PEM/DER 文件（**非递归**）；可与 `tls_p2_ca_file` 同时使用（合并信任）。     |


**P1 证书格式（`tls_p1_`*）：**


| 格式          | 配置                                           | 说明                                                                                       |
| ----------- | -------------------------------------------- | ---------------------------------------------------------------------------------------- |
| PFX / P12   | `tls_p1_pfx_file` + 可选 `tls_p1_pfx_password` | **推荐**；可含完整链                                                                             |
| PEM 证书 + 私钥 | `tls_p1_cert_file` + `tls_p1_key_file`       | 私钥：`BEGIN RSA PRIVATE KEY` 或 PKCS#8 `BEGIN PRIVATE KEY`（Windows 上 EC 等 exotic 密钥仍建议 PFX） |
| 运行时自签名      | 不配 `tls_p1_`*                                | RSA 自签，CN=`tls_p1_cert_cn` 或 `DBPrisma`                                                  |
| —           | `tls_p1_pfx_file` 与 PEM 成对                   | **互斥**，不可同时配置                                                                            |


**P2 信任来源（`verify_ca` / `verify_identity`）：**


| 来源        | 配置                    | 说明                                                   |
| --------- | --------------------- | ---------------------------------------------------- |
| OS 信任库    | 均省略                   | Windows 证书存储 / Linux `/etc/ssl/certs` 等              |
| 自定义 CA 文件 | `tls_p2_ca_file`      | 类似 `mysql --ssl-ca`；配置后**仅**信任该文件（及可选目录），**不用** OS 库 |
| 自定义 CA 目录 | `tls_p2_ca_path`      | 类似 `mysql --ssl-capath`；非递归扫描                        |
| 不支持       | PFX/P12、JKS、mTLS 客户端证 | P2 不向 MySQL 出示代理客户端证书                                |


`**backend_ssl_mode` 可选值：**


| 值                 | 加密  | 验证证书链 | 验证主机名 | 说明                                                                |
| ----------------- | --- | ----- | ----- | ----------------------------------------------------------------- |
| `disabled`        | ✗   | ✗     | ✗     | 明文（默认）                                                            |
| `preferred`       | ✓   | ✗     | ✗     | 优先 TLS；后端不宣告 SSL 或握手失败时降级明文并 WARN                                 |
| `required`        | ✓   | ✗     | ✗     | 强制加密但不验证后端证书；TLS 启动失败时降级明文并 WARN                                  |
| `verify_ca`       | ✓   | ✓     | ✗     | 验证 CA 链；默认 OS 信任库；配 `tls_p2_ca_file` / `tls_p2_ca_path` 时仅信任自定义来源 |
| `verify_identity` | ✓   | ✓     | ✓     | 完整验证；hostname 匹配 `host`；可配 `tls_p2_ca_file` / `tls_p2_ca_path`    |


- `verify_ca` / `verify_identity` 启动时若 P2 验证凭据初始化失败，`**start` 整体失败**（fail-closed）。
- `verify_ca` / `verify_identity` 下若后端不宣告 `CLIENT_SSL` 或 P2 TLS 握手失败，**拒绝连接**（fail-closed，不降级明文）。
- `tls_p2_ca_file` / `tls_p2_ca_path` 在 `disabled` / `required` 下会被 **WARN 并忽略**（`checkconfig`）。
- 代理**不会**把 P1 的 PFX/PEM 当作客户端证书发给 MySQL（当前无 mTLS）。

**`backend_ssl_mode` 的作用范围（按后端角色）：** `backend_ssl_mode` 是**逐 `servers[]` 实例**配置，按实例在拓扑中的角色生效：

| 后端角色 | 适用模式 | 连接来源 | P2 TLS |
| --- | --- | --- | --- |
| 会话主后端（客户端会话的承载后端） | 所有 strategy（含 `dispatch` 主、`read_write` `role=write` 主） | 会话 `server_sock` | ✓ 支持（按该实例 `backend_ssl_mode`）|
| 只读从库（`role=read`） | `read_write` | 惰性只读 `read_sock` | ✓ 支持（按该从库实例 `backend_ssl_mode`，**2026-06 新增**）|
| 影子库（shadow） | `dispatch` 广播 | 影子异步连接 | ✓ 支持（按该影子实例 `backend_ssl_mode`，**2026-06 新增**）|

> 即：从库与影子均可独立配置自己的 `backend_ssl_mode` / CA（与主库不同）。**`dispatch` 影子的 TLS 注意点**：TLS 影子不复用 DB 预热池（预热 socket 已是明文且过了 greeting），改为冷连接 + TLS 握手；且 TLS 影子绕过 SOCKET 键的异步发送队列，统一在 `send_lock` 下「加密 + 阻塞发送」以保证 TLS 记录顺序。明文影子（`disabled`）发送路径与性能不变。

未配置 `tls_p1_`* 文件时行为与旧版相同（运行时自签名）。配置了路径但加载失败时，`**start` 整体失败**（fail-closed）。

**行为对比：**


| 客户端 ssl-mode                    | `tls_termination = false`（默认） | `tls_termination = true` |
| ------------------------------- | ----------------------------- | ------------------------ |
| `DISABLED`                      | 明文，正常工作                       | 明文，正常工作                  |
| `PREFERRED`                     | 明文（代理剥除 CLIENT_SSL）           | 客户端选择明文或 TLS 均可工作        |
| `REQUIRED`                      | **连接失败**（握手错误）                | **正常工作**（代理终结 TLS）       |
| `VERIFY_CA` / `VERIFY_IDENTITY` | 失败                            | 自签名：失败；配置真实 CA 证书后：正常    |


**注意事项：**

- 需要客户端在连接时加 `--ssl-mode=REQUIRED`（或等效参数），也可接受 `PREFERRED`（代理已声明支持 SSL）。
- 使用自签名时，对 `VERIFY_CA` / `VERIFY_IDENTITY` 需关闭校验或导入信任；配置 `tls_p1_pfx_file` / PEM 后可用正式证书。
- 此功能不影响方案 B（透明代理）及任何其他已有功能。
- 示例配置：
  ```json
  {
    "id": 1,
    "host": "127.0.0.1",
    "port": 3306,
    "listen_port": 33006,
    "scheme": "auth",
    "user":  "root",
    "password": "secret",
    "tls_termination": true
  }
  ```

### 2.4 预热连接字段


| 配置项                      | 类型  | 默认值 | 取值范围       | 适用模式        | 说明                                                                  |
| ------------------------ | --- | --- | ---------- | ----------- | ------------------------------------------------------------------- |
| `warmup_tcp_connections` | 整数  | `0` | `0` ~ `32` | **已弃用** | **裸 TCP 预热已退役**（是后端 `ERROR 1129 (Host blocked)` 的根因）。该字段保留仅为兼容：在「认证代理 + 后端明文」时**折算为认证预热条数**（等价于 `warmup_db_connections`）；其余情况（透明代理 / 后端 TLS）被忽略并改用冷连接。新配置请直接用 `warmup_db_connections` |
| `warmup_db_connections`  | 整数  | `0` | `0` ~ `32` | **仅认证代理**   | MySQL 握手预建连接池大小（上限 32）。`0` = 禁用。提前完成完整 MySQL 握手，客户端连接时跳过握手延迟，速度最快   |


> **注意：** `warmup_db_connections` 即使在透明代理下填写了正整数，也不会生效，代码会自动忽略。
>
> **关于 `ERROR 1129`：** 旧版的「裸 TCP 预热」会向后端建立**未完成认证**的连接并在数秒后丢弃，MySQL 将其计入来源 IP 的连接错误，累计超过 `max_connect_errors`（默认 100）即拉黑代理主机，需反复 `mysqladmin flush-hosts`。现已退役该机制：有固定凭据且后端明文时改用**认证预热**（成功握手反而清零计数）；否则用冷连接（真实流量的成功握手自愈）。`warmup_tcp_connections` 显式 > 0 时，`checkconfig` 与启动日志会给出弃用提示。
> **注意：** `tls_termination` 与 `warmup_db_connections` 可同时使用；DB 预热握手发出的合成 Greeting 会自动加上 CLIENT_SSL，客户端可透明升级到 TLS。

---

## 三、`mode` 段（负载均衡统一入口，可选）

启用后，代理在 `listen_port` 上开启一个额外的统一监听端口，按 `strategy` 策略将客户端连接自动分发到 `servers` 数组中的各后端。`servers` 中的各实例独立监听端口仍然有效，`mode` 与各实例**并行工作**，互不影响。

```json
"mode": {
  "enabled":      false,
  "listen_ip":    "0.0.0.0",
  "listen_port":  3300,
  "strategy":     "round_robin"
}
```


| 配置项           | 类型  | 默认值             | 说明                                    |
| ------------- | --- | --------------- | ------------------------------------- |
| `enabled`     | 布尔  | `false`         | `true` 才启动统一监听，`false` 时整个 `mode` 段忽略 |
| `listen_ip`   | 字符串 | `"0.0.0.0"`     | 统一入口监听 IP                             |
| `listen_port` | 整数  | `3300`          | 统一入口监听端口（客户端连此端口，代理按策略路由）             |
| `strategy`    | 字符串 | `"round_robin"` | 负载均衡策略，见下表                            |


**strategy 可选值：**


| 值               | 说明                                                                                                 |
| --------------- | -------------------------------------------------------------------------------------------------- |
| `"round_robin"` | 轮询（默认），依次将请求分发到各后端                                                                                 |
| `"weighted"`    | 权重轮询，按各 server 的 `weight` 字段比例分配                                                                   |
| `"least_conn"`  | 最少连接，优先选择当前活跃连接数最少的后端                                                                              |
| `"dispatch"`    | 分发模式，写操作同步发给所有后端（多后端写广播）                                                                           |
| `"read_write"`  | 读写分离：写走 `role=write` 主库，读走 `role=read` 从库池（见 [read-write-split 设计](../design/read-write-split.md)） |


`**read_write` 专用字段（`strategy` 为 `read_write` 时生效）：**


| 配置项                          | 类型  | 默认值             | 说明                                               |
| ---------------------------- | --- | --------------- | ------------------------------------------------ |
| `read_strategy`              | 字符串 | `"round_robin"` | 读库池内策略：`round_robin` / `least_conn` / `weighted` |
| `read_after_write_sticky_ms` | 整数  | `0`             | DML 后 N 毫秒内 SELECT 强制走主库（0=关闭）                   |
| `read_fallback_to_primary`   | 布尔  | `false`         | 读库连接失败时是否静默回落主库（默认否，返回 45001）                    |
| `read_health_skip_failures`  | 整数  | `3`             | 读库连续连接失败 N 次后暂时跳过（0=关闭；与 `resilience` 探测协同）      |


`**dispatch` 专用字段：**


| 配置项                     | 类型  | 默认值     | 说明                                              |
| ----------------------- | --- | ------- | ----------------------------------------------- |
| `dispatch_primary_only` | 布尔  | `false` | **应急**：写只打 primary，跳过 shadow 广播；支持 `**reload`** |


以上 read_write 四项与 `dispatch_primary_only` 支持 `**reload**`；`mode.strategy` / `listen_*` 变更仍需重启。

**SQL 路由提示（注释，可选）：** `/* master */` 强制主库；`/* slave */` 或 `/* read */` 强制读库（事务内仍走主库）。

`**dispatch` 与 `primary`：** 可在某一 `servers[]` 项上设置 `"primary": true`，使该实例始终作为客户端连接的主后端（响应来自此节点）；其余方案 A 实例为影子。不配置 `primary` 时行为与原先相同（每条新连接轮询选主）。至多一个 `primary`；且须 `enabled: true`。

> **dispatch 最少后端数：** `strategy=dispatch` 要求至少 **2** 个 enabled 的 scheme A 后端（1 primary + ≥1 shadow）。不满足时 `checkconfig` 输出 `**[ERROR]`**，`start` 失败——单节点 dispatch 等同于单写转发，策略本身失去意义。

**影子就绪与写 SQL：** 连接建立后，影子后端在后台异步建连。对 **DML/DDL**（`INSERT`/`UPDATE`/`DELETE`/`CREATE` 等），代理会先等待影子就绪（上限 **3s**，由 `DISPATCH_SHADOW_WAIT_CAP_MS` 控制）；超时或建连结束后仍不足全部影子时，向客户端返回 **45000** 错误，**不会**只写 primary。`SELECT` 等读操作不等待影子。

> **注意：** `mode` 统一入口**仅支持 scheme A（`scheme: "auth"`）后端**。`scheme: "transparent"` 的 server 在 `checkconfig` / `start` 时输出 **Warning** 并被排除在 mode 路由池之外；若 enabled 的 auth 后端数量为 0，则 **启动失败**。各 server 的独立 `listen_port` 仍按自身 `scheme` 正常工作，与 `mode` 互不影响。

---

## 四、`resilience` 段（可选，后端健康）

```json
"resilience": {
  "enabled": true,
  "probe_interval_sec": 10,
  "probe_timeout_ms": 2000,
  "probe_failure_threshold": 3,
  "probe_recovery_threshold": 2,
  "connect_backoff_base_ms": 100,
  "connect_backoff_max_ms": 30000,
  "connect_backoff_multiplier": 2.0,
  "pause_warmup_when_unhealthy": true,
  "backend_tcp_keepalive": true,
  "dns_rotate_on_failure": false,
  "transparent_reconnect": false
}
```


| 字段                            | 类型    | 默认值     | 说明                                          |
| ----------------------------- | ----- | ------- | ------------------------------------------- |
| `enabled`                     | bool  | `true`  | 是否启用后端健康探测；`false` 时所有后端视为永久 healthy        |
| `probe_interval_sec`          | int   | `10`    | 健康探测间隔（秒，≥1）                                |
| `probe_timeout_ms`            | int   | `2000`  | 单次探测连接超时（毫秒）                                |
| `probe_failure_threshold`     | int   | `3`     | 连续失败 N 次后将后端标记为 DOWN（≥1）                    |
| `probe_recovery_threshold`    | int   | `2`     | DOWN 后连续成功 N 次后标记恢复为 UP（≥1）                 |
| `connect_backoff_base_ms`     | int   | `100`   | 重连退避基础等待（毫秒）                                |
| `connect_backoff_max_ms`      | int   | `30000` | 重连退避最大等待（毫秒）                                |
| `connect_backoff_multiplier`  | float | `2.0`   | 退避指数倍增系数（`>= 1.0`）                          |
| `pause_warmup_when_unhealthy` | bool  | `true`  | 后端不健康时暂停连接池预热                               |
| `backend_tcp_keepalive`       | bool  | `true`  | 是否对后端连接开启 TCP Keepalive（**不支持 reload，需重启**） |
| `dns_rotate_on_failure`       | bool  | `false` | 连接失败时轮换 DNS 解析结果（适用于多 A 记录域名）               |
| `transparent_reconnect`       | bool  | `false` | 后端重连时对客户端透明（会话恢复）                           |


> `backend_tcp_keepalive` 需 **重启**，其余字段均支持 `**reload`**。`enabled=false` 时所有后端视为 healthy。省略整个 `resilience` 段时默认 `enabled=true`（健康探测默认开启）。
> 更多弹性字段（keepalive 参数、重试、故障转移、会话泄漏检测）见 [§二十 弹性扩展字段](#二十弹性扩展字段resilience-新增)。

---

## 五、`logging` 段

```json
"logging": {
  "file":          "DBPrisma.log",
  "level":         "info",
  "max_size_mb":   50,
  "rotate_count":  3
}
```

> **注：** 上例 `max_size_mb=50` 是示例选值（与 `bin/<Platform>/<arch>/config/conf.json` 模板一致，便于生产观测）；省略字段时的真实默认为 `10`，见下表。


| 配置项            | 类型  | 默认值              | 说明                                             |
| -------------- | --- | ---------------- | ---------------------------------------------- |
| `file`         | 字符串 | `"DBPrisma.log"` | 日志文件名（相对路径锚定 `log_dir`，默认 `<exe>/logs/`）或绝对路径  |
| `level`        | 字符串 | `"info"`         | 日志级别：`debug` / `info` / `warn` / `error`（从详到简） |
| `max_size_mb`  | 整数  | `10`             | 单个日志文件大小上限（MB）。`0` = 不限制                       |
| `rotate_count` | 整数  | `3`              | 日志轮转保留份数。`0` = 不备份，达到上限后直接截断                   |


---

## 六、完整示例

### 示例 A：透明代理（最简配置）

```json
{
  "servers": [
    {
      "scheme": "transparent",
      "host": "127.0.0.1",
      "port": 3306,
      "listen_port": 33006
    }
  ]
}
```

### 示例 B：认证代理 + DB 握手预热

```json
{
  "proxy": {
    "worker_threads":     0,
    "max_clients":        1024,
    "connect_timeout_ms": 1000,
    "idle_timeout_sec":   300
  },
  "servers": [
    {
      "id":                    1,
      "scheme":                "auth",
      "host":                  "127.0.0.1",
      "port":                  3308,
      "listen_ip":             "0.0.0.0",
      "listen_port":           33006,
      "enabled":               true,
      "user":                  "root",
      "password":              "123qwe",
      "warmup_db_connections": 4
    }
  ],
  "logging": {
    "file":         "DBPrisma.log",
    "level":        "info",
    "max_size_mb":  50,
    "rotate_count": 3
  }
}
```

### 示例 C：多实例（一个透明 + 一个认证）

```json
{
  "proxy": {
    "connect_timeout_ms": 1000
  },
  "servers": [
    {
      "id":          1,
      "scheme":      "transparent",
      "host":        "127.0.0.1",
      "port":        3306,
      "listen_port": 33006
    },
    {
      "id":                    2,
      "scheme":                "auth",
      "host":                  "127.0.0.1",
      "port":                  3308,
      "listen_port":           33007,
      "user":                  "root",
      "password":              "ENC2:rJ3kLmP9...",
      "warmup_db_connections": 4
    }
  ],
  "logging": {
    "level": "warn"
  }
}
```

### 示例 D：P2 验证后端 + 自定义 CA（文件 + 目录）

```json
{
  "servers": [{
    "id": 1,
    "scheme": "auth",
    "host": "192.168.10.34",
    "port": 3306,
    "listen_port": 33006,
    "tls_termination": true,
    "tls_p1_pfx_file": "hellodata.pfx",
    "backend_ssl_mode": "verify_ca",
    "tls_p2_ca_file": "certs/mysql-ca.pem",
    "tls_p2_ca_path": "certs/ca-dir",
    "user": "root",
    "password": "ENC2:..."
  }]
}
```

### 示例 E：`read_write` + 从库 P2 TLS（2026-06 新增）

写主库走会话承载后端（其 `backend_ssl_mode` 走会话 P2 路径）；`role=read` 从库**各自**按自己的 `backend_ssl_mode` 建立 P2 TLS。下例：主库明文、从库强制 TLS（`required`，不验证证书，最易联调）。需校验证书时把从库改为 `verify_ca` 并配 `tls_p2_ca_file`。

```json
{
  "mode": {
    "enabled": true,
    "listen_ip": "0.0.0.0",
    "listen_port": 3307,
    "strategy": "read_write"
  },
  "servers": [
    {
      "id": 1, "scheme": "auth", "role": "write",
      "host": "10.0.0.10", "port": 3306,
      "user": "root", "password": "ENC2:...",
      "backend_ssl_mode": "disabled"
    },
    {
      "id": 2, "scheme": "auth", "role": "read",
      "host": "10.0.0.11", "port": 3306,
      "user": "root", "password": "ENC2:...",
      "backend_ssl_mode": "required"
    },
    {
      "id": 3, "scheme": "auth", "role": "read",
      "host": "10.0.0.12", "port": 3306,
      "user": "root", "password": "ENC2:...",
      "backend_ssl_mode": "verify_ca",
      "tls_p2_ca_file": "certs/mysql-ca.pem"
    }
  ]
}
```

> 联调验证：发只读 SQL（普通 `SELECT`、`COM_STMT_PREPARE`/`EXECUTE`、跨多记录的大结果集），观察日志出现 `Backend <host>: P2 TLS established`，且结果正常返回、无解密错误/连接重置。会话主后端、`read_write` 从库与 `dispatch` 影子均支持 P2 TLS（各按自身 `backend_ssl_mode`）。`dispatch` 影子建连成功时日志出现 `Dispatch: shadow[i] connected ...`；TLS 握手失败则为 `Dispatch: shadow TLS auth failed ...`。

---

## 七、配置编辑命令 `setconfig`

不启动服务，编辑 `conf.json`（服务运行中禁止写入）。写盘前备份为 `.bak`，写后自动 `checkconfig` 校验。

```text
# 查看配置（只读）
setconfig show [all|version|proxy|admin|resilience|startup|shutdown|mode|logging|notify
               |security|capture|circuit_breaker|watchdog|tls|log_config|config_manage|paths
               |command_pipe|server [id]]

# 编辑各段（服务停止时）
setconfig version                    # 编辑 version（semver "0.0.1"；引擎完整版 0.0.1.568）
setconfig proxy                      # 代理核心选项（超时、redact、legacy_pwd）
setconfig admin                      # Admin HTTP 端点选项（独立向导）
setconfig resilience                 # 后端健康探测与退避选项
setconfig startup                    # 预热与 pre-flight 选项
setconfig shutdown                   # 优雅排水与截止时间选项
setconfig mode                       # 均衡策略、read_write 子字段、dispatch 子字段
setconfig logging                    # 日志文件、级别、轮转（相对 file 锚定 log_dir）
setconfig notify                     # OS 通知与事件开关
setconfig paths                      # 运行期目录 run_dir/log_dir/capture_dir（改后须 restart）
setconfig security                   # IP 封禁、防火墙、连接限制（改后须 restart）
setconfig capture                    # SQL 捕获字符集选项（改后须重启）
setconfig circuit_breaker            # 熔断器参数（改后须重启）
setconfig watchdog                   # 看门狗参数（改后须重启）
setconfig tls                        # 全局 TLS 版本与密码套件（改后须重启）
setconfig log_config                 # 日志格式/慢查询扩展（改后须重启）
setconfig config_manage              # 热重载备份/回滚策略（改后须重启）
setconfig command_pipe               # 带外管理控制通道（enabled/pipe_name/local_only/
                                     #   max_instances/session_idle_timeout_sec/auth_store/
                                     #   capture_queue_mb/capture_buf_kb/
                                     #   allowed_commands；改后须 restart）

# server 管理
setconfig server <id>                # 无则创建，交互修改（含 role 字段）
setconfig server add
setconfig server del <id>
setconfig srvuser <id> <user>        # 设后端 user（'-' 删除）
setconfig srvpwd <id>                # 设加密后端密码

# 配置文件控制
setconfig use [path]                 # 本会话默认编辑该文件
setconfig use default                # 恢复 bin\Windows\x64\config\conf.json
setconfig -f <path> <子命令>         # 对单条命令临时指定配置文件
```

运行中改盘：`setconfig` 会被拒绝；请 `stop` 后编辑，或只改可热更字段后执行 `**reload**`（见 §九）。

`setconfig use D:\backup\conf.json` 之后，直接 `setconfig server add` 等即操作该文件；`-f` 仍可对单条命令临时指定其它路径。

交互提示：`Enter` 保留原值，`-` 从 JSON 删除该字段。`server` 向导不含 `user`/`password`，后端凭据用 `setconfig srvuser` / `setconfig srvpwd`。代理用户仍用 `useradd` / `userdel` / `userlist`（`proxy_users.json`）。

> **全段覆盖**：`setconfig` 支持全部顶层配置段（`proxy`/`admin`/`resilience`/`startup`/`shutdown`/`mode`/`logging`/`security`/`capture`/`circuit_breaker`/`watchdog`/`tls`/`log_config`/`config_manage`/`notify`/`paths`/`command_pipe`/`server`）。注意 security/capture/circuit_breaker/watchdog/tls/config_manage/notify/command_pipe 修改后需 **重启**（非 reload）才能生效；`log_config.log_format`/`log_include_sql` 同样须重启；`**log_config.slow_query_ms`**/`**slow_query_file**` 支持 `**reload**` 热更；`**log_config.firewall_log_file**`/`**firewall_log_max_size_mb**`/`**firewall_log_rotate_count**` 支持 `**reload**` 热更；`**log_config.audit_export_enabled**`/`**audit_export_format**`/`**audit_export_file**`/`**audit_chain_enabled**`/`**audit_chain_key**` 支持 `**reload**` 热更。

---

## 八、配置检查命令

不启动代理服务，仅校验配置文件：

```text
DBPrisma.exe checkconfig
DBPrisma.exe checkconfig D:\path\to\conf.json
```

交互模式下也可输入 `checkconfig` 或 `cc`（可选配置文件路径）。

检查项包括：JSON 解析、密码解密、`version`、`primary` 唯一性、端口范围、重复 `id`、监听地址冲突、`mode` 与 `servers` 端口冲突、`**mode.enabled` 时须至少一个 enabled 的 scheme A 后端**（transparent 计入 Warning）、方案 A 是否配置 `user`、`dispatch` 影子后端、`proxy_users.json` 等。存在 `[ERROR]` 时退出码为 1。

### 8.1 审计链校验 `checkaudit`

不启动代理，校验 `log_config.audit_export_file` 导出的 NDJSON 哈希链（须含 `prev_hash`/`line_hash` 行）：

```text
DBPrisma.exe checkaudit
DBPrisma.exe checkaudit D:\path\to\audit_export.ndjson
DBPrisma.exe checkaudit D:\path\to\conf.json
```

省略路径时读取当前 `conf.json` 并解析 `paths.log_dir` + `audit_export_file`。无哈希字段的行计为 `skipped`，不导致失败。

---

## 九、热加载 `reload`

服务**已运行**时，将磁盘上的 `conf.json` 与内存配置对比并应用**可热更**子集（失败则**整次不变**）：

`reload` 是 XR:> 提示符下的运行时命令（非 Shell 子命令）：

```text
XR:> reload
XR:> reload D:\path\to\conf.json
```


| 可 `reload`                                                                                               | 不可 `reload`（须 `stop` → `start`）                                          |
| -------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| `logging.level`                                                                                          | `servers[]` 增删、`id`、端口、`enabled`、`scheme`                                |
| `logging.max_size_mb` / `rotate_count`                                                                   | `mode.strategy`、`mode.listen_*`、`servers[].role`                         |
| `proxy.idle_timeout_sec`（调整已有值）¹                                                                         | `proxy.max_clients`、`worker_threads`、`max_reassemble_bytes`              |
| `proxy.connect_timeout_ms`                                                                               | `tls_p1_*`、`backend_ssl_mode`、`tls_p2_ca_file`、`tls_p2_ca_path`          |
| `proxy.capture_redact_literals`                                                                          | `version`                                                                |
| `proxy.allow_legacy_password`                                                                            | 监听 IP/端口、P1/P2 TLS 证书与 CA 路径                                             |
| `mode.read_strategy`、`read_after_write_sticky_ms`、`read_fallback_to_primary`、`read_health_skip_failures` | `proxy.max_clients`、`servers[].user`/`password`/`scheme`                 |
| `mode.dispatch_primary_only`                                                                             | `resilience.backend_tcp_keepalive`                                       |
| `admin_bearer_token`、`admin_allow_ips`、`admin_request_timeout_ms`、`admin_max_header_bytes`               | `admin_listen_ip`、`admin_listen_port`、`admin_tls_*`（证书/私钥/口令，须重启）        |
| `resilience.*`（除 `backend_tcp_keepalive`）                                                                | `**security.***`（须重启）                                                    |
| `proxy.startup.*`、`proxy.shutdown.*`                                                                     | `**capture.***`（须重启）                                                     |
| `**log_config.slow_query_ms**`、`**log_config.slow_query_file**`                                          | `**circuit_breaker.***`（须重启）                                             |
| `**log_config.firewall_log_file**`、`firewall_log_max_size_mb`、`firewall_log_rotate_count`                | `**watchdog.***`（须重启）                                                    |
| —                                                                                                        | `**tls.***`（须重启）                                                         |
| —                                                                                                        | `**log_config.log_format**`、`log_include_sql`（须重启）                       |
| —                                                                                                        | `**paths.***`（`run_dir`/`log_dir`/`capture_dir`，须 restart）               |
| —                                                                                                        | `**config_manage.***`（配置段本身须 restart；`backup_on_reload` 在 reload 流程中已消费） |
| —                                                                                                        | `**notify.***`（须 restart）                                                |
| —                                                                                                        | `**command_pipe.***`（须 restart；管道服务在启动时按配置创建）                            |


> ¹ `**idle_timeout_sec` 特别说明**：见下方注。

> ⚠️ `**idle_timeout_sec` 特别说明**：`reload` 只能调整已运行空闲计时线程的**阈值**（即启动时 `idle_timeout_sec > 0`）。若启动时该值为 `0`（计时线程未创建），`reload` 将该值改为非零后**不会启动线程**，日志会输出 WARN 提示；须执行 `stop` → `start` 才能激活空闲超时功能。
>
> ⚠️ `**security` 须重启**：`setconfig security` 写盘后必须执行 `stop` → `start` 才能使新的安全配置生效；`reload` **不会**更新内存中的安全策略（IP 封禁、熔断阈值等）。

`useradd` / `setpwd` 后：`proxy_users.json` 由 `**PA_ReloadProxyUsers`** 生效（无需重启，也无需 `reload conf.json`）。

---

## 十、`proxy_users.json` 配置（认证代理专用）

路径：`bin/<Platform>/<arch>/config/proxy_users.json`

当认证代理（Scheme A）的 `proxy_users.json` 存在且非空时，只有列表中的用户才能通过代理认证。文件为空或不存在时，允许任意用户名/密码（后端账号凭据以 `conf.json` 中的 `user`/`password` 为准）。

```json
{
  "users": [
    { "username": "alice", "password": "明文密码" },
    { "username": "bob",   "password": "ENC2:加密密码" }
  ]
}
```


| 字段         | 说明                               |
| ---------- | -------------------------------- |
| `username` | 客户端连接代理时使用的用户名                   |
| `password` | 对应密码，支持明文或 `ENC2:` 加密格式 |


> 代理用户与后端 MySQL 用户**完全独立**。客户端用 `alice` 连代理，代理实际以 `conf.json` 中配置的后端账号连 MySQL。明文密码会打 `[SECURITY WARN]`，建议 `setpwd` 加密。

---

## 十一、CLI 管理 token（`dbprisma.cli_token`）

路径：`<exe>/run/dbprisma.cli_token`（默认；随 `--run-dir`/`DBPRISMA_HOME`/`paths.run_dir` 变化）


| 项       | 说明                                                                            |
| ------- | ----------------------------------------------------------------------------- |
| 生成      | 首次 `start` 时创建 32 字节随机 hex                                                    |
| 权限      | 仅进程属主可读（Windows DACL / POSIX 0600）                                            |
| 保护命令    | `stop`、`restart`、`reload`、`setconfig`、`passwd`、`setpwd`、`useradd`、`userdel` 等 |
| 交互式 TTY | 控制台人工操作可省略 token                                                              |
| 脚本      | `--token <hex>` 或环境变量 `DBPRISMA_CLI_TOKEN`                                    |


只读命令（如 `status`）不需要 token。

---

## 十二、命令行参数与交互命令参考

### 12.1 进程级启动参数

**用法：** `DBPrisma [OPTIONS] [SUBCOMMAND [args...]]`


| 参数                    | 别名   | 是否带值 | 说明                                                                       |
| --------------------- | ---- | ---- | ------------------------------------------------------------------------ |
| `--config <file>`     | `-c` | 是    | 指定配置文件路径（默认：`<config-dir>/conf.json`）                                    |
| `--config-dir <dir>`  | —    | 是    | `conf.json`/`proxy_users.json`/备份所在目录（默认 `<exe>/config/`）                |
| `--run-dir <dir>`     | —    | 是    | `lock`/`pid`/`cli_token`/`cli_history`/`machine.key` 目录（默认 `<exe>/run/`） |
| `--log-dir <dir>`     | —    | 是    | 主日志/防火墙/慢查询日志目录（默认 `<exe>/logs/`）                                        |
| `--capture-dir <dir>` | —    | 是    | `capture.json` 目录（默认 `<exe>/capture/`）                                   |
| `--token <hex>`       | —    | 是    | CLI 鉴权 token（64位 hex，32字节）；非交互模式的特权命令需要；也可通过环境变量 `DBPRISMA_CLI_TOKEN` 设置 |
| `--log-level <level>` | —    | 是    | 覆盖配置文件中的日志级别（`debug`|`info`|`warn`|`error`），仅本次运行生效                      |
| `--bg-capture`        | —    | 否    | 启动时立即开始后台 SQL 捕获到 `capture.json`                                         |
| `--no-interactive`    | —    | 否    | 守护进程模式：跳过 `XR:>` 提示符；Ctrl+C 或 stop 时正常退出                                 |
| `--single-instance`   | —    | 否    | 若另一个 DBPrisma 实例已运行则拒绝启动                                                 |
| `--pid-file <path>`   | —    | 是    | 服务成功启动后将 PID 写入文件（默认 `<run-dir>/dbprisma.pid`）                           |
| `--print-config`      | —    | 否    | 加载 conf.json 打印生效配置后退出                                                   |
| `--warmup-only`       | —    | 否    | 运行预检 + 预热后退出（CI/就绪检查）                                                    |
| `--version`           | `-v` | 否    | 打印版本信息后退出                                                                |
| `--help`              | `-h` | 否    | 打印帮助后退出                                                                  |


> **参数语法**：`checkconfig`/`cc`、`setconfig`/`sc`、`service` 必须作为**第一个参数**（位于所有选项之前），否则报错退出（`subcommand '<x>' must come before options`）；`start` 不受此限，选项可置于其前或其后。带值参数同时接受 `--opt value` 与 `--opt=value`；`--` 表示选项结束。无法识别的选项或拼错的子命令会**报错退出**并给出最接近的拼写建议。

**进程级子命令（立即执行后退出）：**


| 子命令                                             | 别名   | 参数     | 说明                     |
| ----------------------------------------------- | ---- | ------ | ---------------------- |
| `start [config_file]`                           | —    | 可选配置路径 | 启动代理服务                 |
| `checkconfig [config_file]`                     | `cc` | 可选配置路径 | 仅校验配置，不启动服务            |
| `setconfig <verb> [args...]`                    | `sc` | 见 §七   | 编辑磁盘上的配置文件后退出          |
| `service <install|uninstall|start|stop|status>` | —    | —      | 管理操作系统服务（需管理员/root 权限） |


### 12.2 交互模式命令（`XR:>` 提示符）


| 命令                 | 参数                                            | 需要 token | 说明                                                                                              |
| ------------------ | --------------------------------------------- | -------- | ----------------------------------------------------------------------------------------------- |
| `start`            | `[config_file]`                               | 否        | 启动代理（默认 `conf.json`）                                                                            |
| `stop`             | —                                             | **是**    | 优雅停止代理（等同于 `shutdown 0`）                                                                        |
| `restart`          | `[config_file]`                               | **是**    | 优雅重启（stop + start）                                                                              |
| `reload`           | `[config_file]`                               | **是**    | 热加载可更新字段（见 §九）                                                                                  |
| `shutdown`         | `[<秒> [force|cancel]]`                        | **是**    | 带倒计时的优雅停止；`shutdown 30` 表示 30 秒后停止；`shutdown 0 force` 立即强停；`shutdown cancel` 取消等待中的 shutdown    |
| `status`           | `[verbose]`                                   | 否        | 运行时统计；`verbose` 显示详细版本、模式、后端、连接池信息                                                              |
| `backends`         | —                                             | 否        | 列出所有后端服务器状态（在线/离线、连接数、熔断状态）                                                                     |
| `drain`            | `<server_id>` | `cancel <server_id>`          | **是**    | 按后端排空：`drain <id>` 停止向该后端路由新连接（存量连接自然结束）；`drain cancel <id>` 恢复正常路由。必须指定 `server_id`（无"全部排空"形式） |
| `pause`            | —                                             | **是**    | 暂停接受新客户端连接（不断开已有连接）                                                                             |
| `resume`           | —                                             | **是**    | 恢复接受客户端连接（解除 pause）                                                                             |
| `flush`            | `[server_id]`                                 | **是**    | 清空后端**预热连接池**并后台重建：省略参数清空所有后端，指定 `server_id` 仅清空该后端（不断开活跃会话）                                    |
| `conns`            | `[ip]`                                        | 否        | 当前活跃客户端连接列表；可选按客户端 IP 过滤                                                                        |
| `kick`             | `<client_ip> [port]` | `id <n>`               | **是**    | 强制断开客户端；`port=0` 断开该 IP 所有连接；`id <n>` 断开 `conns` 第 N 行连接                                        |
| `loglevel`         | `<debug|info|warn|error>`                     | **是**    | 运行时变更日志级别                                                                                       |
| `capture`          | `[-s IP:PORT] [-c IP:PORT] [-t TYPES] [MODE]` | 否        | 实时 SQL 捕获（见下）                                                                                   |
| `display`          | —                                             | 否        | 显示当前加载的配置文件内容                                                                                   |
| `checkconfig`      | `[config_file]`                               | 否        | 校验配置文件语法与语义                                                                                     |
| `setconfig`        | `<verb> [args...]`                            | **是**    | 编辑配置文件（见 §七）                                                                                    |
| `useradd`          | `[--admin] <username>`                        | **是**    | 创建用户，交互输入密码；默认 `proxy_users.json`，加 `--admin` 写 `admin_users.json`（命令管道管理员）                     |
| `userdel`          | `[--admin] <username>`                        | **是**    | 删除用户（加 `--admin` 操作管理员库）                                                                        |
| `userlist`         | `[--admin]`                                   | 否        | 列出用户（加 `--admin` 列出命令管道管理员）                                                                     |
| `setpwd`           | `[--admin] <username>`                        | **是**    | 修改用户密码（加 `--admin` 操作管理员库）                                                                      |
| `passwd`           | `[明文]`                                        | **是**    | 将密码加密为 `ENC2:` 格式                                                                               |
| `token rotate`     | —                                             | **是**    | 重新生成 CLI 鉴权 token                                                                               |
| `service`          | `<install|uninstall|start|stop|status>`       | **是**    | 操作系统服务管理                                                                                        |
| `version`          | —                                             | 否        | 版本 / 平台 / 构建信息                                                                                  |
| `clear` / `cls`    | —                                             | 否        | 清屏                                                                                              |
| `history` / `hist` | —                                             | 否        | 列出命令历史（128 条环形缓冲；持久化到 `<run>/dbprisma.cli_history`，跨会话保留；↑/↓ 调取）                                |
| `help` / `?`       | `[command]`                                   | 否        | 显示全部命令；带命令名时显示该命令详细用法                                                                           |
| `exit` / `quit`    | —                                             | —        | 退出（服务运行中会先 stop）                                                                                |


`**capture` 选项：**


| 标志           | 说明                                                 |
| ------------ | -------------------------------------------------- |
| `-s IP:PORT` | 过滤指定后端 MySQL 服务器                                   |
| `-c IP:PORT` | 过滤指定客户端 IP:PORT                                    |
| `-t TYPES`   | SQL 类型过滤：`DDL,DML,DQL,DCL,TCL` 或 `all`（默认 all）     |
| `MODE`       | 输出模式：`screen`（默认）/ `json`（写 `capture.json`）/ `all` |


捕获界面：`P`/`p` = 暂停/恢复打印；`E`/`e` = 退出捕获模式。

### 12.3 环境变量

DBPrisma 仅读取以下环境变量（其余配置一律走 `conf.json` 或命令行参数）：


| 变量                   | 说明                                                                                                                                   |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| `DBPRISMA_CLI_TOKEN` | 非交互模式下特权命令的鉴权 token，等价于 `--token <hex>`                                                                                              |
| `DBPRISMA_HOME`      | 设为根目录后，四目录布局解析为 `$DBPRISMA_HOME/config|run|log|capture`；优先级低于 `--config-dir`/`-c` 与 `--*-dir`，高于 `conf.json` 的 `paths` 段及 `<exe>` 默认 |
| `NO_COLOR`           | 设置为任意值即禁用所有彩色终端输出（遵循 no-color.org 约定）                                                                                                |


---

## 十三、安全配置（`security`）

> ⚠️ `security` 段**不支持热重载**。`setconfig security` 修改磁盘配置后，须执行 `stop` → `start` 才能生效。


| 字段                         | 类型     | 默认值        | 说明                                                                                                                                                                                                                                                    |
| -------------------------- | ------ | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `auth_fail_max`            | int    | `5`        | 触发封禁的认证失败次数（1–100）                                                                                                                                                                                                                                    |
| `auth_fail_window_ms`      | int    | `60000`    | 滑动窗口时长（ms，≥1000）                                                                                                                                                                                                                                      |
| `auth_fail_ban_ms`         | int    | `300000`   | 封禁持续时长（ms，0=永久）                                                                                                                                                                                                                                       |
| `max_connections_per_ip`   | int    | `0`        | 单个 IP 并发连接上限（0=禁用）                                                                                                                                                                                                                                    |
| `max_connections_per_user` | int    | `0`        | 单个代理用户并发连接上限（Scheme A `proxy_client_user` / Scheme B 握手用户名；0=禁用）                                                                                                                                                                                      |
| `max_queries_per_user`     | int    | `0`        | 单用户滑动窗口内允许转发的 SQL 条数（0=禁用）                                                                                                                                                                                                                          |
| `query_rate_window_ms`     | int    | `60000`    | 查询速率统计窗口（ms，≥1000；`max_queries_per_user>0` 时生效）                                                                                                                                                                                                      |
| `new_conn_burst_limit`     | int    | `0`        | 单窗口内新建连接数上限（令牌桶，0=禁用）                                                                                                                                                                                                                                 |
| `new_conn_burst_window_ms` | int    | `1000`     | burst 窗口（ms）                                                                                                                                                                                                                                          |
| `query_timeout_ms`         | int    | `0`        | 单条 SQL 执行超时（ms，超时后关闭会话，0=禁用）                                                                                                                                                                                                                          |
| `long_query_warn_ms`       | int    | `0`        | 慢查询 WARN 阈值（ms，不关闭会话，0=禁用）                                                                                                                                                                                                                            |
| `max_packet_size_bytes`    | int    | `0`        | 客户端包最大字节（0=沿用 `max_reassemble_bytes`）                                                                                                                                                                                                                 |
| `hide_server_version`      | bool   | `false`    | **常开加固，无开关作用**：合成握手始终上报通用 MySQL 版本号；字段仅为兼容保留                                                                                                                                                                                                                          |
| `strip_client_attributes`  | bool   | `false`    | **常开加固，无开关作用**：转发至后端前始终剥除 `CLIENT_CONNECT_ATTRS` 载荷；代理向客户端握手仍协商该能力以便本地提取 `tool_name`（`program_name` / `_client_name`）；字段仅为兼容保留                                                                                                                                                                                                                      |
| `mask_client_ip_in_log`    | bool   | `false`    | 仅日志输出对客户端 IP 部分脱敏（IPv4→`A.B.x.x`，IPv6→`a:b::x`）；限流/`kick`/捕获等仍用真实 IP                                                                                                                                                                                  |
| `sql_firewall_enabled`     | bool   | `false`    | 启用轻量 SQL 防火墙（运行时完整生效）                                                                                                                                                                                                                                 |
| `sql_firewall_mode`        | string | `"blocklist"` | 规则语义：`blocklist`（命中即拦，默认，与旧版一致）| `allowlist`（仅允许命中规则的 SQL，须至少一条 `sql_block_patterns`）                                                                                                                                                                                          |
| `sql_firewall_action`      | string | `"reject"` | 拦截策略：`reject`（返回 MySQL 错误 1148 并丢弃命令）| `log_only`（仅记日志，放行）                                                                                                                                                                                            |
| `sql_firewall_log_blocked` | bool   | `true`     | 命中时记录 `FIREWALL_BLOCK` WARN 日志到主日志（含命中 pattern 和 SQL 前 120 字节）；配合 `log_config.firewall_log_file` 可同时写入独立审计文件                                                                                                                                          |
| `sql_block_patterns`       | array  | `[]`       | 全局阻断模式列表（最多 32 条，每条最长 256 字节）。默认**大小写不敏感子串匹配**；以 `regex:` 前缀启用受限正则（支持 `. * + ^ $ \s \d \w`，未锚定 `^` 时按子串匹配）。首条命中即触发。对 `COM_STMT_PREPARE` / `COM_STMT_EXECUTE` 两阶段检查（`fw_stmt_map`）。在 `ssl_passthrough` 模式下不可用。无匹配 scope 的 `sql_access_policies` 时回退本列表。 |
| `sql_access_policies`      | array  | `[]`       | 按用户/库细粒度策略（最多 `PA_MAX_SQL_ACCESS_POLICIES` 条）。每条：`users[]`、`databases[]`（空=全部）、`patterns[]`（同 `sql_block_patterns` 语义，含 `regex:`）、`mode`（`blocklist`/`allowlist`）、`action`（`reject`/`log_only`/`audit_only`）、`priority`（整数，默认 `0`，**多条 scope 同时命中时取最高**）。未命中任何 policy scope 时回退全局 `sql_block_patterns`。 |
| `access_schedule`          | object | 见下       | **访问时段**（本地系统时间）。`enabled=false` 时忽略；`enabled=true` 时窗口外 SQL 返回 **1227**。与防火墙独立，可单独启用。                                                                                                                                                   |
| `access_schedule.enabled`  | bool   | `false`    | 启用时段控制                                                                                                                                                                                                                                              |
| `access_schedule.weekdays` | array  | `[]`       | 允许的工作日：`0`=周日 … `6`=周六（与 `SYSTEMTIME.wDayOfWeek` 一致）；`enabled=true` 时须非空                                                                                                                                                                        |
| `access_schedule.start`    | string | `"00:00"`  | 窗口开始 `HH:MM`（含），须两位小时                                                                                                                                                                                                                               |
| `access_schedule.end`      | string | `"23:59"`  | 窗口结束 `HH:MM`（同日窗口为**不含**该分钟；`start==end` 表示全天）                                                                                                                                                                                              |
| `ip_allowlist`             | array  | `[]`       | 永不被限速/封禁的 IP/CIDR 列表（最多 64 条；支持 `10.0.0.0/8`、`2001:db8::/32` 等）                                                                                                                                                                                       |
| `ip_denylist`              | array  | `[]`       | 始终拒绝连接的 IP/CIDR 列表（最多 64 条）                                                                                                                                                                                                                           |


```json
"security": {
  "auth_fail_max": 5,
  "auth_fail_window_ms": 60000,
  "auth_fail_ban_ms": 300000,
  "max_connections_per_ip": 50,
  "query_timeout_ms": 30000,
  "long_query_warn_ms": 5000,
  "mask_client_ip_in_log": true,
  "ip_allowlist": ["10.0.0.0/8", "192.168.1.100"],
  "ip_denylist": ["1.2.3.4", "203.0.113.0/24"]
}
```

> **注**：`ip_allowlist` / `ip_denylist` 支持 **主机 IP**（如 `10.0.0.1`）或 **CIDR**（如 `192.168.1.0/24`、`2001:db8::/32`）。IPv4 与 IPv6 族必须一致（IPv4 CIDR 不匹配 IPv6 客户端）。`checkconfig` 会校验条目格式；黑白名单条目重叠时 WARN（运行时白名单优先）。

---

## 十四、SQL 捕获配置（`capture`）


| 字段                    | 类型     | 默认值      | 说明                                              |
| --------------------- | ------ | -------- | ----------------------------------------------- |
| `charset`             | string | `"auto"` | 捕获 SQL 使用的字符集：`auto`、`utf8mb4`、`gbk`、`latin1` 等 |
| `charset_auto_detect` | bool   | `true`   | 从 MySQL 服务端握手包中自动识别字符集（覆盖 `charset` 字段）         |


> `charset` / `charset_auto_detect` 已完整实现（TD-009 已偿还）：握手阶段从服务端 HandshakeV10 提取字符集名称，`agent_on_sql_captured` 按优先级（`charset_auto_detect` → 显式 `charset` → 降级 "utf8mb4"）填充 `PA_SqlRecord.charset`。
> `capture` 段所有字段须 `stop` → `start` 生效，不支持 `reload`。

---

## 十五、熔断器（`circuit_breaker`）

每个后端独立维护一个熔断器实例（状态机：`CLOSED→OPEN→HALF_OPEN→CLOSED`）。


| 字段                         | 类型     | 默认值        | 说明                                                         |
| -------------------------- | ------ | ---------- | ---------------------------------------------------------- |
| `enabled`                  | bool   | `false`    | 是否启用熔断器                                                    |
| `error_rate_threshold_pct` | int    | `50`       | 错误率阈值（%，1–100），超过则开路                                       |
| `min_requests`             | int    | `20`       | 评估前最少请求数（样本不足不触发开路）                                        |
| `window_ms`                | int    | `60000`    | 统计窗口（ms）                                                   |
| `open_duration_ms`         | int    | `30000`    | OPEN 状态持续时间（ms），之后进入 HALF_OPEN                             |
| `half_open_probe_count`    | int    | `3`        | HALF_OPEN 状态下允许的探测请求数                                      |
| `fallback_policy`          | string | `"reject"` | OPEN 期间新连接的处理策略：`reject`=立即快速失败（关闭客户端）；`queue`=在后端连接建立处**有界排队**——按该后端连接超时（单转发 `connect_timeout_ms` / mode `timeout_ms`，≤0 时回落 5000ms）每 50ms 轮询，熔断器转 HALF_OPEN/CLOSED 即放行（排队请求成为探测），超时仍未恢复则拒绝。仅作用于**新连接建立**，不影响在途会话/中途重连 |


---

## 十六、看门狗（`watchdog`）


| 字段                      | 类型     | 默认值          | 说明                                                    |
| ----------------------- | ------ | ------------ | ----------------------------------------------------- |
| `enabled`               | bool   | `false`      | 是否启用看门狗线程                                             |
| `heartbeat_interval_ms` | int    | `5000`       | 工作线程心跳检测周期（ms，≥500）                                   |
| `max_missed_heartbeats` | int    | `3`          | 连续丢失心跳次数后触发动作                                         |
| `action`                | string | `"log_only"` | 触发动作：`log_only` | `restart` | `core_dump_and_restart` |
| `restart_delay_ms`      | int    | `1000`       | 触发 restart 前等待（ms）                                    |


> `**restart`**：设置重启标志。宿主进程在**守护/服务模式**的保活循环中每约 5 秒经 `PA_GetStatus` 轮询该标志，命中后优雅 `stop` 并以**退出码 42** 退出，交由外部 supervisor（Windows 服务恢复 / systemd `Restart=`）重新拉起。交互式 REPL 不轮询此标志。  
> `**core_dump_and_restart**`：Windows 触发 minidump，Linux 发送 SIGABRT，随后同 `restart` 设置重启标志。

---

## 十七、全局 TLS 策略（`tls`）

适用于所有服务端（P1）和客户端（P2）TLS 上下文。


| 字段                       | 类型     | 默认值         | 说明                                             |
| ------------------------ | ------ | ----------- | ---------------------------------------------- |
| `min_tls_version`        | string | `"TLSv1.2"` | 最低 TLS 版本：`TLSv1.2` | `TLSv1.3` | `""` （OS 默认） |
| `cipher_list`            | string | `""`        | OpenSSL TLS 1.2 密码套件字符串（空=内置安全列表）              |
| `prefer_server_ciphers`  | bool   | `true`      | 是否优先使用服务端密码套件顺序                                |
| `session_ticket_enabled` | bool   | `false`     | 是否启用 TLS Session Ticket（禁用可提升前向安全性）            |


> **平台差异**：`cipher_list`、`prefer_server_ciphers`、`session_ticket_enabled` 仅在 OpenSSL（Linux）后端生效；Windows SChannel **忽略**这三项。`min_tls_version` 在两个后端均生效（SChannel 通过 `SCHANNEL_CRED` 应用）。

---

## 十八、日志扩展配置（`log_config`）


| 字段                          | 类型     | 默认值      | 说明                                                   |
| --------------------------- | ------ | -------- | ---------------------------------------------------- |
| `log_format`                | string | `"text"` | 日志输出格式：`text` | `json`                               |
| `slow_query_ms`             | int    | `0`      | 慢查询日志阈值（ms，独立记录，0=禁用）                                |
| `slow_query_file`           | string | `""`     | 慢查询日志文件路径（空=写入主日志）                                   |
| `log_include_sql`           | bool   | `false`  | 日志行中包含原始 SQL（遵循 `capture_redact_literals` 设置）        |
| `firewall_log_file`         | string | `""`     | 防火墙审计日志专用文件路径；空=不写独立文件（`FIREWALL_BLOCK` 仍写主日志）；支持热重载 |
| `firewall_log_max_size_mb`  | int    | `0`      | 防火墙日志滚动阈值（MB，0=不限大小）                                 |
| `firewall_log_rotate_count` | int    | `5`      | 滚动保留文件数（命名：`.1`、`.2`…；0=截断不保留）                       |
| `audit_export_enabled`      | bool   | `false`  | 是否写入 SIEM 友好审计导出文件（独立于 `capture.json`）                  |
| `audit_export_format`       | string | `"json"` | 导出格式：`json`（NDJSON）\| `cef` \| `syslog`；支持热重载              |
| `audit_export_file`         | string | `""`     | 审计导出文件路径（相对路径锚定 `paths.log_dir`）；`enabled=true` 时必填   |
| `audit_chain_enabled`       | bool   | `false`  | 为 NDJSON 导出行附加 `prev_hash` / `line_hash`（SHA-256 链）；支持热重载 |
| `audit_chain_key`           | string | `""`     | 哈希链盐值；空则使用机器指纹派生默认值；支持热重载 |


---

## 十九、配置管理（`config_manage`）


| 字段                        | 类型   | 默认值    | 说明                                                                         |
| ------------------------- | ---- | ------ | -------------------------------------------------------------------------- |
| `backup_on_reload`        | bool | `true` | `reload` 前自动备份 `conf.json`（命名：`conf.json.bak.YYYYMMDD_HHMMSS`；`false` 时跳过） |
| `max_backup_count`        | int  | `5`    | 同目录下最多保留的 `.bak.*` 备份数（0=不限）；`reload` 与 `setconfig` 写盘均遵守                  |
| `rollback_on_reload_fail` | bool | `true` | `reload` 解析/校验失败时自动恢复最近一次备份；`setconfig` 校验失败同样回滚                           |


---

## 十九-A、运行期目录（`paths`）

运行期文件按四个子目录分布，默认全部锚定可执行文件目录 `<exe>`。**解析优先级：命令行参数 > `DBPRISMA_HOME` 环境变量 > `conf.json` 的 `paths` 段 > `<exe>/<子目录>` 默认。**


| 字段            | 类型     | 默认值                           | 命令行覆盖           | 说明                                                                                      |
| ------------- | ------ | ----------------------------- | --------------- | --------------------------------------------------------------------------------------- |
| `run_dir`     | string | `run`（即 `<exe>/run/`）         | `--run-dir`     | `dbprisma.lock`、`dbprisma.pid`、`cli_token`、`cli_history`、`machine.key`                  |
| `log_dir`     | string | `logs`（即 `<exe>/logs/`）       | `--log-dir`     | 主日志（`logging.file`）、`log_config.firewall_log_file`、`log_config.slow_query_file` 的相对路径锚点 |
| `capture_dir` | string | `capture`（即 `<exe>/capture/`） | `--capture-dir` | `capture.json` 及其轮转副本                                                                   |


> `config_dir`（`conf.json`/`proxy_users.json`/备份所在目录）无法在 `conf.json` 内自指，只能通过 `-c/--config <文件>`、`--config-dir <目录>` 或 `DBPRISMA_HOME` 指定，默认 `<exe>/config/`。

说明：

- `paths.*` 中**相对路径**锚定 `<exe>`；**绝对路径**按原样使用。
- 设置环境变量 `DBPRISMA_HOME=/srv/dbprisma` 会把四个子目录整体定位到该根下，且**优先级高于 `conf.json`**。
- `logging.file`、`firewall_log_file`、`slow_query_file` 若为相对路径，现统一锚定到 `log_dir`（绝对路径不受影响）。
- `machine.key`（`run/dbprisma.machine.key`）：**历史遗留**，机器身份已不再依赖此文件；新装可忽略。仍列于 `run_dir` 解析路径中以便兼容旧部署。
- 目录在启动时按需自动创建（best-effort）。
- `setconfig paths` 可交互编辑 `run_dir`/`log_dir`/`capture_dir`。

```json
"paths": {
    "run_dir":     "run",
    "log_dir":     "logs",
    "capture_dir": "capture"
}
```

---

## 二十、弹性扩展字段（`resilience` 新增）

在原有 `resilience` 段中增加以下字段：


| 字段                             | 类型     | 默认值     | 说明                                                                                          |
| ------------------------------ | ------ | ------- | ------------------------------------------------------------------------------------------- |
| `keepalive_idle_sec`           | int    | `60`    | 空闲多久后开始发探测（`backend_tcp_keepalive=true` 时生效）。Linux→`TCP_KEEPIDLE`；Windows→`SIO_KEEPALIVE_VALS.keepalivetime`。在新建后端连接时下发 |
| `keepalive_interval_sec`       | int    | `10`    | 探测间隔。Linux→`TCP_KEEPINTVL`；Windows→`SIO_KEEPALIVE_VALS.keepaliveinterval`                                            |
| `keepalive_count`              | int    | `5`     | 探测失败几次判定断开。Linux→`TCP_KEEPCNT`；**Windows 不可调（由 OS 固定）**                                                            |
| `retry_on_lost_connection`     | bool   | `false` | ⚠️ **保留/未实现**（计划中）：检测到 "Lost connection to MySQL server" 时自动重试 |
| `retry_idempotent_only`        | bool   | `true`  | ⚠️ **保留/未实现**（计划中）：仅对幂等 SQL（SELECT/SHOW）重试                  |
| `max_retry_count`              | int    | `1`     | ⚠️ **保留/未实现**（计划中）：最大重试次数                                  |
| `retry_delay_ms`               | int    | `50`    | ⚠️ **保留/未实现**（计划中）：重试前等待（ms）                               |
| `auto_failover`                | bool   | `false` | 主后端被韧性标记为不健康时，**新连接**自动重定向至 `failover_target_id`（若健康）或下一健康后端；写 WARN 日志。不影响已建立会话。`failover_restore_on_recovery` / `failover_restore_delay_ms` 仍为保留（不自动恢复 primary 路由） |
| `failover_target_id`           | int    | `0`     | 故障转移目标 `server_id`（`0`=选第一个健康的其他 enabled 后端） |
| `failover_restore_on_recovery` | bool   | `true`  | ⚠️ **保留/未实现**（计划中）：主后端恢复后自动切回                              |
| `failover_restore_delay_ms`    | int    | `30000` | ⚠️ **保留/未实现**（计划中）：切回前等待（ms）                               |
| `session_leak_detect`          | bool   | `false` | 启用会话泄漏检测（TD-006 log-only 已实现：后台线程每 5s 扫描 `SESSION_CLOSING` 超过阈值的会话并写 WARN 日志）               |
| `session_leak_threshold_sec`   | int    | `300`   | `SESSION_CLOSING` 超时阈值（秒，≥10；低于 10 时启动 WARN 提示可能误报）                                         |
| `session_leak_action`          | string | `"log"` | 泄漏动作：`log`（**已实现**）\| `force_close`（⚠️ **有意不实现**，填写不报错但无 force_close 行为）。安全决策（TD-006）：检测器只盯**已进入 `SESSION_CLOSING` 且 socket 已全部关闭**的卡死会话，其卡死成因是 `io_inflight>0`（IOCP 完成包未回）；要"强制"释放只能绕过 `io_inflight==0` 门控直接 free，而这正是引擎用门控+CAS 防止的 **use-after-free**。故强制关闭风险>收益，保持 log-only 由运维介入排查 |


---

## 二十一、读写分离扩展字段（`mode` 新增）

> **实现状态说明**：下列字段在当前版本中已完成配置解析与校验，但部分功能引擎侧尚未完全接线，详见各字段备注。


| 字段                         | 类型     | 默认值             | 说明                                                                                                                                                 |
| -------------------------- | ------ | --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `rw_ddl_to_primary`        | bool   | `true`          | DDL（CREATE/ALTER/DROP…）强制路由到写库。**当前行为已等价于 `true`**：`rw_should_route_read()` 对非 DQL 一律返回 FALSE → 主库；将字段改为 `false` 无运行时效果（引擎未实现 DDL→读库路径，属已知 TD-008） |
| `rw_dcl_to_primary`        | bool   | `true`          | DCL（GRANT/REVOKE…）强制路由到写库。同上，当前行为已等价于 `true`；改为 `false` 无效（TD-008）                                                                                 |
| `rw_transaction_detection` | string | `"pessimistic"` | 事务检测模式。引擎内置 pessimistic 检测（`rw_in_transaction`/`rw_autocommit_off`），与默认值行为一致；`"optimistic"` 路径未实现，修改此字段无运行时效果（TD-008）                              |
| `mirror_enabled`           | bool   | `false`         | 启用流量镜像（⚠️ **功能尚未实现**，当前版本配置解析但不执行任何镜像操作）                                                                                                           |
| `mirror_server_id`         | int    | `0`             | 镜像目标 server_id（⚠️ 同上）                                                                                                                              |
| `mirror_ratio_pct`         | int    | `100`           | 镜像比例（1–100，百分比）（⚠️ 同上）                                                                                                                             |
| `mirror_async`             | bool   | `true`          | 镜像异步发送（⚠️ 同上）                                                                                                                                      |
| `mirror_ddl`               | bool   | `false`         | 是否镜像 DDL 语句（⚠️ 同上）                                                                                                                                 |


---

## 二十二、`notify` 段（通知/告警）

控制代理在关键事件发生时是否写入操作系统事件日志（Windows Event Log / syslog）。

```json
"notify": {
    "win_event_log":    true,
    "syslog":           false,
    "debounce_sec":     60,
    "on_backend_error": true,
    "on_circuit_breaker": true,
    "on_firewall_block": false,
    "on_auth_fail":     true,
    "on_config_reload": false
}
```


| 字段                   | 类型   | 默认值     | 说明                                   |
| -------------------- | ---- | ------- | ------------------------------------ |
| `win_event_log`      | bool | `true`  | 是否写入 Windows Event Log（仅 Windows 有效） |
| `syslog`             | bool | `false` | 是否写入 syslog（仅 Linux 有效）              |
| `debounce_sec`       | int  | `60`    | 同一事件连续通知最小间隔（秒），防日志风暴                |
| `on_backend_error`   | bool | `true`  | 后端连接失败/DOWN 时通知                      |
| `on_circuit_breaker` | bool | `true`  | 熔断器触发（OPEN/HALF_OPEN）时通知             |
| `on_firewall_block`  | bool | `false` | SQL 防火墙拦截时通知                         |
| `on_auth_fail`       | bool | `true`  | 认证失败超出阈值时通知                          |
| `on_config_reload`   | bool | `false` | 配置热重载成功/失败时通知                        |


> 整个 `notify` 段可省略，使用上表所示默认值。**不支持 `reload`**，修改后须执行 `stop` → `start` 重启才能生效。

---

## 二十三、`command_pipe` 段（带外管理控制通道）

让运维客户端通过本地 IPC **远程登录并执行管理子命令**，而不必进入 `XR:>` 提示符。
适用于守护/服务模式：Linux `systemd` 或 `--no-interactive`；Windows SCM 或 `--no-interactive`
（交互模式下不启用，绝不影响 `XR:>`）。控制通道与代理启动**解耦**——即使核心代理启动失败，
管道仍可用于远程诊断/恢复。

- **Windows**：命名管道 `\\.\pipe\<pipe_name>`
- **Linux**：AF_UNIX 套接字 `<run_dir>/<pipe_name>.sock`

```json
"command_pipe": {
    "enabled":                  false,
    "pipe_name":                "DBPrisma_ctrl",
    "local_only":               true,
    "max_instances":            4,
    "session_idle_timeout_sec": 300,
    "auth_store":               "admin_users.json",
    "capture_queue_mb":         256,
    "capture_buf_kb":           64,
    "allowed_commands":         ["status", "version", "reload", "start", "stop",
                                 "restart", "conns", "backends", "pause", "resume",
                                 "drain", "flush", "loglevel", "kick", "display", "capture"]
}
```


| 字段                         | 类型    | 默认值                | 说明                                                                                                            |
| -------------------------- | ----- | ------------------ | ------------------------------------------------------------------------------------------------------------- |
| `enabled`                  | bool  | `false`            | 是否启动控制通道（仅守护/服务模式生效）                                                                                          |
| `pipe_name`                | 字符串   | `DBPrisma_ctrl`    | 管道/套接字基名（见上方平台路径）                                                                                             |
| `local_only`               | bool  | `true`             | 收紧 ACL/DACL 仅限本机客户端，**强烈建议保持 true**                                                                           |
| `max_instances`            | int   | `4`                | 并发客户端上限（运行期钳制到 1–64）                                                                                          |
| `session_idle_timeout_sec` | int   | `300`              | 会话空闲超时（秒，0=禁用）                                                                                                |
| `auth_store`               | 字符串   | `admin_users.json` | 管理员凭据文件（相对 config 目录）                                                                                         |
| `capture_queue_mb`         | int   | `256`              | `capture` 流式抓取的内存 FIFO 上限（MB，运行期钳制 1–4096）；超限丢弃最旧记录并以 `{"_dropped":N}` 通知客户端。Windows/Linux 通用                 |
| `capture_buf_kb`           | int   | `64`               | `capture` 数据通道传输缓冲（KB，运行期钳制 4–1024）。Windows=命名管道收发缓冲；Linux=AF_UNIX 套接字 `SO_SNDBUF/SO_RCVBUF`。Windows/Linux 通用 |
| `allowed_commands`         | 字符串数组 | 见下                 | 允许经管道执行的子命令白名单；为空时用内置默认集                                                                                      |


> **不支持 `reload`**：修改后须 `restart`（或重启 OS 服务）。`checkconfig` 会校验：`local_only=false`
> 给出安全 WARN、`auth_store` 不存在或无用户给出 WARN、正常时打印 `[OK] ... (N admin users)`。

### 23.1 鉴权文件 `admin_users.json`

与 `proxy_users.json` **完全独立**——管道登录用户**不是**代理用户、也不是后端 MySQL 用户。
路径默认 `config/admin_users.json`（由 `auth_store` 指定）。

```json
{
  "users": [
    { "username": "admin", "password": "ENC2:加密密码" }
  ]
}
```

**推荐用命令管理**（复用 `useradd`/`userdel`/`userlist`/`setpwd`，加 `--admin` 旗标即操作本文件，
密码自动以 `ENC2:` 加密落盘；不加旗标时仍操作 `proxy_users.json`）：

```text
XR:> useradd  --admin admin      # 新建管理员（交互输入密码，无回显）
XR:> setpwd   --admin admin      # 改密码
XR:> userlist --admin            # 列出管理员
XR:> userdel  --admin admin      # 删除
```

- `--admin` 用**旗标**而非位置关键字，因此名为 `admin` 的代理用户不会产生歧义；短写 `-a` 等价。
- 也可手工编辑本文件：密码支持明文或 `ENC2:`（机器绑定 AES-256-GCM）；明文可用于快速联调，
生产建议用上面命令或 `DBPrisma passwd` 生成的 `ENC2:` 串。
- 写盘为原子替换（`.tmp`→重命名，旧档留 `.bak`）。同进程内运行的管道服务会**即时**看到变更；
分离的守护进程在下次启动时加载。
- `enabled=true` 但该文件缺失/无用户时，**所有管道登录都会被拒绝**（控制通道形同禁用）。

### 23.2 协议与白名单

> **命令与参数完整列表**：[命令管道命令参考手册](command-pipe-command-reference.md)（默认白名单、各动词语法、`capture` 管道专语义、§1A 管道原生 op、硬性禁止命令）。

- 帧格式：4 字节小端长度前缀 + UTF-8 JSON。
- 流程：`{"op":"login","user","pass"}` → 返回 `{"ok":true,"token":...}`；
`{"op":"cmd","token","line":"status"}` → 返回 `{"ok":true,"output":...}`；`{"op":"logout","token"}`。
- **管道原生 op**（非 `cmd`/`line`）：`ping`（免登录）、`whoami`、`metrics`、`capstatus`、`capend`（后四者须 token）。详见 [命令管道命令参考手册 §1A](command-pipe-command-reference.md#1a-管道原生-op仅管道无-xr-对应)。
- `allowed_commands` 之外的子命令一律拒绝；此外一组**交互/REPL 专属**命令
（如 `clear`/`exit`/`help`/`history` 及交互式 `capture` 的按键模式）被硬性禁止，无法经管道执行。

### 23.3 流式 SQL 抓取 `capture`

经管道执行 `capture` 会**另开一条专用数据通道**用于流式推送，主通道立即返回
`{"ok":true,"channel":"<管道/套接字名>","token":"<一次性令牌>"}`（失败返回 `{"ok":false,"error":...}`）。

- 客户端连接 `channel` 后，**首行发送 `token\n` 做握手鉴权**（防同机其它进程窃取明文 SQL 流），
服务端回 `{"ok":true}` 后开始按行推送 **NDJSON**（字段与 `capture.json` 逐字节一致）。
- 过滤参数沿用交互式语义：`-s ip[:port]`（后端）、`-c ip[:port]`（客户端）、
`-t DML,DQL,DDL,DCL,TCL|ALL`（类型）。**输出恒为 screen 流式**，无 `mode` 参数。
- 停止抓取：发送 **§1A 管道原生 op** `{"op":"capend","token":"..."}`（不经白名单、不进 `XR:>`）。
- 背压：有界队列（上限由 `capture_queue_mb` 配置，默认 256 MB，尽量不丢；溢出才丢最旧记录，并以 `{"_dropped":N}` 行告知客户端）。
回调只在代理 IO 线程做格式化+入队，**不阻塞**代理热路径。
- 传输缓冲由 `capture_buf_kb` 配置（默认 64 KB，Windows/Linux 通用）：Windows 设命名管道收发缓冲，Linux 设 AF_UNIX 套接字 `SO_SNDBUF/SO_RCVBUF`。
- `capture` 受 `allowed_commands` 白名单门控（须含 `capture`）；`stop`/进程退出会自动结束抓取。

### 23.4 测试客户端

仓库自带跨平台测试客户端，演示登录/命令/登出与 `capture` 流式：

```text
# Windows（命名管道）
tools\cmdpipe_client.ps1 -User admin -Pass secret -Command "status"
tools\cmdpipe_client.ps1 -User admin -Pass secret -Capture "capture -t DML,DQL" -Seconds 30

# 跨平台（Linux AF_UNIX / Windows 命名管道）
tools/cmdpipe_client.py --sock <run_dir>/DBPrisma_ctrl.sock --user admin --pass secret --command status
tools/cmdpipe_client.py --sock <run_dir>/DBPrisma_ctrl.sock --user admin --pass secret \
    --capture "capture -t DML" --seconds 30
```

> **二次开发**：用 C#/Java/Python 自行对接两条管道（控制管道帧协议 + capture 数据通道握手/NDJSON），
> 见 [命令管道客户端开发手册](command-pipe-client-guide.md) 及示例目录
> `[examples/command-pipe/](../../examples/command-pipe/)`。

---

## 二十三.A、`license` 段（授权门控提示，可选）

顶层可选段，仅影响**到期前提示**的提前量；不影响授权/试用的放行与拒绝判定（那套逻辑见 [运维手册 §13](operations-manual.md)）。缺省时使用内置默认值，无需显式配置。

```jsonc
"license": {
  "warn_days": 14
}
```

| 键 | 类型 | 默认 | 说明 |
|----|------|------|------|
| `warn_days` | 整数 | `14` | 到期前提示的提前天数；license **或**试用剩余天数 ≤ 此值时，启动与每小时复检打印 `[license] ... expires in N day(s)` 提醒，`status verbose` 的 `Days left` 转黄。`0` = 关闭提醒。运行时取值范围 `0–365`（越界自动夹取）。对正式 license 与试用**对称生效**。 |

> 配合监控：`/metrics`（及命令管道 `metrics` op）输出 `dbprisma_license_state`（2=授权/1=试用/0=拒绝/-1=未知）与 `dbprisma_license_days_left`（剩余整天数）两项指标，可在到期前自动告警。
>
> `warn_days` 仅控制提示，不改变到期行为：正式 license 过期后是否仍有缓冲由签发侧 `licissue --grace-days` 决定（默认 0 = 过期即拒）；试用无宽限。

---

## 二十四、Lab 配置模板


| 路径                             | 用途                               |
| ------------------------------ | -------------------------------- |
| `config/windows/conf_lab.json` | Windows 四后端 + mode dispatch 模板   |
| `config/linux/conf_lab.json`   | Linux 37 lab（远程后端 192.168.10.34） |
| `config/conf.json`             | 仓库根目录通用模板                        |


部署时复制到运行目录并重命名为 `config/conf.json`。

*文档版本：2026-07-13（r19）。修订要点（r18→r19）：安全中期——`sql_access_policies[].action` 新增 `audit_only`（写入审计导出 `policy_audit` 事件 + 防火墙日志，不拦截）；`priority` 字段（多条 scope 命中时取最高）。韧性中期——`resilience.auto_failover` 落地（新连接在首选后端不健康时重定向至 `failover_target_id` 或下一健康节点；mode 统一端口与各 `listen_port` 均适用）。此前（r17→r18）：审计阶段 3——`GET /audit/search` 增补 `event`/`user`/`tool`/`offset` 过滤与 `returned`/`has_more` 分页响应（§一 Admin 端点表、§8.1）；新增 CLI `checkaudit`/`ca` 校验 NDJSON 哈希链（§8.1）；`admin_url_decode_param` 在查询串末参数处遇空白截断，避免将 `HTTP/1.1` 误读入末参。此前（r16→r17）：`max_reassemble_bytes` 取值范围由 `65536 ~ 16777215` 更正为 `65536 ~ 1073741824`（1 GiB，对齐 MySQL `max_allowed_packet` 硬上限与协议层 `PROTO_REASSEMBLE_MAX`）——修复 `config_loader.c` 遗留的 16 MiB−1 静默钳制后，大于 16 MiB 的配置现可实际生效（一条逻辑语句可跨多个 16 MiB 续传包）；超限/过小时钳制并打印 `[WARN]`，不再静默截断；表行补充「后端 `max_allowed_packet` 须不小于该值」提示。此前（r15→r16）：随 TD-007 偿还 + TD-012 续偿，更新「实现缺口」总表与字段明细的状态标注——`circuit_breaker.fallback_policy`（`queue` 有界排队已生效）、`security.max_packet_size_bytes`、`log_config.log_include_sql`、`notify.syslog`、`resilience.keepalive_idle_sec/interval_sec/count`（调优已生效，Linux `TCP_KEEP*` / Windows `SIO_KEEPALIVE_VALS`）均由「保留/未实现」改为「已生效」；`resilience.retry_*`/`auto_failover`/`failover_*` 明细行补「⚠️ 保留/未实现（计划中）」标记与 gap 表一致；`session_leak_action=force_close` 标注「有意不实现」并附 TD-006 安全理由（绕过 `io_inflight` 门控会引入 UAF）；`mode.mirror_*` 归属由 TD-008 更正为 TD-005（计划中）。此前（r14→r15）：新增 §二十三.A「`license` 段（授权门控提示，可选）」，含 `warn_days`（到期前提醒提前天数，默认 14，对正式 license 与试用对称生效，范围 0–365）与配套 `/metrics` 指标 `dbprisma_license_state`/`dbprisma_license_days_left` 说明；顶层结构 JSON 同步加入 `license` 段；全量同步对齐 `config/` 全部模板与 `bin/**/config` 各副本（conf.json、conf_tran/auth/lab、conf_security.example、conf_arm64_min 等均显式带上 `license` 块）。此前（r13→r14）：`command_pipe` 新增 `capture_queue_mb`（抓取 FIFO 上限，默认 256 MB）与 `capture_buf_kb`（数据通道传输缓冲，默认 64 KB）两项配置，Windows/Linux 通用，已同步 `setconfig`/`checkconfig`/示例 conf.json（§二十三、§二十三.3）。此前（r12→r13）：新增 §二十三「`command_pipe` 段（带外管理控制通道）」，涵盖配置键、`admin_users.json` 鉴权文件、帧协议与白名单、`capture`/`capend` 流式 SQL 抓取（数据通道 + token 握手 + `-s`/`-c`/`-t`，恒 screen，`capend` 不进 `XR:>`）、测试客户端；顶层结构 JSON 与 §七 `setconfig`、§九 `reload` 表同步加入 `command_pipe`（须 restart）；文件布局补 `admin_users.json`；Lab 模板段顺延为 §二十四。`admin_users.json` 现可用 `useradd`/`userdel`/`userlist`/`setpwd` 加 `--admin` 旗标命令化管理（§二十三-1、§十二.2）。*新增 §二十三.A「`license` 段（授权门控提示，可选）」，含 `warn_days`（到期前提醒提前天数，默认 14，对正式 license 与试用对称生效，范围 0–365）与配套 `/metrics` 指标 `dbprisma_license_state`/`dbprisma_license_days_left` 说明；顶层结构 JSON 同步加入 `license` 段；全量同步对齐 `config/` 全部模板与 `bin/**/config` 各副本（conf.json、conf_tran/auth/lab、conf_security.example、conf_arm64_min 等均显式带上 `license` 块）。此前（r13→r14）：`command_pipe` 新增 `capture_queue_mb`（抓取 FIFO 上限，默认 256 MB）与 `capture_buf_kb`（数据通道传输缓冲，默认 64 KB）两项配置，Windows/Linux 通用，已同步 `setconfig`/`checkconfig`/示例 conf.json（§二十三、§二十三.3）。此前（r12→r13）：新增 §二十三「`command_pipe` 段（带外管理控制通道）」，涵盖配置键、`admin_users.json` 鉴权文件、帧协议与白名单、`capture`/`capend` 流式 SQL 抓取（数据通道 + token 握手 + `-s`/`-c`/`-t`，恒 screen，`capend` 不进 `XR:>`）、测试客户端；顶层结构 JSON 与 §七 `setconfig`、§九 `reload` 表同步加入 `command_pipe`（须 restart）；文件布局补 `admin_users.json`；Lab 模板段顺延为 §二十四。`admin_users.json` 现可用 `useradd`/`userdel`/`userlist`/`setpwd` 加 `--admin` 旗标命令化管理（§二十三-1、§十二.2）。*
*（r11→r12）：补齐 `paths` 段（统一运行期目录解析器，2026-06-12/13 落地）的总览计数与结构对齐——① 顶层结构 JSON 补 `paths` 段；② 键数由 **159 更正为 162**（`config_loader.c` 1178–1185 行解析 `paths.run_dir`/`log_dir`/`capture_dir` 三个标量叶子键，标量由 154→157，5 个数组/集合键不变；即在下方 r5→r6 历史分段累加的 159 之上再加 `paths×3`=162）。`paths` 的语义/优先级/字段文档此前已在 §十九-A 给出，本次仅补总览处的计数与顶层结构（历史修订注记按当时口径保留，不追改）。
（r10→r11）：在「内存提示」补充 glibc 多 arena 内存保留的运维说明与 `MALLOC_ARENA_MAX` 可选调优，并记录 ASan/LSan 在双 TLS 路径下验证**无按连接内存泄漏**的结论。
（r9→r10）：更正 `max_reassemble_bytes` 内存模型——
重组/TLS 缓冲（c2s/s2c/tls_plain）改为**按需几何增长*（初值约 16 KiB/64 KiB，上限仍为本配置值的天花板），
不再每会话急切预分配满额；§1.0 表行与「内存提示」据实改写，最坏上界仍约 `2 × max_reassemble_bytes`/会话，
并标注本值为**非 reload 字段**（改后须重启）。源码：`mysql_proto.{c,h}` `proto_ctx_init`/`proto_buf_ensure`/`buf_cap`。
（r8→r9）：CLI/REPL 与源码再对齐——
①子命令顺序规则按 `main.c` 实际行为更正：仅 `checkconfig`/`cc`、`setconfig`/`sc`、`service` 须作首参（选项之前），`start` 选项可前可后；
②`drain` 更正为按后端 `drain <server_id>` / `drain cancel <server_id>`（无"全部排空"形式）；
③`flush` 更正为清空**预热连接池**并后台重建，参数 `[server_id]` 可选（省略=全部）；
④`history`/`hist` 更正为 128 条环形缓冲且**持久化**到 `<run>/dbprisma.cli_history`（非"仅内存"）；
⑤§密码加密：`passwd` 标注为 XR:> 运行时命令；⑥新增 §12.3 环境变量表（`DBPRISMA_CLI_TOKEN`/`DBPRISMA_HOME`/`NO_COLOR`）。
（r7→r8）：核对 `config_loader.c` 真实键数——
首部计数由 158 更正为 **159**（原 158 把 4 个 `json_get_`* 取值器函数定义误计入调用点；
真实为 154 个标量叶子键 + 5 个数组/集合键）；§全字段计数中 `proxy` 由 ×13 更正为 ×17
（补回 `admin_listen_ip/port`、`admin_bearer_token`、`admin_request_timeout_ms`、
`admin_max_header_bytes`、`admin_tls_pfx_file/password`、`admin_tls_cert_file/key_file`、
`admin_allow_ips` 等 admin_* 键），总计由 155 更正为 159。其余 15 段计数与代码一致。
（r6→r7）：§12 CLI 与源码对齐——`status` 仅 `verbose`（移除 `-v`/`--verbose`）；`conns [ip]` 过滤；新增 `kick id <n>`、`history`/`hist`、`help [command]`；§12.1 补 `--pid-file`/`--print-config`/`--warmup-only` 并说明 `--opt=value`/`--`/未知参数报错。r5→r6：① 新增 §二十二 notify 段（8个字段，之前未文档化）；② resilience.enabled 默认值更正为 `true`；③ 修订全字段计数：version×1 + proxy×17（含 admin_* 10 项，其中 admin_allow_ips 为数组）+ proxy.startup×5 + proxy.shutdown×11 + servers[]×24（含 backend_ssl_mode=preferred 与 servers 数组本身）+ logging×4 + mode×17 + resilience×26 + security×18（含 ip_allowlist/ip_denylist/sql_block_patterns 3 个数组）+ capture×2 + circuit_breaker×7 + watchdog×5 + tls×4 + log_config×7 + config_manage×3 + notify×8 = 159。*