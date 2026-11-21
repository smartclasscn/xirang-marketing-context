# DBPrisma Enterprise 配置参考
**日期：** 2026年11月

本文说明 `conf.json` 的配置键。产品全称是息壤数据库串行审计安全网关。配置文件路径是 `bin/<Platform>/<arch>/config/conf.json`。格式为标准 JSON。字段都可以省略；省略时使用各表中的默认值。解析器识别 **162** 个配置键（含 `version`；`servers[]` 每个元素的字段计 1，数组本身计 1）。其中 157 个是标量叶子键，5 个是数组或集合键：`proxy.admin_allow_ips`、`security.ip_allowlist`、`security.ip_denylist`、`security.sql_block_patterns`、`servers`。标量叶子键包含 `paths` 的 3 个运行期目录键：`run_dir`、`log_dir`、`capture_dir`。

配置键、命令和枚举值保持英文。中文说明使用已批准叫法。

## 实现状态总览

网关会解析下表中的键。其中一部分当前未被运行时使用，或已被固定行为取代。`checkconfig` 与 `start` 若发现 `conf.json` **显式写了**这些键，会打印 `[WARN] config: <段>.<键> ...`，避免误以为已经生效。

| 段.键 | 状态 | 说明 |
| --- | --- | --- |
| `resilience.keepalive_idle_sec` / `keepalive_interval_sec` / `keepalive_count` | **已生效** | `backend_tcp_keepalive=true` 时下发。Linux 为 `TCP_KEEPIDLE` / `TCP_KEEPINTVL` / `TCP_KEEPCNT`。Windows 为 `SIO_KEEPALIVE_VALS`（idle 与 interval；count 由操作系统固定） |
| `resilience.retry_on_lost_connection` / `retry_idempotent_only` / `max_retry_count` / `retry_delay_ms` | 保留 | 失败重试未实现 |
| `resilience.auto_failover` / `failover_target_id` / `failover_restore_on_recovery` / `failover_restore_delay_ms` | 部分生效 | `auto_failover` 与 `failover_target_id` 已用于把**新连接**重定向到健康后端。`failover_restore_*` 仍为保留，不会自动切回 primary 配置 |
| `resilience.session_leak_action` | 仅 `log` | `log` 已实现。`force_close` **有意不实现**（TD-006：强制释放卡死会话会绕过 `io_inflight` 门控，引入释放后仍被使用的风险，风险大于收益） |
| `mode.rw_ddl_to_primary` / `rw_dcl_to_primary` / `rw_transaction_detection` | 保留（TD-008） | 非 DQL 与事务已经固定走主库。这些值未被读取 |
| `mode.mirror_enabled` / `mirror_server_id` / `mirror_ratio_pct` / `mirror_async` / `mirror_ddl` | 计划中（TD-005） | 镜像流量未实现。解析和 `checkconfig` 校验已经具备 |
| `security.hide_server_version` | 常开加固（无开关） | 合成握手**始终**上报通用 MySQL 版本号。该字段只为兼容保留，写成 `true` 或 `false` 都没有差别 |
| `security.strip_client_attributes` | 常开加固（无开关） | 客户端 `CLIENT_CONNECT_ATTRS` **始终**被剥离，网关不转发属性载荷。该字段只为兼容保留 |
| `security.max_packet_size_bytes` | **已生效** | 大于 0 时，作为单条客户端命令更严格的字节上限，超出即拒绝或断开。0 表示沿用包重组上限 `proxy.max_reassemble_bytes`。须重启 |
| `circuit_breaker.fallback_policy` | **已生效** | `reject`：OPEN 时快速失败。`queue`：在建立连接处按连接超时预算有界排队，等待熔断恢复，超时再拒绝 |
| `log_config.log_include_sql` | **已生效** | `true` 时，每条捕获语句写入主日志（`SQL session=... sql=...`，遵循 `capture_redact_literals`）。须重启 |
| `config_manage.max_backup_count` | **已生效** | `reload` 与 `setconfig` 写盘都会创建 `conf.json.bak.*`，并按上限裁剪旧备份 |
| `notify.syslog` | **已生效** | `true` 时（仅 Linux）事件写入 syslog。`false` 不写 |
| `notify.win_event_log` | 平台受限 | 仅 Windows 生效 |
| `tls.cipher_list` / `prefer_server_ciphers` / `session_ticket_enabled` | 平台受限 | 仅 OpenSSL/Linux 生效。Windows SChannel 忽略这三项 |

`tls.min_tls_version` 已生效，Linux 与 Windows 都生效。在 OpenSSL 路径上，`cipher_list`、`prefer_server_ciphers`、`session_ticket_enabled` 同时生效。`security.long_query_warn_ms` 已生效：超过阈值时，主日志打出 `[WARN] long query ...`。

上表里标为保留或计划中的键，字段说明放在后文各自的专章，不提前展开。

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

`notify` 段（N-06，操作系统即时通知，含防抖）包括：`win_event_log`（仅 Windows）、`syslog`（仅 Linux）、`debounce_sec`、`on_backend_error`、`on_circuit_breaker`、`on_firewall_block`、`on_auth_fail`、`on_config_reload`。`syslog` 是 Linux 开关，`win_event_log` 是 Windows 开关，各按本平台生效。字段表见后文「notify」。

`command_pipe` 段是可选的带外管理通道，默认禁用。运维客户端通过 Windows 命名管道或 Linux AF_UNIX 套接字登录，并执行管理子命令（`start`、`stop`、`status`、`reload`、`capture` 等）。鉴权用户来自独立的 `admin_users.json`。字段表见后文「command_pipe」。

| 顶层字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `version` | 字符串 | （必填） | 配置文件版本，必须为三段语义化版本（如 `"0.0.1"`，即 `PA_CONFIG_FILE_VERSION` / `PA_VERSION_SEMVER`）。须与引擎 `PA_VERSION_STRING` 的前三段一致。引擎完整版可以含构建号，如 `0.0.1.568`，不要写入配置。省略、写成四段或不匹配时，`checkconfig` 与 `start` 失败。旧的整数键 `config_version` 会报错 |

## proxy

`proxy` 是全局参数。包重组上限的配置键是 `max_reassemble_bytes`。管理接口的监听、令牌和证书也在本段。

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

随包模板 `bin/<Platform>/<arch>/config/conf.json` 把 `connect_timeout_ms` 写成 `1000`、`idle_timeout_sec` 写成 `300`。上面示例是**省略字段时的回退默认**。

| 配置项 | 类型 | 默认值 | 取值范围 | 说明 |
| --- | --- | --- | --- | --- |
| `worker_threads` | 整数 | `0` | `0` 或 `1`–`64` | IOCP/epoll 工作线程数。`0` 表示自动（CPU 核数 × 2）。超出范围则启动失败 |
| `max_clients` | 整数 | `1024` | `1` ~ `1024` | 最大并发客户端连接数（上限 `PA_MAX_CLIENTS=1024`）。超出上限时自动截断 |
| `connect_timeout_ms` | 整数 | `5000` | `> 0` | 连接后端的超时（毫秒）。小于或等于 0 时启动失败 |
| `idle_timeout_sec` | 整数 | `0` | `>= 0` | 客户端空闲超时（秒）。`0` 表示永不超时。小于 0 时启动失败 |
| `max_reassemble_bytes` | 整数 | `16777216` | `65536` ~ `1073741824` | 包重组上限。每个会话、每个方向（c2s / s2c）在堆上重组粘包时，缓冲的增长上限（字节）。缓冲不是按本值一次性预分配，而是从约 16 KiB 起按需成倍增长，最高到该上限。默认 16 MiB。一条逻辑语句可以跨多个 16 MiB 协议包，因此允许大于 16 MiB，上限 1 GiB（与服务端 `max_allowed_packet` 硬上限对齐）。超出 1 GiB 或低于 64 KiB 时自动钳制并打印 `[WARN]`。若配置大于 16 MiB，后端 `max_allowed_packet` 必须不小于该值，同等大小的语句才能被转发 |
| `capture_redact_literals` | 布尔 | 见下 | — | 语句捕获与影子后端告警日志的字面量脱敏。含 `proxy` 段且省略本字段时默认为 `true`。整文件没有 `proxy` 段时为 `false` |
| `allow_legacy_password` | 布尔 | `false` | — | 是否参与 `mysql_old_password`（scramble323）。生产建议保持 `false`。只有仍使用旧式口令的遗留客户端才需要 `true`。支持热重载 |
| `admin_listen_ip` | 字符串 | `127.0.0.1` | — | 管理接口绑定地址。修改后须重启 |
| `admin_listen_port` | 整数 | `0` | `0`–`65535` | 管理接口端口。`0` 表示关闭 |
| `admin_bearer_token` | 字符串 | `""` | — | 非空时，所有端点都要带 Bearer。支持密文格式 `ENC2:`。可以热重载 |
| `admin_allow_ips` | 字符串数组 | `[]` | — | 非空时，只有列出的客户端 IP 可以访问管理接口。可以热重载 |
| `admin_request_timeout_ms` | 整数 | `5000` | — | 管理接口单连接读超时 |
| `admin_max_header_bytes` | 整数 | `8192` | `1024`–`16383` | 管理接口 HTTP 请求头接收缓冲上限。超出 16383 时截断到 16383 |
| `admin_tls_pfx_file` | 字符串 | `""` | — | 管理接口 HTTPS 证书（PFX/PKCS#12），路径相对 `conf.json` 目录。配置后该端口启用 HTTPS。修改后须重启 |
| `admin_tls_pfx_password` | 字符串 | `""` | — | PFX 口令。支持密文格式 `ENC2:`（AES-256-GCM）。修改后须重启 |
| `admin_tls_cert_file` | 字符串 | `""` | — | 管理接口 HTTPS 证书（PEM），须与 `admin_tls_key_file` 成对（`pfx` 优先）。修改后须重启 |
| `admin_tls_key_file` | 字符串 | `""` | — | 管理接口 HTTPS 私钥（PEM）。修改后须重启 |

