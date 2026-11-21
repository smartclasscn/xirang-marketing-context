# 命令管道命令参考手册

**版本**: r3 | **日期**: 2026-06-18 | **适用版本**: DBPrisma ≥ 0.0.1.568

本文档是 **`command_pipe` 控制管道可执行命令的完整参考**：列出默认白名单内的全部动词、每条命令的**语法与参数**、管道与 `XR:>` 的差异，以及**永远无法经管道执行**的命令。

> **线缆协议**（帧格式、登录、`capture` 数据通道握手）见 [命令管道客户端开发手册](command-pipe-client-guide.md)。  
> **配置项**（`enabled`、`allowed_commands`、`capture_queue_mb` 等）见 [配置参考手册 §二十三](configuration-reference.md)。  
> **启用步骤**见 [管理员手册 §4.4](admin-manual.md#44-命令管道远程管控command_pipe)。

---

## 1. 如何发送命令

经控制管道登录后，每一行管理命令通过 JSON 帧发送：

```json
{"op":"cmd","token":"<会话 token>","line":"<整行命令>"}
```

- `line` 的语法与 `XR:>` 提示符下键入的**同一行**一致（首个 token 为动词）。
- 管道会话使用 `admin_users.json` 鉴权，**不需要** CLI 的 `--token` / `DBPRISMA_CLI_TOKEN`。
- 成功时响应 `{"ok":true,"output":"<stdout 捕获文本>"}`；失败时 `{"ok":false,"error":"..."}`。

---

## 1A. 管道原生 op（仅管道，无 XR:> 对应）

下列 **五个**操作通过**独立 `op` 帧**调用（**不是** `{"op":"cmd","line":"..."}`），且**不会**出现在 `XR:>` 提示符或 `allowed_commands` 白名单中。

| op | 需要 login | 请求示例 | 成功响应要点 |
|----|:----------:|----------|--------------|
| **`ping`** | 否 | `{"op":"ping"}` | `pipe`、`command_pipe`、`service_running`、`proxy_running`、`version` |
| **`whoami`** | 是 | `{"op":"whoami","token":"..."}` | `user`：当前管道登录名 |
| **`metrics`** | 是 | `{"op":"metrics","token":"..."}` | `format":"prometheus"` + `metrics` 文本（同 Admin HTTP `/metrics`） |
| **`capstatus`** | 是 | `{"op":"capstatus","token":"..."}` | 流式 `capture` 会话状态（见下） |
| **`capend`** | 是 | `{"op":"capend","token":"..."}` | 停止当前流式 `capture` 并销毁数据管道（幂等） |

### `ping`（连通性探针）

```json
{"op":"ping"}
```

```json
{"ok":true,"pipe":"DBPrisma_ctrl","command_pipe":true,
 "service_running":true,"proxy_running":true,"version":"0.0.1.568"}
```

- **`service_running`**：控制管道服务是否在监听。
- **`proxy_running`**：MySQL 代理核心是否已 `start` 且引擎报告运行中。

### `whoami`（当前管道用户）

```json
{"op":"whoami","token":"<会话 token>"}
```

```json
{"ok":true,"user":"admin"}
```

### `metrics`（Prometheus 指标快照）

```json
{"op":"metrics","token":"<会话 token>"}
```

```json
{"ok":true,"format":"prometheus","metrics":"# HELP dbprisma_up ...\ndbprisma_up 1\n..."}
```

与 `GET /metrics` 同源（`PA_GetStatus` + `admin_metrics_format`），适合不经 HTTP 的监控采集。

### `capstatus`（流式抓取状态）

```json
{"op":"capstatus","token":"<会话 token>"}
```

无活跃抓取：

```json
{"ok":true,"active":false}
```

有活跃抓取：

```json
{"ok":true,"active":true,"channel":"\\\\.\\pipe\\DBPrisma_cap_...",
 "dropped":0,"queue_bytes":4096,
 "server_ip":"","server_port":0,"client_ip":"","client_port":0,
 "sql_type_mask":31,"redact_literals":false}
```

| 字段 | 说明 |
|------|------|
| `channel` | 数据管道连接地址（与 `capture` 响应一致） |
| `dropped` | FIFO 溢出丢弃的记录数 |
| `queue_bytes` | 当前队列占用字节 |
| `server_ip` / `client_ip` / `-t` 相关 | 与启动 `capture` 时 filter 一致 |

### `capend`（停止流式抓取）

```json
{"op":"capend","token":"<会话 token>"}
```

```json
{"ok":true}
```

- 停止当前流式 `capture` 会话并销毁数据管道；**幂等**（无活跃抓取时也返回 `{"ok":true}`）。
- **不会**出现在 `XR:>` 提示符中；也**不需要**写入 `allowed_commands`。
- 此外，客户端断开数据管道、执行 `stop` 或进程退出也会自动结束抓取。

> **兼容说明**：早期客户端曾通过 `{"op":"cmd","line":"capend"}` 发送；服务端仍接受该形式，但新集成应使用上表 `op:"capend"`。

---

## 2. 白名单与访问控制

### 2.1 默认白名单（`allowed_commands` 未配置时）

与源码 `CP_DEFAULT_ALLOWED` 一致，共 **16** 个动词：

| # | 动词 | 说明 |
|---|------|------|
| 1 | `start` | 启动代理 |
| 2 | `stop` | 优雅停止 |
| 3 | `restart` | 优雅重启 |
| 4 | `status` | 运行状态 |
| 5 | `reload` | 热更配置 |
| 6 | `conns` | 活跃连接列表 |
| 7 | `backends` | 后端健康摘要 |
| 8 | `pause` | 暂停接受新连接 |
| 9 | `resume` | 恢复接受新连接 |
| 10 | `drain` | 后端排水 |
| 11 | `flush` | 刷新预热池 |
| 12 | `loglevel` | 运行时改日志级别 |
| 13 | `version` | 版本信息 |
| 14 | `kick` | 踢出连接 |
| 15 | `display` | 读盘显示配置 |
| 16 | `capture` | 流式 SQL 抓取（管道专语义，见 §4） |

可在 `conf.json` 的 `command_pipe.allowed_commands` 中**增减**白名单条目；配置为空数组时回退到上表默认集。

> **不在白名单内的管道原生 op**：`ping`、`whoami`、`metrics`、`capstatus`、`capend` 见 §1A，不经 `allowed_commands` 校验。

### 2.2 硬性禁止（白名单无法放开）

下列命令在源码 `dbp_pipe_command_refused()` 中**永久拒绝**，即使写入 `allowed_commands` 也会返回 `command not permitted`：

| 类别 | 动词 | 原因 |
|------|------|------|
| REPL 专属 | `clear` / `cls`、`exit` / `quit`、`help` / `?`、`history` / `hist` | 交互式终端专用 |
| 交互式用户管理 | `useradd`、`userdel`、`setpwd`、`passwd` | 需交互式密码输入 |
| 配置编辑 | `setconfig` / `sc` | 磁盘写配置 + 多步交互 |
| 进程/服务 | `shutdown`、`service` | 会终止宿主或改动 OS 服务注册 |
| CLI 令牌 | `token` | 打印/轮换 CLI token 路径 |
| 交互式抓取 | `capture` / `cap`（经 `dbp_exec_line` 路径） | 阻塞式 REPL 抓取；管道须用 §4 的流式 `capture` |

> **可扩展但非默认**：`userlist`、`checkconfig` 不在默认白名单，也未被硬性禁止。若手动加入 `allowed_commands` 且经 `dbp_exec_line` 执行，**可以**经管道使用（见 §5）。

---

## 3. 命令详细参考

下列命令均假设已通过管道 `login` 且动词在有效白名单内（§1A 管道原生 op 除外）。

### 3.1 服务生命周期

#### `start`

```
start [config_file]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `config_file` | 否 | 配置文件路径；省略时使用默认 `conf.json`（相对 config 目录或 `-c` 指定路径） |

**示例**

```text
start
start /opt/dbprisma/config/conf.json
```

**说明**：代理已在运行时再次 `start` 会报错；输出为启动日志摘要。

---

#### `stop`

```
stop
```

无参数。优雅停止代理（等待连接排水），**不会**退出守护进程本身。

---

#### `restart`

```
restart [config_file]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `config_file` | 否 | 重启时加载的配置；省略则用当前配置 |

等价于 `stop` → 预检/预热 → `start`。

---

#### `reload`

```
reload [config_file]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `config_file` | 否 | 要热加载的配置文件；省略为当前 `conf.json` |

仅对**可热更字段**生效（logging、部分 resilience、admin token、慢查询阈值等）；`security.*`、`servers[]` 结构性变更须 `restart`。详见 [配置参考 §九](configuration-reference.md)。

---

### 3.2 状态与监控

#### `status`

```
status [verbose]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `verbose` | 否 | 唯一合法子参数；增加 QPS、计数器、逐后端探针详情、**License 段**（Mode / Days left / Detail） |

**示例**

```text
status
status verbose
```

---

#### `backends`

```
backends
```

无参数。列出各后端在线/离线、排水、探针失败/成功、熔断与 backoff 摘要。

---

#### `version`

```
version
```

无参数。输出版本号、平台架构、构建日期。

---

#### `display`

```
display
```

无参数。从磁盘读取并显示当前 `conf.json` 与 `proxy_users.json` 摘要（只读，不反映仅内存中的运行时状态）。

---

#### `conns`

```
conns [ip]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `ip` | 否 | 客户端 IP 过滤；省略则列出全部活跃连接 |

输出含连接 ID、客户端 IP:端口、用户、库、状态、持续时间等。`kick id <n>` 的行号取自**最近一次** `conns` 输出。

---

### 3.3 连接与后端控制

#### `pause`

```
pause
```

无参数。暂停接受**所有**新客户端连接；已有连接不受影响。

---

#### `resume`

```
resume
```

无参数。解除 `pause`。

---

#### `drain`

```
drain <server_id>
drain cancel <server_id>
```

| 形式 | 说明 |
|------|------|
| `drain <server_id>` | 标记 `servers[].id == server_id` 的后端为排水：停止向其路由**新**连接，存量连接自然结束 |
| `drain cancel <server_id>` | 取消该后端排水，恢复路由 |

> 必须指定 `server_id`；**无**「排空全部后端」形式。

**示例**

```text
drain 1
drain cancel 1
```

---

#### `flush`

```
flush [server_id]
```

| 参数 | 必填 | 说明 |
|------|------|------|
| `server_id` | 否 | 后端 ID；**省略则刷新全部**后端的预热连接池 |

清空并后台重建 warmup pool；**不断开**当前活跃会话。

---

#### `kick`

```
kick <client_ip> [port]
kick id <n>
```

| 形式 | 说明 |
|------|------|
| `kick <client_ip> [port]` | 断开指定客户端 IP 的连接；`port` 省略或为 `0` 表示该 IP 所有端口 |
| `kick id <n>` | 断开 `conns` 列表第 **N** 行（`#` 列）对应的单条连接 |

**示例**

```text
kick 192.168.1.100
kick 192.168.1.100 54321
kick id 3
```

---

### 3.4 日志

#### `loglevel`

```
loglevel <level>
```

| 参数 | 必填 | 合法值 |
|------|------|--------|
| `level` | **是** | `debug` \| `info` \| `warn` \| `error` |

立即生效，无需重启；仅影响当前运行实例。

---

## 4. 流式 SQL 抓取：`capture`

管道上的 `capture` **不是** `XR:>` 里阻塞式屏幕抓取，而是：

1. 在控制管道发 `{"op":"cmd","line":"capture ..."}` → 立即返回 `channel` + 一次性 `token`；
2. 客户端连接数据管道，首行发送 `token\n` 握手；
3. 服务端按行推送 **NDJSON**（字段与 `capture.json` 一致）；
4. 发 **`{"op":"capend"}`**（§1A）或断开数据管道、或 `stop`/进程退出 → 结束抓取。

### 4.1 `capture`（控制管道）

```
capture [-s IP[:PORT]] [-c IP[:PORT]] [-t TYPES]
```

| 选项 | 说明 |
|------|------|
| `-s IP[:PORT]` | 按**后端** MySQL 地址过滤；`:PORT` 可省略 |
| `-c IP[:PORT]` | 按**客户端**地址过滤 |
| `-t TYPES` | SQL 类型：`DDL`、`DML`、`DQL`、`DCL`、`TCL`，逗号分隔；或 `ALL`（默认全部） |

**管道与 XR:> 的差异**

| 特性 | 管道 `capture` | XR:> `capture` / `cap` |
|------|----------------|-------------------------|
| 输出 | 独立数据管道 NDJSON 流 | 当前窗口 screen / json 文件 |
| `screen`/`json`/`all` 模式 | **不支持**（恒为流式 screen 语义） |
| 别名 `cap` | **不支持**（动词须为 `capture`） |
| 交互键 `P`/`E` | **不支持** | 支持 |
| 白名单 | 须在 `allowed_commands` 含 `capture` | 不受白名单限制 |

**成功响应示例**

```json
{"ok":true,"channel":"\\\\.\\pipe\\DBPrisma_cap_a1b2c3d4","token":"f3e2..."}
```

（Linux 上 `channel` 为 `<run_dir>/DBPrisma_cap_*.sock` 绝对路径。）

**常见错误**

| `error` | 含义 |
|---------|------|
| `command not permitted` | 白名单未含 `capture` |
| `capture already active` | 已有抓取会话，须先 `{"op":"capend"}` |
| `service not running` | 代理未运行 |
| `capture start failed` | 数据通道初始化失败 |

协议细节见 [客户端开发手册 §3](command-pipe-client-guide.md#3-capture-数据管道协议流式-sql-抓取)。停止抓取见 §1A **`capend`**。

---

## 5. 可选扩展命令（非默认白名单）

若将下列动词加入 `command_pipe.allowed_commands`，经管道**可以**执行（未被硬性禁止）：

### `userlist`

```
userlist [--admin]
```

| 参数 | 说明 |
|------|------|
| `--admin` / `-a` | 列出 `admin_users.json`（管道管理员）；省略则列 `proxy_users.json` |

### `checkconfig`

```
checkconfig [config_file]
```

| 参数 | 说明 |
|------|------|
| `config_file` | 要校验的配置路径；省略为默认 `conf.json` |

校验 JSON 语法、必填字段、安全 WARN 等；**不启动**代理。

---

## 6. 命令总表（速查）

### 6.1 管道原生 op（§1A，不经白名单）

| op | 需要 login | 请求帧 | 备注 |
|----|:----------:|--------|------|
| `ping` | 否 | `{"op":"ping"}` | 探活 |
| `whoami` | 是 | `{"op":"whoami","token":"..."}` | 当前管道用户 |
| `metrics` | 是 | `{"op":"metrics","token":"..."}` | Prometheus 文本 |
| `capstatus` | 是 | `{"op":"capstatus","token":"..."}` | 流式抓取状态 |
| `capend` | 是 | `{"op":"capend","token":"..."}` | 停止流式抓取 |

### 6.2 白名单动词（经 `{"op":"cmd","line":"..."}`）

| 动词 | 完整语法 | 管道默认 | 备注 |
|------|----------|:--------:|------|
| `start` | `start [config_file]` | ✓ | |
| `stop` | `stop` | ✓ | |
| `restart` | `restart [config_file]` | ✓ | |
| `reload` | `reload [config_file]` | ✓ | 仅热更字段 |
| `status` | `status [verbose]` | ✓ | `verbose` 含 License 段 |
| `backends` | `backends` | ✓ | |
| `version` | `version` | ✓ | |
| `display` | `display` | ✓ | 读盘 |
| `conns` | `conns [ip]` | ✓ | |
| `pause` | `pause` | ✓ | |
| `resume` | `resume` | ✓ | |
| `drain` | `drain <id>` \| `drain cancel <id>` | ✓ | |
| `flush` | `flush [server_id]` | ✓ | 省略=全部 |
| `kick` | `kick <ip> [port]` \| `kick id <n>` | ✓ | |
| `loglevel` | `loglevel <debug\|info\|warn\|error>` | ✓ | |
| `capture` | `capture [-s …] [-c …] [-t …]` | ✓ | 流式 NDJSON，见 §4 |
| `userlist` | `userlist [--admin]` | — | 须手动加入白名单 |
| `checkconfig` | `checkconfig [config_file]` | — | 须手动加入白名单 |

**永不可经管道**：`setconfig`、`useradd`、`userdel`、`setpwd`、`passwd`、`token`、`service`、`shutdown`、`help`、`history`、`clear`、`exit`、`cap`（请用 `capture`）。

---

## 7. 相关文档

| 文档 | 内容 |
|------|------|
| [command-pipe-client-guide.md](command-pipe-client-guide.md) | 帧协议、登录、capture 数据通道、C#/Java/Python 示例 |
| [configuration-reference.md §二十三](configuration-reference.md) | `command_pipe` 配置键与 `allowed_commands` |
| [admin-manual.md §4.4](admin-manual.md#44-命令管道远程管控command_pipe) | 启用、凭据、测试客户端 |
| [configuration-reference.md §十二](configuration-reference.md) | **XR:> 全量 CLI**（含管道不可用的命令） |

---

*修订历史*

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-06-18 | r1 | 首版：默认白名单 16 动词 + `capend` 全量语法；硬性禁止表；`capture` 管道专语义；可选 `userlist`/`checkconfig` |
| 2026-06-18 | r2 | 新增管道原生 op：`ping`、`whoami`、`metrics`、`capstatus`（§1A） |
| 2026-06-18 | r3 | `capend` 归入 §1A 管道原生 op（`{"op":"capend"}`）；白名单速查表拆为 §6.1/§6.2 |