**管理接口端点（HTTP 与 HTTPS 相同，只读）：** `GET /health`（JSON）、`GET /metrics`（Prometheus 文本）、`GET /events`（最近 64 条事件 JSON）、`GET /audit/search?q=<关键词>&event=<类型>&user=<用户>&tool=<工具>&limit=<N>&offset=<M>`（在审计 NDJSON 中按子串检索。`limit` 默认 20、最大 100。`offset` 跳过前 N 条匹配。响应含 `returned` 与 `has_more`）。其他路径返回 `404`。不是 `GET` 则返回 `405`。

**HTTP / HTTPS 同端口识别：** 管理接口按每个连接的首字节区分协议（TLS 记录以 `0x16` 开头）。

- 配置了证书（`admin_tls_pfx_file`，或 `admin_tls_cert_file` 加 `admin_tls_key_file`）时，同一端口同时接受明文 HTTP 与 HTTPS。
- 未配置证书时，只有 HTTP。
- 配置了证书但加载失败时，启动失败，不会静默降级为明文。

默认只监听本机。绑定 `0.0.0.0` 时必须配置 `admin_bearer_token`，并建议启用 HTTPS。

**大包转发：** 网关在客户端侧按包重组上限 `max_reassemble_bytes` 重组后，整包转发到后端。单条 `COM_QUERY` 应小于该上限，且不超过后端 `max_allowed_packet`。语句捕获最多记录 **4096** 字节（`PA_MAX_SQL_LEN`）。

**内存：** 重组缓冲按需增长。每个会话的 c2s / s2c 缓冲从约 16 KiB/方向 起分配，只有包确实需要更大空间时才成倍增长，上限为 `max_reassemble_bytes`。客户端到网关的传输加密若启用，另有一块解密暂存缓冲，初值约 64 KiB，同样增长到该上限。常见小语句连接的稳态占用是数十 KiB。只有处理接近上限的大包时，单方向才会接近 `max_reassemble_bytes`。最坏上界仍约为每个会话 `2 × max_reassemble_bytes`（双向各一块）。`max_clients=1024` 且 `max_reassemble_bytes=16777216` 时，理论上限约 **32 GiB**。该上界只有在所有连接同时处理满额大包时才会接近。生产环境若同时有高并发和大包，仍应按需调低 `max_reassemble_bytes` 或 `max_clients`。本键**不能热重载**。热重载发现它变化时会记录需要重启，并保持旧上限。

**Linux 上 glibc 的内存归还：** 高并发连接建立与关闭时，可以看到进程 RSS 在前几分钟缓慢上升，然后进入平台期（手册记录：单核 2 GB 测试机上，双向传输加密 30 万次连接后稳定在约 247–286 MB）。在网关到数据库的传输加密路径（`backend_ssl_mode=required`）上做 3000 次连接检查，没有按连接泄漏（只有 2 处进程生命周期内的一次性启动分配）。RSS 的少量上升来自 glibc malloc 多 arena 保留已释放内存，不是泄漏。内存敏感的小内存主机可以在服务环境中设置 `MALLOC_ARENA_MAX=2`（并可配 `MALLOC_TRIM_THRESHOLD_=131072`）。手册记录的 15 分钟检查中，RSS 增幅由约 82 MB 降到约 27 MB，尾部平稳。代码不强制限制 arena 数，以免影响多核主机的分配并发。这是可选的运行环境调整，不是 `conf.json` 的键。

### proxy.startup

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

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `wait_warmup_before_accept` | 布尔 | `true` | `true` 时，等到预热连接池就绪后才接受客户端连接 |
| `warmup_ready_timeout_ms` | 整数 | `30000` | 等待预热就绪的最长时间（毫秒）。超时后无论就绪率如何都继续启动 |
| `warmup_ready_min_pct` | 整数 | `50` | 预热就绪率阈值（0–100%）。达到该比例后即认为预热完成，可以接受连接 |
| `backend_preflight_tcp` | 布尔 | `true` | 启动时对每个启用的后端做一次 TCP 连接探测。失败只记 WARN，不阻止启动 |
| `backend_preflight_timeout_ms` | 整数 | `0` | 探测连接超时（毫秒）。`0` 表示继承 `connect_timeout_ms` |

### proxy.shutdown

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

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `graceful_drain_ms` | 整数 | `15000` | 优雅关闭的排空时长（毫秒）。期间不再接受新连接，等待现有连接完成 |
| `txn_drain_ms` | 整数 | `60000` | 事务排空的最长等待（毫秒）。尽量等待活跃事务提交或回滚 |
| `stop_deadline_ms` | 整数 | `180000` | 整体停止的截止时限（毫秒）。超时后强制终止。`0` 表示不设截止（不推荐） |
| `pool_empty_wait_ms` | 整数 | `15000` | 连接池排空等待（毫秒） |
| `pool_empty_wait_final_ms` | 整数 | `5000` | 连接池最终排空等待（毫秒） |
| `io_join_initial_ms` | 整数 | `45000` | IO 工作线程第一阶段 join 等待（毫秒） |
| `io_join_extended_ms` | 整数 | `120000` | IO 工作线程第二阶段 join 等待（毫秒） |
| `rw_relay_stop_join_ms` | 整数 | `45000` | 读写分离 relay 线程停止等待（毫秒） |
| `reject_errno` | 整数 | `1317` | 排空期间对新建连接返回的错误码 |
| `reject_message` | 字符串 | `"Server shutdown in progress"` | 排空期间返回给客户端的错误消息 |
| `allow_ping` | 布尔 | `true` | 排空期间是否允许 `COM_PING`。`true` 时健康检查仍可通过 |

`startup` 与 `shutdown` 的全部字段都支持热重载。

## servers

`servers` 是数组，最多 16 个（`PA_MAX_SERVERS`）。每个元素是一个监听实例。`scheme` 必须显式配置，用来决定工作模式。

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

### 基础字段

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `id` | 整数 | 自动递增 | 服务标识，只用于日志区分，不影响功能 |
| `host` | 字符串 | `"127.0.0.1"` | 后端服务器 IP 地址 |
| `port` | 整数 | `3306` | 后端端口 |
| `listen_ip` | 字符串 | `"0.0.0.0"` | 网关监听 IP。`"0.0.0.0"` 表示监听所有网卡 |
| `listen_port` | 整数 | `3307` | 网关监听端口（客户端连接此端口） |
| `weight` | 整数 | `1` | `>= 0` |
| `enabled` | 布尔 | `true` | `false` 时该实例不启动监听 |
| `primary` | 布尔 | `false` | 仅当 `mode.strategy` 为 `dispatch`（广播）时有效。把该实例固定为广播的主后端。全组最多一个。不配置时仍按轮询选主 |
| `role` | 字符串 | `""` | 仅当 `mode.strategy` 为 `read_write` 时有效。`"write"`（恰好 1 个 enabled）或 `"read"`（至少 1 个 enabled）。其他 strategy 下设置会 WARN 并忽略 |

### 工作模式

`scheme` 是必填项，用来指定工作模式。

| `scheme` 值 | 工作模式 | 说明 |
| --- | --- | --- |
| `"auth"` | **认证托管** | 网关拦截认证。客户端使用 `proxy_users.json` 中的账号 |
| `"transparent"` | **透明透传** | 客户端认证原样交给后端 |

`scheme` 必填。省略或填写其他值会导致启动失败。不能再靠 `user` 字段推断工作模式。

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `scheme` | 字符串 | — | **必填**。`"auth"` 为认证托管，`"transparent"` 为透明透传。省略或其他值时启动失败 |
| `user` | 字符串 | `""` | 后端账号。认证托管时，这是连接后端的凭据 |
| `password` | 字符串 | `""` | 后端密码。可以是明文，或密文格式 `ENC2:` |

**密码加密格式：**

在交互提示符 `XR:>` 下使用 `passwd`（交互式，推荐）生成加密字符串。也可以用 `DBPrisma setconfig srvpwd <id>` 直接把指定后端的加密密码写入磁盘。加密算法为 AES-256-GCM（密文格式 `ENC2:`，含 IV 与认证标签，并绑定机器）：

```json
"password": "ENC2:rJ3kLmP9..."
```

旧的 `ENC:`（AES-256-CBC）已移除。不再生成，也不再被识别为密文。配置里若残留 `ENC:...`，会被当作明文使用，从而导致认证失败。请用 `passwd` 重新生成 `ENC2:` 串。

### 客户端到网关的传输加密与网关到数据库的传输加密

下列 `tls_termination` 与 `tls_p1_*` 只在认证托管（`scheme` 为 `auth`）时使用。`backend_ssl_mode` 与 `tls_p2_*` 是网关到数据库这一段。

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `tls_termination` | 布尔 | `false` | 仅 `scheme` 为 `"auth"` 时有效。`true` 表示在客户端到网关这一段终结传输加密 |
| `tls_p1_pfx_file` | 字符串 | `""` | PFX/P12 服务端证书（优先）。路径相对 `conf.json` 所在目录 |
| `tls_p1_pfx_password` | 字符串 | `""` | PFX 密码。支持密文格式 `ENC2:`。最大 128 字节（含 `ENC2:` 前缀和结尾的空字符）。超长会被截断，PFX 解码将失败 |
| `tls_p1_cert_file` | 字符串 | `""` | PEM 证书，须与 `tls_p1_key_file` 成对 |
| `tls_p1_key_file` | 字符串 | `""` | PEM 私钥（`BEGIN RSA PRIVATE KEY` 或 PKCS#8 `BEGIN PRIVATE KEY`）。Windows 上椭圆曲线等较少见的密钥仍建议使用 PFX |
| `tls_p1_cert_cn` | 字符串 | `""` | 未配置证书文件时，自签名回退证书的 CN。默认 `DBPrisma` |
| `backend_ssl_mode` | 字符串 | `"disabled"` | 网关到数据库的传输加密策略，对应客户端 `--ssl-mode` 的四档语义 |
| `tls_p2_ca_file` | 字符串 | `""` | 自定义 CA 文件。仅 `verify_ca` / `verify_identity` 使用。PEM（可含多张证书）或 DER。路径相对 `conf.json` 所在目录 |
| `tls_p2_ca_path` | 字符串 | `""` | 自定义 CA 目录。仅校验模式使用。目录内 PEM/DER 文件，不递归。可与 `tls_p2_ca_file` 同时使用，信任合并 |

**客户端到网关的证书格式（`tls_p1_*`）：**

| 格式 | 配置 | 说明 |
| --- | --- | --- |
| PFX / P12 | `tls_p1_pfx_file` + 可选 `tls_p1_pfx_password` | 推荐。可含完整链 |
| PEM 证书 + 私钥 | `tls_p1_cert_file` + `tls_p1_key_file` | 私钥为 `BEGIN RSA PRIVATE KEY` 或 PKCS#8 `BEGIN PRIVATE KEY`。Windows 上椭圆曲线等较少见的密钥仍建议使用 PFX |
| 运行时自签名 | 不配置 `tls_p1_*` | RSA 自签。CN 为 `tls_p1_cert_cn`，未配置时为 `DBPrisma` |
| — | `tls_p1_pfx_file` 与 PEM 成对 | 互斥，不能同时配置 |

**网关到数据库的信任来源（`verify_ca` / `verify_identity`）：**

| 来源 | 配置 | 说明 |
| --- | --- | --- |
| 操作系统信任库 | 均省略 | Windows 证书存储，或 Linux `/etc/ssl/certs` 等 |
| 自定义 CA 文件 | `tls_p2_ca_file` | 类似 `--ssl-ca`。配置后只信任该文件以及可选目录，不使用操作系统信任库 |
| 自定义 CA 目录 | `tls_p2_ca_path` | 类似 `--ssl-capath`。非递归扫描 |
| 不支持 | PFX/P12、JKS、客户端双向证书 | 网关到数据库这一段不向数据库出示客户端证书 |

**`backend_ssl_mode` 可选值：**

| 值 | 加密 | 验证证书链 | 验证主机名 | 说明 |
| --- | --- | --- | --- | --- |
| `disabled` | 否 | 否 | 否 | 明文（默认） |
| `preferred` | 是 | 否 | 否 | 优先加密。后端不宣告 SSL 或握手失败时，降级为明文并 WARN |
| `required` | 是 | 否 | 否 | 强制加密，但不验证后端证书。传输加密启动失败时，降级为明文并 WARN |
| `verify_ca` | 是 | 是 | 否 | 验证 CA 链。默认使用操作系统信任库。配置 `tls_p2_ca_file` 或 `tls_p2_ca_path` 时，只信任自定义来源 |
| `verify_identity` | 是 | 是 | 是 | 完整验证。主机名匹配 `host`。可配置 `tls_p2_ca_file` 或 `tls_p2_ca_path` |

- `verify_ca` / `verify_identity` 在启动时若验证凭据初始化失败，`start` 整体失败，不降级。
- `verify_ca` / `verify_identity` 下，若后端不宣告 `CLIENT_SSL`，或网关到数据库的握手失败，则拒绝连接，不降级为明文。
- `tls_p2_ca_file` / `tls_p2_ca_path` 在 `disabled` / `required` 下会被 WARN 并忽略（`checkconfig`）。
- 网关不会把客户端到网关这一段的 PFX/PEM 当作客户端证书发给数据库。当前没有双向证书。

**`backend_ssl_mode` 按后端角色生效。** 它是逐个 `servers[]` 实例的配置。

| 后端角色 | 适用模式 | 连接来源 | 网关到数据库的传输加密 |
| --- | --- | --- | --- |
| 会话主后端（承载该客户端会话） | 所有 strategy（含广播 `dispatch` 的主后端、`read_write` 且 `role=write` 的主后端） | 会话 `server_sock` | 支持，按该实例的 `backend_ssl_mode` |
| 只读从库（`role=read`） | `read_write` | 惰性只读 `read_sock` | 支持，按该从库实例的 `backend_ssl_mode` |
| 影子后端 | `dispatch` 广播 | 影子后端的异步连接 | 支持，按该影子后端实例的 `backend_ssl_mode` |

从库和影子后端都可以单独配置自己的 `backend_ssl_mode` 与 CA，不必与主库相同。广播时，启用了网关到数据库传输加密的影子后端不复用数据库预热池（预热套接字已是明文，并且已经过了 greeting），改为冷连接再做握手。这类影子后端不走按套接字键的异步发送队列，而是在 `send_lock` 下先加密再阻塞发送，以保证记录顺序。明文影子后端（`disabled`）的发送路径不变。

未配置 `tls_p1_*` 文件时，行为与旧版相同，使用运行时自签名。配置了路径但加载失败时，`start` 整体失败，不降级。

**行为对比：**

| 客户端 ssl-mode | `tls_termination = false`（默认） | `tls_termination = true` |
| --- | --- | --- |
| `DISABLED` | 明文，正常工作 | 明文，正常工作 |
| `PREFERRED` | 明文（网关去掉 CLIENT_SSL） | 客户端选择明文或加密都可以工作 |
| `REQUIRED` | 连接失败（握手错误） | 正常工作（网关终结客户端到网关的传输加密） |
| `VERIFY_CA` / `VERIFY_IDENTITY` | 失败 | 自签名时失败。配置真实 CA 证书后正常 |

**注意事项：**

- 客户端连接时需要 `--ssl-mode=REQUIRED`（或等效参数），也可以使用 `PREFERRED`（网关已声明支持 SSL）。
- 使用自签名时，`VERIFY_CA` / `VERIFY_IDENTITY` 需要关闭校验或导入信任。配置 `tls_p1_pfx_file` 或 PEM 后可以使用正式证书。
- 此功能不影响透明透传，也不影响其他已有功能。
- 示例：

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

### 预热连接

| 配置项 | 类型 | 默认值 | 取值范围 | 适用模式 | 说明 |
| --- | --- | --- | --- | --- | --- |
| `warmup_tcp_connections` | 整数 | `0` | `0` ~ `32` | **已弃用** | 裸 TCP 预热已退役。它是后端 `ERROR 1129 (Host blocked)` 的原因。该字段仅为兼容保留：在认证托管且后端为明文时，折算为认证预热条数，等价于 `warmup_db_connections`。其余情况（透明透传，或后端启用传输加密）会被忽略，并改用冷连接。新配置请直接使用 `warmup_db_connections` |
| `warmup_db_connections` | 整数 | `0` | `0` ~ `32` | **仅认证托管** | 完成握手的预建连接池大小，上限 32。`0` 表示禁用。提前完成握手后，客户端连接时可以跳过这段延迟 |

`warmup_db_connections` 即使在透明透传下写成正整数也不会生效，代码会忽略它。

旧版裸 TCP 预热会向后端建立未完成认证的连接，并在数秒后丢弃。服务端把这计入来源 IP 的连接错误。累计超过 `max_connect_errors`（默认 100）后，会拉黑网关主机，需要反复执行 `mysqladmin flush-hosts`。该机制已退役。有固定凭据且后端为明文时，改为认证预热（成功握手会清零计数）。否则使用冷连接，由真实流量的成功握手自行恢复。`warmup_tcp_connections` 显式大于 0 时，`checkconfig` 与启动日志会给出弃用提示。

`tls_termination` 可以与 `warmup_db_connections` 同时使用。数据库预热握手发出的合成 Greeting 会自动加上 `CLIENT_SSL`，客户端可以升级到传输加密。

## mode

`mode` 是可选的统一入口。启用后，网关在 `listen_port` 上再开一个统一监听端口，按 `strategy` 把客户端连接分到 `servers` 中的各后端。`servers` 里各实例自己的监听端口仍然有效。`mode` 与各实例并行工作，互不影响。

```json
"mode": {
  "enabled":      false,
  "listen_ip":    "0.0.0.0",
  "listen_port":  3300,
  "strategy":     "round_robin"
}
```

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `enabled` | 布尔 | `false` | `true` 才启动统一监听。`false` 时整个 `mode` 段被忽略 |
| `listen_ip` | 字符串 | `"0.0.0.0"` | 统一入口的监听 IP |
| `listen_port` | 整数 | `3300` | 统一入口的监听端口。客户端连接此端口，网关按策略路由 |
| `strategy` | 字符串 | `"round_robin"` | 负载策略，见下表 |

**strategy 可选值：**

| 值 | 说明 |
| --- | --- |
| `"round_robin"` | 轮询（默认）。依次分到各后端 |
| `"weighted"` | 按各 server 的 `weight` 比例分配 |
| `"least_conn"` | 优先选择当前活跃连接数最少的后端 |
| `"dispatch"` | 广播。写操作同步发给所有后端，即多后端同时接收写操作 |
| `"read_write"` | 读写分离。写走 `role=write` 的主库，读走 `role=read` 的从库池 |

**`read_write` 专用字段（`strategy` 为 `read_write` 时生效）：**

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `read_strategy` | 字符串 | `"round_robin"` | 读库池内策略：`round_robin` / `least_conn` / `weighted` |
| `read_after_write_sticky_ms` | 整数 | `0` | DML 之后的 N 毫秒内，SELECT 强制走主库。`0` 表示关闭 |
| `read_fallback_to_primary` | 布尔 | `false` | 读库连接失败时是否静默回落主库。默认否，返回 45001 |
| `read_health_skip_failures` | 整数 | `3` | 读库连续连接失败 N 次后暂时跳过。`0` 表示关闭。与 `resilience` 探测协同 |

**`dispatch` 专用字段：**

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `dispatch_primary_only` | 布尔 | `false` | 应急开关。写只打 primary，跳过向影子后端广播。支持热重载 |

上面四项 `read_write` 字段和 `dispatch_primary_only` 支持热重载。`mode.strategy` 与 `listen_*` 的变更仍须重启。

**语句路由提示（注释，可选）：** `/* master */` 强制主库。`/* slave */` 或 `/* read */` 强制读库。事务内仍然走主库。

**广播与 `primary`：** 可以在某一个 `servers[]` 项上设置 `"primary": true`，使该实例始终作为客户端连接的主后端，响应来自该节点。其余认证托管实例是影子后端。不配置 `primary` 时，每条新连接仍按轮询选主。最多一个 `primary`，且须 `enabled: true`。

`strategy=dispatch` 至少要有 **2** 个 enabled 的认证托管后端（1 个主后端，加上至少 1 个影子后端）。不满足时，`checkconfig` 输出 `[ERROR]`，`start` 失败。单节点广播等同于单写转发，策略本身没有意义。

**影子后端就绪与写语句：** 连接建立后，影子后端在后台异步建连。对 DML/DDL（`INSERT`、`UPDATE`、`DELETE`、`CREATE` 等），网关会先等待影子后端就绪，上限 **3 秒**（`DISPATCH_SHADOW_WAIT_CAP_MS`）。超时或建连结束后仍未凑齐全部影子后端时，向客户端返回 **45000**，不会只写 primary。`SELECT` 等读操作不等待影子后端。

`mode` 统一入口只接受认证托管（`scheme` 为 `"auth"`）的后端。`scheme` 为 `"transparent"` 的 server 在 `checkconfig` / `start` 时输出 Warning，并被排除在 `mode` 路由池之外。若 enabled 的认证托管后端数量为 0，则启动失败。各 server 自己的 `listen_port` 仍按自身 `scheme` 工作，与 `mode` 无关。

读写分离和镜像的扩展字段不在本节展开，见后文「读写分离扩展字段」。

## resilience

`resilience` 可选，用于后端健康。

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

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `enabled` | bool | `true` | 是否启用后端健康探测。`false` 时所有后端视为永久 healthy |
| `probe_interval_sec` | int | `10` | 健康探测间隔（秒，≥1） |
| `probe_timeout_ms` | int | `2000` | 单次探测连接超时（毫秒） |
| `probe_failure_threshold` | int | `3` | 连续失败 N 次后把后端标为 DOWN（≥1） |
| `probe_recovery_threshold` | int | `2` | DOWN 之后连续成功 N 次，标为恢复 UP（≥1） |
| `connect_backoff_base_ms` | int | `100` | 重连退避的基础等待（毫秒） |
| `connect_backoff_max_ms` | int | `30000` | 重连退避的最大等待（毫秒） |
| `connect_backoff_multiplier` | float | `2.0` | 退避倍增系数（`>= 1.0`） |
| `pause_warmup_when_unhealthy` | bool | `true` | 后端不健康时暂停连接池预热 |
| `backend_tcp_keepalive` | bool | `true` | 是否对后端连接开启 TCP Keepalive。不支持热重载，须重启 |
| `dns_rotate_on_failure` | bool | `false` | 连接失败时轮换 DNS 解析结果。适用于多 A 记录的域名 |
| `transparent_reconnect` | bool | `false` | 后端重连时对客户端保持会话 |

`backend_tcp_keepalive` 须重启。其余字段支持热重载。`enabled=false` 时，所有后端视为 healthy。省略整个 `resilience` 段时，默认 `enabled=true`，健康探测默认开启。

keepalive 参数、重试、故障转移和会话泄漏检测见后文「弹性扩展字段」，不并进上表。

## logging

```json
"logging": {
  "file":          "DBPrisma.log",
  "level":         "info",
  "max_size_mb":   50,
  "rotate_count":  3
}
```

上例 `max_size_mb=50` 是示例选值，与随包模板一致。省略字段时的默认是 `10`，见下表。

| 配置项 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `file` | 字符串 | `"DBPrisma.log"` | 日志文件名。相对路径锚定 `log_dir`（默认 `<exe>/logs/`），也可以写绝对路径 |
| `level` | 字符串 | `"info"` | 日志级别：`debug` / `info` / `warn` / `error`（从详到简） |
| `max_size_mb` | 整数 | `10` | 单个日志文件大小上限（MB）。`0` 表示不限制 |
| `rotate_count` | 整数 | `3` | 轮转保留份数。`0` 表示不备份，达到上限后直接截断 |

## 完整示例

### 示例 A：透明透传（最简配置）

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

### 示例 B：认证托管 + 握手预热

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

### 示例 C：多实例（一个透明透传 + 一个认证托管）

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

### 示例 D：校验后端证书 + 自定义 CA（文件 + 目录）

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

### 示例 E：`read_write` + 从库的网关到数据库传输加密

手册写明从库这一路径在 2026-06 已支持。写主库走会话承载后端，其 `backend_ssl_mode` 走会话上的网关到数据库路径。`role=read` 的从库各自按自己的 `backend_ssl_mode` 建立这一段加密。下例中主库为明文，一台从库强制加密（`required`，不验证证书），另一台从库校验证书。需要校验证书时，把从库改为 `verify_ca` 并配置 `tls_p2_ca_file`。

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

联调时发送只读语句（普通 `SELECT`、`COM_STMT_PREPARE` / `EXECUTE`、跨多条记录的大结果集），日志中应出现 `Backend <host>: P2 TLS established`，结果正常返回，且没有解密错误或连接重置。会话主后端、`read_write` 从库和广播的影子后端都支持网关到数据库的传输加密，各按自身 `backend_ssl_mode`。影子后端建连成功时，日志出现 `Dispatch: shadow[i] connected ...`。握手失败时为 `Dispatch: shadow TLS auth failed ...`。

## setconfig

`setconfig` 不启动服务，只编辑 `conf.json`。服务运行中禁止写入。写盘前备份为 `.bak`，写完后自动做 `checkconfig`。

```text
# 查看配置（只读）
setconfig show [all|version|proxy|admin|resilience|startup|shutdown|mode|logging|notify
               |security|capture|circuit_breaker|watchdog|tls|log_config|config_manage|paths
               |command_pipe|server [id]]

# 编辑各段（服务停止时）
setconfig version                    # 编辑 version（三段语义化版本 "0.0.1"；引擎完整版可含构建号，如 0.0.1.568，不要写入配置）
setconfig proxy                      # 代理核心选项（超时、字面量脱敏、旧式口令）
setconfig admin                      # 管理接口选项（独立向导）
setconfig resilience                 # 后端健康探测与退避
setconfig startup                    # 预热与预检
setconfig shutdown                   # 优雅排空与截止时间
setconfig mode                       # 均衡策略、read_write 子字段、dispatch 子字段
setconfig logging                    # 日志文件、级别、轮转（相对路径锚定 log_dir）
setconfig notify                     # 操作系统通知与事件开关
setconfig paths                      # 运行期目录 run_dir / log_dir / capture_dir（改后须重启）
setconfig security                   # IP 封禁、SQL 防火墙、连接限制（改后须重启）
setconfig capture                    # 语句捕获字符集（改后须重启）
setconfig circuit_breaker            # 熔断参数（改后须重启）
setconfig watchdog                   # 进程守护参数（改后须重启）
setconfig tls                        # 全局传输加密版本与密码套件（改后须重启）
setconfig log_config                 # 日志格式与慢查询扩展（改后须重启）
setconfig config_manage              # 热重载备份与回滚（改后须重启）
setconfig command_pipe               # 带外管理通道（enabled / pipe_name / local_only /
                                     #   max_instances / session_idle_timeout_sec / auth_store /
                                     #   capture_queue_mb / capture_buf_kb /
                                     #   allowed_commands；改后须重启）

# server 管理
setconfig server <id>                # 没有则创建，交互修改（含 role）
setconfig server add
setconfig server del <id>
setconfig srvuser <id> <user>        # 设置后端 user（'-' 表示删除）
setconfig srvpwd <id>                # 设置加密的后端密码

# 配置文件控制
setconfig use [path]                 # 本会话默认编辑该文件
setconfig use default                # 恢复 bin\Windows\x64\config\conf.json
setconfig -f <path> <子命令>         # 对这一条命令临时指定配置文件
```

服务运行中，`setconfig` 写盘会被拒绝。请先 `stop` 再编辑，或只改可以热重载的字段，然后执行 `reload`（见后文「热重载（reload）」）。

`setconfig use D:\backup\conf.json` 之后，直接执行 `setconfig server add` 等即操作该文件。`-f` 仍可对单条命令临时指定其他路径。

交互提示：回车保留原值，`-` 从 JSON 中删除该字段。`server` 向导不含 `user` / `password`。后端凭据用 `setconfig srvuser` 与 `setconfig srvpwd`。代理用户仍用 `useradd` / `userdel` / `userlist`，对应 `proxy_users.json`。

`setconfig` 覆盖全部顶层配置段：`proxy`、`admin`、`resilience`、`startup`、`shutdown`、`mode`、`logging`、`security`、`capture`、`circuit_breaker`、`watchdog`、`tls`、`log_config`、`config_manage`、`notify`、`paths`、`command_pipe`、`server`。`security`、`capture`、`circuit_breaker`、`watchdog`、`tls`、`config_manage`、`notify`、`command_pipe` 修改后须重启，不能靠热重载生效。`log_config.log_format` 与 `log_include_sql` 同样须重启。`log_config.slow_query_ms` 与 `slow_query_file` 可以热重载。`log_config.firewall_log_file`、`firewall_log_max_size_mb`、`firewall_log_rotate_count` 可以热重载。`log_config.audit_export_enabled`、`audit_export_format`、`audit_export_file`、`audit_chain_enabled`、`audit_chain_key` 可以热重载。

## 配置检查

配置检查不启动网关，只校验配置文件：

```text
DBPrisma.exe checkconfig
DBPrisma.exe checkconfig D:\path\to\conf.json
```

交互模式下也可以输入 `checkconfig` 或 `cc`（可选配置文件路径）。

检查项包括：JSON 解析、密码解密、`version`、`primary` 是否唯一、端口范围、重复 `id`、监听地址冲突、`mode` 与 `servers` 的端口冲突、`mode.enabled` 时至少要有一个 enabled 的认证托管后端（透明透传计入 Warning）、认证托管是否配置了 `user`、广播的影子后端、`proxy_users.json`。存在 `[ERROR]` 时退出码为 1。

### 审计链校验 checkaudit

`checkaudit` 不启动网关。它校验 `log_config.audit_export_file` 导出的 NDJSON 哈希链。文件中的行须含 `prev_hash` 与 `line_hash`。

```text
DBPrisma.exe checkaudit
DBPrisma.exe checkaudit D:\path\to\audit_export.ndjson
DBPrisma.exe checkaudit D:\path\to\conf.json
```

省略路径时，读取当前 `conf.json`，并解析 `paths.log_dir` 与 `audit_export_file`。没有哈希字段的行计为 `skipped`，不导致失败。

## 热重载（reload）

服务**已在运行**时，`reload` 把磁盘上的 `conf.json` 与内存配置对比，并应用可以热重载的子集。失败则这一次完全不变。

`reload` 是交互提示符 `XR:>` 下的运行时命令，不是单独的外壳子命令：

```text
XR:> reload
XR:> reload D:\path\to\conf.json
```

| 可以热重载 | 不能热重载（须 `stop` 后再 `start`） |
| --- | --- |
| `logging.level` | `servers[]` 的增删、`id`、端口、`enabled`、`scheme` |
| `logging.max_size_mb` / `rotate_count` | `mode.strategy`、`mode.listen_*`、`servers[].role` |
| `proxy.idle_timeout_sec`（调整已有值）¹ | `proxy.max_clients`、`worker_threads`、`max_reassemble_bytes` |
| `proxy.connect_timeout_ms` | `tls_p1_*`、`backend_ssl_mode`、`tls_p2_ca_file`、`tls_p2_ca_path` |
| `proxy.capture_redact_literals` | `version` |
| `proxy.allow_legacy_password` | 监听 IP/端口，以及客户端到网关、网关到数据库的证书与 CA 路径 |
| `mode.read_strategy`、`read_after_write_sticky_ms`、`read_fallback_to_primary`、`read_health_skip_failures` | `proxy.max_clients`、`servers[].user` / `password` / `scheme` |
| `mode.dispatch_primary_only` | `resilience.backend_tcp_keepalive` |
| `admin_bearer_token`、`admin_allow_ips`、`admin_request_timeout_ms`、`admin_max_header_bytes` | `admin_listen_ip`、`admin_listen_port`、`admin_tls_*`（证书、私钥、口令，须重启） |
| `resilience.*`（除 `backend_tcp_keepalive`） | `security.*`（须重启） |
| `proxy.startup.*`、`proxy.shutdown.*` | `capture.*`（须重启） |
| `log_config.slow_query_ms`、`log_config.slow_query_file` | `circuit_breaker.*`（须重启） |
| `log_config.firewall_log_file`、`firewall_log_max_size_mb`、`firewall_log_rotate_count` | `watchdog.*`（须重启） |
| — | `tls.*`（须重启） |
| — | `log_config.log_format`、`log_include_sql`（须重启） |
| — | `paths.*`（`run_dir` / `log_dir` / `capture_dir`，须重启） |
| — | `config_manage.*`（配置段本身须重启；`backup_on_reload` 已在热重载流程中使用） |
| — | `notify.*`（须重启） |
| — | `command_pipe.*`（须重启。管道在启动时按配置创建） |

¹ `idle_timeout_sec` 的特别说明见下。

`reload` 只能调整已经在运行的空闲计时的阈值，也就是启动时 `idle_timeout_sec > 0` 的情况。若启动时该值为 `0`，计时并未创建。此后用热重载改成非零，不会开始计时，日志会输出 WARN。须执行 `stop` 再 `start`，空闲超时才会生效。

`security` 须重启。`setconfig security` 写盘后，必须 `stop` 再 `start`，新的安全策略才会生效。热重载不会更新内存中的安全策略（IP 封禁、熔断阈值等）。

`useradd` / `setpwd` 之后，`proxy_users.json` 由 `PA_ReloadProxyUsers` 生效。不必重启，也不必再 `reload` 整份 `conf.json`。

## proxy_users.json

路径：`bin/<Platform>/<arch>/config/proxy_users.json`。本文件用于认证托管。

当认证托管的 `proxy_users.json` 存在且非空时，只有列表中的用户能通过网关认证。文件为空或不存在时，允许任意用户名和密码。此时连接后端的账号仍以 `conf.json` 中的 `user` / `password` 为准。

```json
{
  "users": [
    { "username": "alice", "password": "明文密码" },
    { "username": "bob",   "password": "ENC2:加密密码" }
  ]
}
```

| 字段 | 说明 |
| --- | --- |
| `username` | 客户端连接网关时使用的用户名 |
| `password` | 对应密码。支持明文或密文格式 `ENC2:` |

代理用户与后端用户完全独立。客户端用 `alice` 连接网关，网关实际用 `conf.json` 里配置的后端账号连接数据库。明文密码会打出 `[SECURITY WARN]`。建议用 `setpwd` 加密。

## CLI token

路径：`<exe>/run/dbprisma.cli_token`（默认。随 `--run-dir`、`DBPRISMA_HOME`、`paths.run_dir` 变化）。

| 项 | 说明 |
| --- | --- |
| 生成 | 首次 `start` 时创建 32 字节随机十六进制串 |
| 权限 | 仅进程属主可读（Windows DACL / POSIX 0600） |
| 受保护的命令 | `stop`、`restart`、`reload`、`setconfig`、`passwd`、`setpwd`、`useradd`、`userdel` 等 |
| 交互式终端 | 控制台人工操作可以不带 token |
| 脚本 | `--token <hex>`，或环境变量 `DBPRISMA_CLI_TOKEN` |

只读命令（如 `status`）不需要 token。

## 命令行

### 进程级启动参数

用法：`DBPrisma [OPTIONS] [SUBCOMMAND [args...]]`

| 参数 | 别名 | 是否带值 | 说明 |
| --- | --- | --- | --- |
| `--config <file>` | `-c` | 是 | 指定配置文件（默认 `<config-dir>/conf.json`） |
| `--config-dir <dir>` | — | 是 | `conf.json`、`proxy_users.json` 和备份所在目录（默认 `<exe>/config/`） |
| `--run-dir <dir>` | — | 是 | `lock`、`pid`、`cli_token`、`cli_history`、`machine.key` 所在目录（默认 `<exe>/run/`） |
| `--log-dir <dir>` | — | 是 | 主日志、防火墙日志、慢查询日志目录（默认 `<exe>/logs/`） |
| `--capture-dir <dir>` | — | 是 | `capture.json` 目录（默认 `<exe>/capture/`） |
| `--token <hex>` | — | 是 | CLI 鉴权 token（64 位十六进制，32 字节）。非交互模式下的特权命令需要它。也可由环境变量 `DBPRISMA_CLI_TOKEN` 设置 |
| `--log-level <level>` | — | 是 | 覆盖配置文件中的日志级别（`debug`、`info`、`warn`、`error`）。只对本次运行生效 |
| `--bg-capture` | — | 否 | 启动时立即把语句捕获到 `capture.json` |
| `--no-interactive` | — | 否 | 不进入 `XR:>`。Ctrl+C 或 stop 时正常退出 |
| `--single-instance` | — | 否 | 若已有另一个实例在运行，则拒绝启动 |
| `--pid-file <path>` | — | 是 | 服务启动成功后把 PID 写入该文件（默认 `<run-dir>/dbprisma.pid`） |
| `--print-config` | — | 否 | 加载 `conf.json`，打印生效配置后退出 |
| `--warmup-only` | — | 否 | 做完预检和预热后退出，用于就绪检查 |
| `--version` | `-v` | 否 | 打印版本信息后退出 |
| `--help` | `-h` | 否 | 打印帮助后退出 |

`checkconfig` / `cc`、`setconfig` / `sc`、`service` 必须作为第一个参数，位于所有选项之前，否则报错退出（`subcommand '<x>' must come before options`）。`start` 不受此限，选项可以放在它前面或后面。带值参数同时接受 `--opt value` 与 `--opt=value`。`--` 表示选项结束。无法识别的选项或拼错的子命令会报错退出，并给出最接近的拼写建议。

**进程级子命令（执行后立即退出）：**

| 子命令 | 别名 | 参数 | 说明 |
| --- | --- | --- | --- |
| `start [config_file]` | — | 可选配置路径 | 启动 |
| `checkconfig [config_file]` | `cc` | 可选配置路径 | 只校验配置，不启动 |
| `setconfig <verb> [args...]` | `sc` | 见「setconfig」 | 编辑磁盘上的配置文件后退出 |
| `service <install\|uninstall\|start\|stop\|status>` | — | — | 管理操作系统服务（需要管理员或 root） |

### 交互命令

交互提示符为 `XR:>`。

| 命令 | 参数 | 需要 token | 说明 |
| --- | --- | --- | --- |
| `start` | `[config_file]` | 否 | 启动（默认 `conf.json`） |
| `stop` | — | **是** | 优雅停止（等同于 `shutdown 0`） |
| `restart` | `[config_file]` | **是** | 优雅重启（stop 再 start） |
| `reload` | `[config_file]` | **是** | 热重载可以更新的字段（见「热重载（reload）」） |
| `shutdown` | `[<秒> [force\|cancel]]` | **是** | 带倒计时的优雅停止。`shutdown 30` 表示 30 秒后停止。`shutdown 0 force` 立即强停。`shutdown cancel` 取消等待中的停止 |
| `status` | `[verbose]` | 否 | 运行时统计。`verbose` 显示版本、模式、后端和连接池 |
| `backends` | — | 否 | 列出后端状态（在线或离线、连接数、熔断状态） |
| `drain` | `<server_id>`，或 `cancel <server_id>` | **是** | 排空（优雅下线）。`drain <id>` 停止把新连接送到该后端，已有连接自然结束。`drain cancel <id>` 恢复路由。必须指定 `server_id`，没有“全部排空” |
| `pause` | — | **是** | 暂停接受新的客户端连接，不断开已有连接 |
| `resume` | — | **是** | 恢复接受客户端连接 |
| `flush` | `[server_id]` | **是** | 清空后端预热连接池并在后台重建。省略参数则清空所有后端。指定 `server_id` 则只清空该后端。不断开活跃会话 |
| `conns` | `[ip]` | 否 | 当前活跃客户端连接。可以按客户端 IP 过滤 |
| `kick` | `<client_ip> [port]`，或 `id <n>` | **是** | 强制断开客户端。`port=0` 断开该 IP 的全部连接。`id <n>` 断开 `conns` 输出的第 N 行 |
| `loglevel` | `<debug\|info\|warn\|error>` | **是** | 运行时变更日志级别 |
| `capture` | `[-s IP:PORT] [-c IP:PORT] [-t TYPES] [MODE]` | 否 | 实时语句捕获（见下） |
| `display` | — | 否 | 显示当前加载的配置 |
| `checkconfig` | `[config_file]` | 否 | 校验配置的语法与语义 |
| `setconfig` | `<verb> [args...]` | **是** | 编辑配置（见「setconfig」） |
| `useradd` | `[--admin] <username>` | **是** | 创建用户，交互输入密码。默认写入 `proxy_users.json`。加 `--admin` 则写入 `admin_users.json`（带外通道的管理员） |
| `userdel` | `[--admin] <username>` | **是** | 删除用户。加 `--admin` 则操作管理员库 |
| `userlist` | `[--admin]` | 否 | 列出用户。加 `--admin` 则列出带外通道的管理员 |
| `setpwd` | `[--admin] <username>` | **是** | 修改用户密码。加 `--admin` 则操作管理员库 |
| `passwd` | `[明文]` | **是** | 把密码加密为密文格式 `ENC2:` |
| `token rotate` | — | **是** | 重新生成 CLI 鉴权 token |
| `service` | `<install\|uninstall\|start\|stop\|status>` | **是** | 操作系统服务管理 |
| `version` | — | 否 | 版本、平台与构建信息 |
| `clear` / `cls` | — | 否 | 清屏 |
| `history` / `hist` | — | 否 | 列出命令历史（128 条环形缓冲，持久化到 `<run>/dbprisma.cli_history`，跨会话保留。用上箭头和下箭头调取） |
| `help` / `?` | `[command]` | 否 | 显示全部命令。带命令名时显示该命令的用法 |
| `exit` / `quit` | — | — | 退出。服务仍在运行时会先 stop |

**`capture` 选项：**

| 标志 | 说明 |
| --- | --- |
| `-s IP:PORT` | 只看指定后端 |
| `-c IP:PORT` | 只看指定客户端 |
| `-t TYPES` | 语句类型：`DDL,DML,DQL,DCL,TCL`，或 `all`（默认 all） |
| `MODE` | 输出：`screen`（默认）/ `json`（写入 `capture.json`）/ `all` |

捕获界面：`P` / `p` 暂停或恢复打印。`E` / `e` 退出捕获。

### 环境变量

只读取下表中的环境变量。其余配置一律来自 `conf.json` 或命令行参数。

| 变量 | 说明 |
| --- | --- |
| `DBPRISMA_CLI_TOKEN` | 非交互模式下特权命令的鉴权 token，等价于 `--token <hex>` |
| `DBPRISMA_HOME` | 设为根目录后，四个目录解析为 `$DBPRISMA_HOME/config`、`run`、`log`、`capture`。优先级低于 `--config-dir`、`-c` 和 `--*-dir`，高于 `conf.json` 的 `paths` 以及 `<exe>` 默认值 |
| `NO_COLOR` | 设为任意值即关闭彩色终端输出 |

## security

安全策略配置段名是 `security`。本段不支持热重载。`setconfig security` 改完磁盘配置后，须 `stop` 再 `start` 才会生效。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `auth_fail_max` | int | `5` | 触发封禁的认证失败次数（1–100） |
| `auth_fail_window_ms` | int | `60000` | 滑动窗口时长（毫秒，≥1000） |
| `auth_fail_ban_ms` | int | `300000` | 封禁持续时长（毫秒，0 表示永久） |
| `max_connections_per_ip` | int | `0` | 单个 IP 的并发连接上限（0 表示禁用） |
| `max_connections_per_user` | int | `0` | 单个用户的并发连接上限。认证托管使用代理用户，透明透传使用握手用户名。0 表示禁用 |
| `max_queries_per_user` | int | `0` | 单个用户在滑动窗口内允许转发的语句条数（0 表示禁用） |
| `query_rate_window_ms` | int | `60000` | 语句速率统计窗口（毫秒，≥1000）。`max_queries_per_user>0` 时生效 |
| `new_conn_burst_limit` | int | `0` | 单个窗口内新建连接数上限（令牌桶，0 表示禁用） |
| `new_conn_burst_window_ms` | int | `1000` | 突发窗口（毫秒） |
| `query_timeout_ms` | int | `0` | 单条语句执行超时（毫秒）。超时后关闭会话。0 表示禁用 |
| `long_query_warn_ms` | int | `0` | 慢语句 WARN 阈值（毫秒）。不关闭会话。0 表示禁用 |
| `max_packet_size_bytes` | int | `0` | 客户端包的最大字节数。0 表示沿用包重组上限 `max_reassemble_bytes` |
| `hide_server_version` | bool | `false` | 常开加固，没有开关作用。合成握手始终上报通用 MySQL 版本号。字段只为兼容保留 |
| `strip_client_attributes` | bool | `false` | 常开加固，没有开关作用。转发到后端前始终去掉 `CLIENT_CONNECT_ATTRS` 载荷。网关向客户端握手时仍协商该能力，以便在本地提取 `tool_name`（`program_name` / `_client_name`）。字段只为兼容保留 |
| `mask_client_ip_in_log` | bool | `false` | 只在日志里对客户端 IP 做部分脱敏（IPv4 显示为 `A.B.x.x`，IPv6 显示为 `a:b::x`）。限流、`kick` 和捕获仍使用真实 IP |
| `sql_firewall_enabled` | bool | `false` | 启用 SQL 防火墙 |
| `sql_firewall_mode` | string | `"blocklist"` | `blocklist`：命中即拦，默认，与旧版一致。`allowlist`：只允许命中规则的语句，且至少要有一条 `sql_block_patterns` |
| `sql_firewall_action` | string | `"reject"` | `reject`：返回错误 1148 并丢弃命令。`log_only`：只记日志，放行 |
| `sql_firewall_log_blocked` | bool | `true` | 命中时把 `FIREWALL_BLOCK` WARN 写入主日志（含命中的 pattern，以及语句前 120 字节）。配合 `log_config.firewall_log_file` 可同时写入独立文件 |
| `sql_block_patterns` | array | `[]` | 全局阻断模式，最多 32 条，每条最长 256 字节。默认按大小写不敏感的子串匹配。以 `regex:` 前缀启用受限正则（支持 `. * + ^ $ \s \d \w`。未用 `^` 锚定时按子串匹配）。第一条命中即触发。对 `COM_STMT_PREPARE` 与 `COM_STMT_EXECUTE` 做两阶段检查（`fw_stmt_map`）。在 `ssl_passthrough` 下不可用。没有匹配范围的 `sql_access_policies` 时，回退到本列表 |
| `sql_access_policies` | array | `[]` | 按用户和库的细粒度策略，最多 `PA_MAX_SQL_ACCESS_POLICIES` 条。每条含 `users[]`、`databases[]`（空表示全部）、`patterns[]`（语义同 `sql_block_patterns`，含 `regex:`）、`mode`（`blocklist` / `allowlist`）、`action`（`reject` / `log_only` / `audit_only`）、`priority`（整数，默认 `0`。多条范围同时命中时取最高）。未命中任何策略范围时，回退到全局 `sql_block_patterns` |
| `access_schedule` | object | 见下 | 访问时段，使用本地系统时间。`enabled=false` 时忽略。`enabled=true` 时，窗口外的语句返回 **1227**。与 SQL 防火墙独立，可以单独启用 |
| `access_schedule.enabled` | bool | `false` | 是否启用时段控制 |
| `access_schedule.weekdays` | array | `[]` | 允许的工作日：`0` 为周日，直到 `6` 为周六（与 `SYSTEMTIME.wDayOfWeek` 一致）。`enabled=true` 时不能为空 |
| `access_schedule.start` | string | `"00:00"` | 窗口开始 `HH:MM`（含）。小时必须是两位 |
| `access_schedule.end` | string | `"23:59"` | 窗口结束 `HH:MM`。同一天的窗口不含该分钟。`start` 与 `end` 相同表示全天 |
| `ip_allowlist` | array | `[]` | 永不被限速或封禁的 IP/CIDR，最多 64 条。支持 `10.0.0.0/8`、`2001:db8::/32` 等 |
| `ip_denylist` | array | `[]` | 始终拒绝连接的 IP/CIDR，最多 64 条 |

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

`ip_allowlist` 与 `ip_denylist` 支持主机 IP（如 `10.0.0.1`）或 CIDR（如 `192.168.1.0/24`、`2001:db8::/32`）。IPv4 与 IPv6 必须同族，IPv4 的 CIDR 不匹配 IPv6 客户端。`checkconfig` 会校验条目格式。允许名单与拒绝名单重叠时给出 WARN。运行时允许名单优先。

## capture

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `charset` | string | `"auto"` | 捕获语句使用的字符集：`auto`、`utf8mb4`、`gbk`、`latin1` 等 |
| `charset_auto_detect` | bool | `true` | 从服务端握手包中识别字符集。识别结果覆盖 `charset` |

`charset` 与 `charset_auto_detect` 已实现。握手阶段从服务端 HandshakeV10 取出字符集名称。写入捕获记录时的优先级是：自动识别、显式 `charset`、再降为 `utf8mb4`。

`capture` 的全部字段须 `stop` 再 `start` 才生效，不支持热重载。

## 熔断（circuit_breaker）

每个后端各自维护一套熔断状态：`CLOSED` → `OPEN` → `HALF_OPEN` → `CLOSED`。配置段名是 `circuit_breaker`。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `enabled` | bool | `false` | 是否启用熔断 |
| `error_rate_threshold_pct` | int | `50` | 错误率阈值（%，1–100）。超过则进入 OPEN |
| `min_requests` | int | `20` | 评估前的最少请求数。样本不足时不进入 OPEN |
| `window_ms` | int | `60000` | 统计窗口（毫秒） |
| `open_duration_ms` | int | `30000` | OPEN 持续时间（毫秒）。之后进入 HALF_OPEN |
| `half_open_probe_count` | int | `3` | HALF_OPEN 时允许的探测请求数 |
| `fallback_policy` | string | `"reject"` | OPEN 期间新连接的处理。`reject` 表示立即失败并关闭客户端。`queue` 表示在建立后端连接处有界排队：按该后端连接超时（单转发 `connect_timeout_ms` / mode `timeout_ms`，小于或等于 0 时回落 5000 毫秒）每 50 毫秒轮询。熔断转入 HALF_OPEN 或 CLOSED 即放行，排队中的请求成为探测。超时仍未恢复则拒绝。只作用于新连接的建立，不影响已经在途的会话和中途重连 |

## 进程守护（watchdog）

配置段名是 `watchdog`。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `enabled` | bool | `false` | 是否启用进程守护线程 |
| `heartbeat_interval_ms` | int | `5000` | 工作线程心跳检测周期（毫秒，≥500） |
| `max_missed_heartbeats` | int | `3` | 连续丢失心跳达到该次数后触发动作 |
| `action` | string | `"log_only"` | `log_only`、`restart` 或 `core_dump_and_restart` |
| `restart_delay_ms` | int | `1000` | 触发 restart 之前的等待（毫秒） |

`restart`：设置重启标志。宿主进程在守护或服务模式的保活循环里，大约每 5 秒经 `PA_GetStatus` 查看该标志。命中后优雅 `stop`，并以退出码 **42** 退出，由外部监督程序重新拉起（Windows 服务恢复，或 systemd 的 `Restart=`）。交互提示符不轮询此标志。

`core_dump_and_restart`：Windows 生成 minidump，Linux 发送 SIGABRT，然后与 `restart` 一样设置重启标志。

## tls

本段是全局传输加密策略，用于客户端到网关以及网关到数据库的加密上下文。配置段名是 `tls`。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `min_tls_version` | string | `"TLSv1.2"` | 最低版本：`TLSv1.2`、`TLSv1.3`，或 `""`（操作系统默认） |
| `cipher_list` | string | `""` | OpenSSL 上 TLS 1.2 的密码套件字符串。空表示内置列表 |
| `prefer_server_ciphers` | bool | `true` | 是否优先使用服务端的密码套件顺序 |
| `session_ticket_enabled` | bool | `false` | 是否启用 TLS Session Ticket。关闭可以加强前向安全 |

`cipher_list`、`prefer_server_ciphers`、`session_ticket_enabled` 只在 OpenSSL（Linux）上生效。Windows SChannel 忽略这三项。`min_tls_version` 在两边都生效。SChannel 通过 `SCHANNEL_CRED` 应用它。

## log_config

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `log_format` | string | `"text"` | 日志格式：`text` 或 `json` |
| `slow_query_ms` | int | `0` | 慢语句日志阈值（毫秒，单独记录）。0 表示禁用 |
| `slow_query_file` | string | `""` | 慢语句日志路径。空表示写入主日志 |
| `log_include_sql` | bool | `false` | 日志行中包含原始语句，并遵循 `capture_redact_literals` |
| `firewall_log_file` | string | `""` | SQL 防火墙审计日志的专用文件。空表示不写独立文件（`FIREWALL_BLOCK` 仍写主日志）。支持热重载 |
| `firewall_log_max_size_mb` | int | `0` | 防火墙日志滚动阈值（MB）。0 表示不限大小 |
| `firewall_log_rotate_count` | int | `5` | 滚动保留文件数（命名 `.1`、`.2`…）。0 表示截断且不保留 |
| `audit_export_enabled` | bool | `false` | 是否写入审计导出文件，格式便于安全信息与事件管理（SIEM）使用。独立于 `capture.json` |
| `audit_export_format` | string | `"json"` | 导出格式：`json`（NDJSON）、`cef` 或 `syslog`。支持热重载 |
| `audit_export_file` | string | `""` | 审计导出文件路径。相对路径锚定 `paths.log_dir`。`enabled=true` 时必填 |
| `audit_chain_enabled` | bool | `false` | 为 NDJSON 导出行附加 `prev_hash` 与 `line_hash`（SHA-256 链）。支持热重载 |
| `audit_chain_key` | string | `""` | 哈希链盐值。空则使用由机器指纹派生的默认值。支持热重载 |

## config_manage

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `backup_on_reload` | bool | `true` | 热重载前自动备份 `conf.json`，命名为 `conf.json.bak.YYYYMMDD_HHMMSS`。`false` 时跳过 |
| `max_backup_count` | int | `5` | 同一目录最多保留的 `.bak.*` 份数。0 表示不限。`reload` 与 `setconfig` 写盘都遵守 |
| `rollback_on_reload_fail` | bool | `true` | 热重载解析或校验失败时，自动恢复最近一次备份。`setconfig` 校验失败同样回滚 |

## paths

运行期文件分布在四个子目录，默认都锚定可执行文件目录 `<exe>`。解析优先级是：命令行参数，然后是环境变量 `DBPRISMA_HOME`，然后是 `conf.json` 的 `paths`，最后是 `<exe>/<子目录>`。

| 字段 | 类型 | 默认值 | 命令行覆盖 | 说明 |
| --- | --- | --- | --- | --- |
| `run_dir` | string | `run`（即 `<exe>/run/`） | `--run-dir` | `dbprisma.lock`、`dbprisma.pid`、`cli_token`、`cli_history`、`machine.key` |
| `log_dir` | string | `logs`（即 `<exe>/logs/`） | `--log-dir` | 主日志（`logging.file`）、`log_config.firewall_log_file`、`log_config.slow_query_file` 的相对路径锚点 |
| `capture_dir` | string | `capture`（即 `<exe>/capture/`） | `--capture-dir` | `capture.json` 及其轮转副本 |

`config_dir`（`conf.json`、`proxy_users.json` 和备份所在目录）不能在 `conf.json` 里指向自己。只能通过 `-c` / `--config <文件>`、`--config-dir <目录>` 或 `DBPRISMA_HOME` 指定。默认是 `<exe>/config/`。

- `paths.*` 的相对路径锚定 `<exe>`。绝对路径按原样使用。
- 设置 `DBPRISMA_HOME=/srv/dbprisma` 会把四个子目录都放到该根下，且优先级高于 `conf.json`。
- `logging.file`、`firewall_log_file`、`slow_query_file` 若为相对路径，统一锚定到 `log_dir`。绝对路径不受影响。
- `machine.key`（`run/dbprisma.machine.key`）是历史遗留。机器身份已不再依赖此文件。新装可以忽略。它仍列在 `run_dir` 的解析结果中，以便兼容旧部署。
- 目录在启动时按需创建。
- `setconfig paths` 可以交互编辑 `run_dir`、`log_dir`、`capture_dir`。

```json
"paths": {
    "run_dir":     "run",
    "log_dir":     "logs",
    "capture_dir": "capture"
}
```

## 弹性扩展字段

下列字段写在 `resilience` 段里，是前文健康探测表之外的扩展。本节单独列出，不并进前文。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `keepalive_idle_sec` | int | `60` | 空闲多久后开始探测。`backend_tcp_keepalive=true` 时生效。Linux 对应 `TCP_KEEPIDLE`。Windows 对应 `SIO_KEEPALIVE_VALS.keepalivetime`。在新建后端连接时下发 |
| `keepalive_interval_sec` | int | `10` | 探测间隔。Linux 对应 `TCP_KEEPINTVL`。Windows 对应 `SIO_KEEPALIVE_VALS.keepaliveinterval` |
| `keepalive_count` | int | `5` | 探测失败多少次后判定断开。Linux 对应 `TCP_KEEPCNT`。Windows 不可调，由操作系统固定 |
| `retry_on_lost_connection` | bool | `false` | 保留，未实现。计划在检测到连接丢失时自动重试 |
| `retry_idempotent_only` | bool | `true` | 保留，未实现。计划只对幂等语句（SELECT/SHOW）重试 |
| `max_retry_count` | int | `1` | 保留，未实现。计划中的最大重试次数 |
| `retry_delay_ms` | int | `50` | 保留，未实现。计划中的重试前等待（毫秒） |
| `auto_failover` | bool | `false` | 主后端被标为不健康时，**新连接**自动转到 `failover_target_id`（若该后端健康）或下一个健康后端，并写 WARN。不影响已经建立的会话。`failover_restore_on_recovery` 与 `failover_restore_delay_ms` 仍为保留，不会自动恢复 primary 路由 |
| `failover_target_id` | int | `0` | 转移目标的 `server_id`。`0` 表示选第一个健康的其他 enabled 后端 |
| `failover_restore_on_recovery` | bool | `true` | 保留，未实现。计划在主后端恢复后自动切回 |
| `failover_restore_delay_ms` | int | `30000` | 保留，未实现。计划中的切回前等待（毫秒） |
| `session_leak_detect` | bool | `false` | 启用会话泄漏检测。只记日志的部分已实现：后台线程每 5 秒扫描处于 `SESSION_CLOSING` 且超过阈值的会话，并写 WARN |
| `session_leak_threshold_sec` | int | `300` | `SESSION_CLOSING` 超时阈值（秒，≥10）。低于 10 时，启动 WARN 提示可能误报 |
| `session_leak_action` | string | `"log"` | `log` 已实现。`force_close` 有意不实现：写上不会报错，但没有强制关闭行为。检测器只盯已经进入 `SESSION_CLOSING`、套接字也已全部关闭的卡死会话。卡死原因是 `io_inflight>0`（完成通知尚未回来）。要强制释放，只能绕过 `io_inflight==0` 的门控直接释放，而这正是引擎用门控防止的释放后仍被使用。因此保持只记日志，由运维排查（TD-006） |

## 读写分离扩展字段

下列字段写在 `mode` 段里。当前版本已经完成解析与校验。部分行为在引擎侧尚未按这些开关改变，见各字段说明。本节单独列出，不并进前文「mode」。

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `rw_ddl_to_primary` | bool | `true` | DDL（CREATE/ALTER/DROP 等）强制送到写库。当前行为已经等同于 `true`：非 DQL 一律走主库。把字段改成 `false` 没有运行效果。引擎没有把 DDL 送到读库的路径（TD-008） |
| `rw_dcl_to_primary` | bool | `true` | DCL（GRANT/REVOKE 等）强制送到写库。当前行为已经等同于 `true`。改成 `false` 无效（TD-008） |
| `rw_transaction_detection` | string | `"pessimistic"` | 事务检测。引擎内置的检测与默认值一致。`"optimistic"` 未实现。修改此字段没有运行效果（TD-008） |
| `mirror_enabled` | bool | `false` | 启用流量镜像。功能尚未实现。当前只解析配置，不执行镜像 |
| `mirror_server_id` | int | `0` | 镜像目标的 server_id。同上，尚未实现 |
| `mirror_ratio_pct` | int | `100` | 镜像比例（1–100）。同上，尚未实现 |
| `mirror_async` | bool | `true` | 镜像是否异步发送。同上，尚未实现 |
| `mirror_ddl` | bool | `false` | 是否镜像 DDL。同上，尚未实现 |

## notify

`notify` 控制关键事件是否写入操作系统事件日志（Windows 事件日志或 syslog）。

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

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `win_event_log` | bool | `true` | 是否写入 Windows 事件日志。仅 Windows 有效 |
| `syslog` | bool | `false` | 是否写入 syslog。仅 Linux 有效 |
| `debounce_sec` | int | `60` | 同一事件连续通知的最小间隔（秒），用于防抖，避免日志风暴 |
| `on_backend_error` | bool | `true` | 后端连接失败或 DOWN 时通知 |
| `on_circuit_breaker` | bool | `true` | 熔断进入 OPEN 或 HALF_OPEN 时通知 |
| `on_firewall_block` | bool | `false` | SQL 防火墙拦截时通知 |
| `on_auth_fail` | bool | `true` | 认证失败超过阈值时通知 |
| `on_config_reload` | bool | `false` | 配置热重载成功或失败时通知 |

整个 `notify` 段可以省略，此时使用上表默认值。本段不支持热重载。修改后须 `stop` 再 `start`。

## command_pipe

`command_pipe` 让运维客户端通过本地进程间通信登录并执行管理子命令，而不必进入 `XR:>`。适用于服务模式：Linux 的 systemd 或 `--no-interactive`，Windows 的服务控制管理器或 `--no-interactive`。交互模式下不启用，也不影响 `XR:>`。控制通道与网关启动分开。即使核心启动失败，管道仍可用于远程诊断和恢复。

- Windows：命名管道 `\\.\pipe\<pipe_name>`
- Linux：AF_UNIX 套接字 `<run_dir>/<pipe_name>.sock`

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

| 字段 | 类型 | 默认值 | 说明 |
| --- | --- | --- | --- |
| `enabled` | bool | `false` | 是否启动控制通道。只在守护或服务模式生效 |
| `pipe_name` | 字符串 | `DBPrisma_ctrl` | 管道或套接字的基名，路径见上面的平台说明 |
| `local_only` | bool | `true` | 收紧访问控制，只允许本机客户端。建议保持 `true` |
| `max_instances` | int | `4` | 并发客户端上限。运行时钳制到 1–64 |
| `session_idle_timeout_sec` | int | `300` | 会话空闲超时（秒）。0 表示禁用 |
| `auth_store` | 字符串 | `admin_users.json` | 管理员凭据文件，相对 config 目录 |
| `capture_queue_mb` | int | `256` | `capture` 流式抓取的内存队列上限（MB）。运行时钳制到 1–4096。超出时丢弃最旧记录，并用 `{"_dropped":N}` 通知客户端。Windows 与 Linux 相同 |
| `capture_buf_kb` | int | `64` | `capture` 数据通道的传输缓冲（KB）。运行时钳制到 4–1024。Windows 为命名管道收发缓冲。Linux 为 AF_UNIX 套接字的 `SO_SNDBUF` / `SO_RCVBUF`。两边相同 |
| `allowed_commands` | 字符串数组 | 见下 | 允许经管道执行的子命令。为空时使用内置默认集 |

本段不支持热重载。修改后须重启，或重启操作系统服务。`checkconfig` 会检查：`local_only=false` 时给出安全 WARN；`auth_store` 不存在或没有用户时给出 WARN；正常时打印 `[OK] ... (N admin users)`。

### 鉴权文件 admin_users.json

`admin_users.json` 与 `proxy_users.json` 完全独立。管道登录用户不是代理用户，也不是后端用户。路径默认是 `config/admin_users.json`，由 `auth_store` 指定。

```json
{
  "users": [
    { "username": "admin", "password": "ENC2:加密密码" }
  ]
}
```

建议用命令管理。`useradd`、`userdel`、`userlist`、`setpwd` 加上 `--admin` 即操作本文件。密码自动以密文格式 `ENC2:` 写入。不加该旗标时，仍然操作 `proxy_users.json`。

```text
XR:> useradd  --admin admin      # 新建管理员（交互输入密码，不回显）
XR:> setpwd   --admin admin      # 修改密码
XR:> userlist --admin            # 列出管理员
XR:> userdel  --admin admin      # 删除
```

- `--admin` 是旗标，不是位置参数。因此名为 `admin` 的代理用户不会和它冲突。短写 `-a` 等价。
- 也可以手工编辑本文件。密码支持明文或密文格式 `ENC2:`（绑定机器的 AES-256-GCM）。明文只适合联调。生产环境请用上面的命令，或用 `DBPrisma passwd` 生成 `ENC2:` 串。
- 写盘是原子替换：先写 `.tmp`，再重命名，旧文件留 `.bak`。同一进程里的管道服务会立刻看到变更。分开的守护进程在下次启动时加载。
- `enabled=true` 但该文件缺失或没有用户时，所有管道登录都会被拒绝。控制通道等于没有启用。

### 协议与白名单

命令与参数的完整列表见命令管道命令参考。这里只保留本配置需要的帧格式。

- 帧格式：4 字节小端长度前缀，加上 UTF-8 JSON。
- 流程：`{"op":"login","user","pass"}` 返回 `{"ok":true,"token":...}`。`{"op":"cmd","token","line":"status"}` 返回 `{"ok":true,"output":...}`。`{"op":"logout","token"}` 结束会话。
- 管道自身的 op（不是 `cmd` / `line`）：`ping`（不必登录）、`whoami`、`metrics`、`capstatus`、`capend`。后四个需要 token。
- `allowed_commands` 之外的子命令一律拒绝。另有一组只属于交互提示符的命令被硬性禁止，不能经管道执行，例如 `clear`、`exit`、`help`、`history`，以及交互式 `capture` 的按键模式。

### 流式语句抓取 capture

经管道执行 `capture` 会另开一条数据通道做流式推送。主通道立即返回 `{"ok":true,"channel":"<管道或套接字名>","token":"<一次性令牌>"}`。失败则返回 `{"ok":false,"error":...}`。

- 客户端连上 `channel` 后，第一行发送 `token\n` 做握手，避免同机其他进程读取明文语句流。服务端回 `{"ok":true}` 后，按行推送 NDJSON。字段与 `capture.json` 一致。
- 过滤参数与交互式相同：`-s ip[:port]` 指定后端，`-c ip[:port]` 指定客户端，`-t DML,DQL,DDL,DCL,TCL|ALL` 指定类型。输出固定为流式文本，没有 `mode` 参数。
- 停止抓取：发送管道自身的 op `{"op":"capend","token":"..."}`。它不经过白名单，也不进入 `XR:>`。
- 队列有上界，由 `capture_queue_mb` 配置，默认 256 MB。尽量不丢。溢出时才丢最旧记录，并用一行 `{"_dropped":N}` 告诉客户端。回调只在 IO 线程里做格式化和入队，不挡住转发。
- 传输缓冲由 `capture_buf_kb` 配置，默认 64 KB，Windows 与 Linux 相同。Windows 设置命名管道收发缓冲。Linux 设置 AF_UNIX 套接字的 `SO_SNDBUF` / `SO_RCVBUF`。
- `capture` 受 `allowed_commands` 约束，白名单里要有 `capture`。`stop` 或进程退出会结束抓取。

### 测试客户端

仓库带有测试客户端，用来演示登录、执行命令、登出，以及 `capture` 的流式输出。

```text
# Windows（命名管道）
tools\cmdpipe_client.ps1 -User admin -Pass secret -Command "status"
tools\cmdpipe_client.ps1 -User admin -Pass secret -Capture "capture -t DML,DQL" -Seconds 30

# 跨平台（Linux AF_UNIX / Windows 命名管道）
tools/cmdpipe_client.py --sock <run_dir>/DBPrisma_ctrl.sock --user admin --pass secret --command status
tools/cmdpipe_client.py --sock <run_dir>/DBPrisma_ctrl.sock --user admin --pass secret \
    --capture "capture -t DML" --seconds 30
```

自行对接时使用两条通道：控制通道的帧，以及 capture 数据通道的握手和 NDJSON。示例目录为 `examples/command-pipe/`。

## license

`license` 是顶层可选段。它只影响到期前提示提前多少天，不改变授权或试用的放行与拒绝。缺省时使用内置默认值，不必显式配置。

```json
"license": {
  "warn_days": 14
}
```

| 键 | 类型 | 默认 | 说明 |
| --- | --- | --- | --- |
| `warn_days` | 整数 | `14` | 到期前提示的提前天数。正式授权或试用的剩余天数小于或等于该值时，启动时以及每小时复检会打印 `[license] ... expires in N day(s)`。`status verbose` 里的 `Days left` 转为黄色。`0` 表示关闭提醒。运行时取值 `0`–`365`，越界会夹取。对正式授权和试用同样生效 |

配合监控：`/metrics` 以及命令管道的 `metrics` op 输出 `dbprisma_license_state`（2 表示授权，1 表示试用，0 表示拒绝，-1 表示未知）和 `dbprisma_license_days_left`（剩余整天数）。

`warn_days` 只控制提示，不改变到期后的行为。正式授权过期后是否还有缓冲，由签发时的 `--grace-days` 决定，默认 0，即过期即拒绝。试用没有宽限。

## Lab

| 路径 | 用途 |
| --- | --- |
| `config/windows/conf_lab.json` | Windows 四后端，加上 `mode` 的 `dispatch` 广播模板 |
| `config/linux/conf_lab.json` | Linux 实验模板（远程后端 192.168.10.34） |
| `config/conf.json` | 仓库根目录的通用模板 |

部署时复制到运行目录，并重命名为 `config/conf.json`。
