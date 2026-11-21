# DBPrisma Enterprise 安装、配置与运维手册
**日期：** 2026年11月

DBPrisma（息壤数据库串行审计安全网关）部署在现有数据库前面。应用把数据库连接地址指到网关即可接入。当前支持 MySQL 和 PostgreSQL，以及兼容这两类数据库的信创产品。

本手册给安装与运维人员使用。步骤来自开发手册中的快速入门、管理员手册、配置手册和运维手册。配置键的热重载边界以开发配置参考 r19 为准。逐项说明见该参考，本手册不罗列全部配置键。

## 安装

### 前提

| 条件 | 说明 |
| --- | --- |
| 后端 | 当前支持 MySQL 和 PostgreSQL，以及兼容这两类数据库的信创产品。准备至少一个可访问实例 |
| 操作系统 | Windows 10 及以上（x64），或 Linux（glibc 不低于 2.17，x64） |
| 后端账号 | 具备 `SELECT`、`INSERT`、`UPDATE`、`DELETE` |
| 端口 | 客户端要连的监听端口未被占用。快速入门示例使用 `3307` |
| 安装物 | 使用预编译包。本手册不写自行编译 |

### 预编译包目录

预编译包把主程序与四个目录放在同一层。`config` 放主配置和代理用户。`run`、`logs`、`capture` 在运行时写入。

```text
DBPrisma/
├── DBPrisma.exe / DBPrisma
├── PrismaAgent.dll / libPrismaAgent.so
├── config/
│   ├── conf.json
│   └── proxy_users.json
├── run/
├── logs/
└── capture/
```

`run` 存放锁文件、进程号和命令行令牌 cli_token。`logs` 存放主日志。`capture` 存放 `capture.json`。相对路径的日志文件落在 `logs`。四个目录可用 `paths` 段、`--config-dir`、`--run-dir`、`--log-dir`、`--capture-dir` 或环境变量 `DBPRISMA_HOME` 改到其他位置。改 `paths` 后必须重启。

Linux 生产目录可以放在 `/opt/dbprisma`。证书另放 `certs`，目录权限 `700`，私钥文件权限 `640`。配置目录权限 `750`，日志目录权限 `755`。运行账号使用不可登录的专用用户，例如：

```bash
useradd -r -s /sbin/nologin -d /opt/dbprisma dbprisma
chown -R dbprisma:dbprisma /opt/dbprisma
chmod 750 /opt/dbprisma/config
chmod 700 /opt/dbprisma/certs
chmod 755 /opt/dbprisma/logs
chmod 640 /opt/dbprisma/certs/*.key
```

正式授权文件 `license.dat` 与 `conf.json` 放在同一 `config` 目录。

### 首次启动与核对

先核对配置，再启动。`checkconfig` 和 `start` 可以在外壳里直接执行。`checkconfig` 必须作为第一个参数。存在 `[ERROR]` 时退出码为 1，此时不要启动。

```bash
./DBPrisma checkconfig
./DBPrisma start
./DBPrisma start --no-interactive
```

`start` 适合初次在前台验证。后台、systemd 系统服务或脚本使用 `start --no-interactive`。Windows 注册系统服务：

```text
DBPrisma.exe service install
DBPrisma.exe service start
```

Linux 生产可用 systemd。管理员手册中的示例单元把程序写在 `/opt/dbprisma/bin/DBPrisma`，配置写在 `/opt/dbprisma/config/conf.json`，工作目录为 `/opt/dbprisma`。预编译包若把主程序放在安装根目录，把这两处改成实际路径。`KillSignal` 使用终止信号 `SIGTERM`。单元里不要写 `ExecStop` 或 `ExecReload` 去调用 `DBPrisma stop`、`DBPrisma reload`，这两条不是外壳子命令。`systemctl reload` 不会重读配置。

```bash
systemctl daemon-reload
systemctl enable dbprisma
systemctl start dbprisma
systemctl status dbprisma
```

启动后核对下面几项。

1. `checkconfig` 的输出含 `Configuration valid`。
2. 日志中有监听地址，并有管理接口地址。
3. 未放置 `license.dat` 时，自首次运行起默认 30 天试用。试用到期且没有有效授权文件时，进程拒绝启动。放入 `license.dat` 后重启，日志中的 `[license]` 行给出授权状态与剩余天数。申请前可在客户机执行 `licreq --print-fingerprint` 查看机器指纹。看到 `[license] Startup blocked` 时，进程没有进入服务。不要删除 `run/dbprisma.trial.state`，删除会重置试用窗口，且该文件不能拷到另一台机器。
4. 启动后立即退出时，先确认 `logs` 目录存在。
5. 在交互提示符 `XR:>` 下执行 `status`、`conns`、`backends`，分别核对运行状态、当前连接和后端健康。`status` 期望为 `RUNNING`，后端期望为 `UP`。
6. 管理接口已打开时，用 `GET /health` 再核对一次。携带管理令牌的方法见运维一章。

`status`、`stop`、`reload`、`useradd` 等命令只在 `XR:>` 里执行。直接运行 `./DBPrisma status` 或 `./DBPrisma stop` 不会查询状态，也不会停止进程。进入提示符的方式是运行 `./DBPrisma`。守护进程用 `./DBPrisma start --no-interactive` 启动，必须带 `start`。

## 配置

### conf.json 与 proxy_users.json

两个文件分工不同。

| 文件 | 职责 |
| --- | --- |
| `config/conf.json` | 主配置：监听、后端、传输加密、日志、路由和安全策略 |
| `config/proxy_users.json` | 认证托管时，客户端登录网关的账号。与后端账号分开 |

`conf.json` 里每个后端的 `scheme` 必须写成 `auth` 或 `transparent`。省略或写成其他值时，进程启动失败。

认证托管把 `scheme` 设为 `auth`。客户端使用 `proxy_users.json` 里的账号连接网关，网关再用该后端的 `user` 和 `password` 连接数据库。文件须包含 `users` 数组。生产使用非空账号名单。在 `XR:>` 下执行 `useradd` 添加账号，密码交互输入，不回显，写入该文件。`useradd` 和 `setpwd` 之后，代理用户名单立即生效，不必为了这份名单重启，也不必再执行一次 `reload`。

透明透传把 `scheme` 设为 `transparent`。网关不替换客户端认证，客户端使用后端上的真实账号。这种接法不在客户端到网关这一段终结传输加密。加密通道里的语句捕获不可用。监听地址不要暴露到公网，绑在内网，并由主机防火墙限制来源。

`setconfig` 只改磁盘上的 `conf.json`，而且进程运行时拒绝写入。改完后，可热重载的字段在 `XR:>` 执行 `reload`；其余字段先 `stop` 再 `start`。

### 密码用 ENC2

生产环境的后端密码、证书口令和管理令牌使用密文格式 `ENC2:`。算法为带认证的分组加密 AES-256-GCM，并与本机绑定。在 `XR:>` 下执行 `passwd`，把输出的整串填进对应的 `password` 字段。启动时识别这个前缀并解密。

配置里若仍留着旧前缀 `ENC:`，程序把它当成明文使用，认证会失败。用 `passwd` 重新生成 `ENC2:` 再替换。

### 监听与安全策略

客户端连接的是后端项里的 `listen_ip` 与 `listen_port`。启用统一入口时，客户端连接 `mode.listen_ip` 与 `mode.listen_port`，再按 `mode.strategy` 转发。管理接口的地址和端口是 `admin_listen_ip`、`admin_listen_port`。配置参考里 `admin_listen_port` 的默认值是 `0`，表示关闭管理接口。快速入门示例把该端口设为 `8080`，并把 `admin_listen_ip` 设为 `127.0.0.1`。

监听地址、监听端口、`mode.strategy`、后端的增删以及 `scheme` 的变更，都必须重启后才生效。

安全策略写在 `security` 段，整段都必须重启后才生效。`reload` 不会更新已经载入内存的安全策略。按接入路径，策略依次为：

1. IP 访问控制：`ip_allowlist`、`ip_denylist`。条目可以是单个地址或网段（CIDR）。
2. 客户端到网关的传输加密：仅认证托管可用。`tls_termination` 设为 `true`，证书放在 `tls_p1_cert_file` 与 `tls_p1_key_file`，或使用证书包 `tls_p1_pfx_file`。口令可用 `ENC2:`。证书路径相对 `conf.json` 所在目录。
3. 认证与限速：代理用户名单，以及 `auth_fail_max`、`auth_fail_window_ms`、`auth_fail_ban_ms`。
4. 连接配额：`max_connections_per_ip`。`0` 表示不限制。
5. SQL 防火墙：`sql_firewall_enabled`。动作为 `sql_firewall_action`，`reject` 表示拦截，`log_only` 表示只记录并放行。规则在 `sql_block_patterns`。

网关到数据库的传输加密用 `backend_ssl_mode` 和 `tls_p2_ca_file`（或 `tls_p2_ca_path`）。`backend_ssl_mode` 取 `disabled`、`preferred`、`required`、`verify_ca`、`verify_identity`。生产至少使用 `required`，需要核对数据库证书时使用 `verify_identity`。更换客户端到网关或网关到数据库的证书后，先 `checkconfig`，再在 `XR:>` 执行 `restart`。证书路径不支持热重载。

管理接口若启用，设置 `admin_bearer_token`，并让 `admin_listen_ip` 保持在 `127.0.0.1`，或用 `admin_allow_ips` 写明允许访问的精确地址。`admin_allow_ips` 不写 CIDR。来源网段的放行与拒绝用 `security.ip_allowlist` 和 `ip_denylist`，改完必须重启。

生产环境保持 `allow_legacy_password` 为 `false`。语句捕获需要去掉字面值时，把 `capture_redact_literals` 设为 `true`。这两项可以热重载。熔断使用 `circuit_breaker` 段，修改必须重启。进程守护使用 `watchdog` 段，修改必须重启。

多后端同时接收写操作时，把 `mode.strategy` 设为 `dispatch`。至少两个已启用、且 `scheme` 为 `auth` 的后端。客户端收到应答的那个后端用 `primary` 标出，全组至多一个。其余已启用的认证托管后端同时接收写操作。应急时把 `dispatch_primary_only` 设为 `true`，写操作只发给主后端。这个开关可以热重载。`mode.strategy` 本身必须重启。

### 包重组上限

单条 SQL 的包重组上限 `max_reassemble_bytes` 可配置到 1 GiB（1,073,741,824 字节），默认 16 MiB；该项须重启后生效，且后端 `max_allowed_packet` 不得小于该值。

`security.max_packet_size_bytes` 大于 `0` 时，作为更严的单条命令上限，同样必须重启。

逐项说明见开发配置参考 r19。

## 运维

### 日常检查

每天查看：

1. 在 `XR:>` 执行 `status`，确认进程处于运行状态。
2. 执行 `backends`，确认后端健康。
3. 执行 `conns`，看活跃连接是否异常。
4. 在 `logs` 中查看 `ERROR` 与 `WARN`。
5. 请求 `GET /health`，确认管理接口可达。
6. 查看日志所在磁盘的剩余空间。

每周查看日志是否按大小和份数轮转，清理过期的 `conf.json.bak.*`，看证书是否临近到期。到期前更换证书并重启。每月在 `XR:>` 执行 `token rotate` 轮换管理令牌，执行 `userlist` 核对代理用户，并复查 IP 名单。`token rotate` 立即生效，不必重启。

### 启动与停止

外壳可以直接执行的是 `start`、`checkconfig`、`setconfig`、`service`。停止、热重载、查状态都在 `XR:>`。

| 动作 | 做法 |
| --- | --- |
| 前台启动 | `./DBPrisma`，进入 `XR:>` 后再 `start`；或直接 `./DBPrisma start` |
| 后台启动 | `./DBPrisma start --no-interactive --config config/conf.json` |
| systemd | `systemctl start dbprisma`。停止用 `systemctl stop dbprisma`，信号为 `SIGTERM` |
| Windows 服务 | `DBPrisma.exe service start` 与 `DBPrisma.exe service stop` |
| 优雅停止 | 在 `XR:>` 执行 `stop`。进程先停止接受新连接，再等待现有事务结束 |
| 带倒计时停止 | `shutdown`，或 `shutdown 30`。`shutdown 0 force` 跳过等待，只在紧急时使用 |
| 重启进程 | 在 `XR:>` 执行 `restart` |

优雅停止的等待时间由 `graceful_drain_ms` 和 `txn_drain_ms` 控制。计划维护某一后端时，在 `XR:>` 执行 `drain` 加该后端的 `server_id` 做排空（优雅下线）：停止把新连接送到它，已有连接自然结束。恢复用 `drain cancel` 加同一编号。`pause` 停止接受新的客户端连接且不断开已有连接，`resume` 恢复接受。

### 可以热重载的项，以及必须重启的项

已运行时，在 `XR:>` 执行 `reload`。程序只应用可以热重载的字段。碰到必须重启的字段时保持旧值，日志中可见需要重启。整次重载若失败，则内存中的配置保持不变。`config_manage.backup_on_reload` 为 `true` 时，重载前在 `config` 目录生成 `conf.json.bak.YYYYMMDD_HHMMSS`。

可以热重载的主要包括：

- `logging.level`、`logging.max_size_mb`、`logging.rotate_count`
- 已经开启的空闲超时阈值 `proxy.idle_timeout_sec`，以及 `proxy.connect_timeout_ms`
- `proxy.capture_redact_literals`、`proxy.allow_legacy_password`
- `mode.read_strategy`、`read_after_write_sticky_ms`、`read_fallback_to_primary`、`read_health_skip_failures`，以及 `mode.dispatch_primary_only`
- `admin_bearer_token`、`admin_allow_ips`
- `resilience` 中除 `backend_tcp_keepalive` 以外的字段
- `proxy.startup` 与 `proxy.shutdown`。关闭相关的值在下一次停止时使用
- 慢查询日志路径、防火墙日志路径，以及 `audit_export_enabled`、`audit_export_format`、`audit_export_file`、`audit_chain_enabled`、`audit_chain_key`
- 在 `XR:>` 执行 `loglevel` 可立即改变日志级别

启动时 `idle_timeout_sec` 为 `0` 的，热重载改成非零后不会开始计时，需要重启才启用空闲超时。

必须重启的主要包括：

- 监听地址和监听端口，含各后端监听、`mode.listen_ip`、`mode.listen_port`、`admin_listen_ip`、`admin_listen_port`
- 安全策略整段 `security`
- 包重组上限 `max_reassemble_bytes`
- 客户端到网关、网关到数据库的证书与 `backend_ssl_mode`，以及 `admin_tls_*`
- `servers` 的增删、`id`、`enabled`、`scheme`、账号和密码
- `mode.strategy`
- `circuit_breaker`、`watchdog`、`capture`、`tls`、`paths`、`notify`、`command_pipe`
- 主日志路径 `logging.file`

改安全策略的顺序是：`XR:>` 执行 `stop`，用 `setconfig security` 或直接编辑 `conf.json`，再 `checkconfig`，然后 `start`。

审计导出面向安全信息与事件管理（SIEM）。打开 `audit_export_enabled`，用 `audit_export_format` 选择 `json`、`cef` 或 `syslog`，用 `audit_export_file` 指定文件。需要哈希链时打开 `audit_chain_enabled`。这些导出字段可以热重载。不启动进程时，可用外壳命令 `checkaudit` 校验导出文件中的哈希链。

### 故障时先看什么

先做四件事，再按现象分支。

1. `XR:>` 执行 `status`，确认进程是否在运行。
2. `XR:>` 执行 `backends`，确认后端是否可达，以及熔断是否已经打开。
3. 在 `logs` 里查看最新的 `ERROR`。
4. 请求 `GET /health`，确认管理接口是否正常。

| 现象 | 先核对 |
| --- | --- |
| 进程未运行 | 日志里的 `[license]` 与启动错误；`logs` 目录是否存在；端口是否被占用。然后按启动步骤再起 |
| 客户端连接被拒绝 | 进程是否在听目标端口。Linux 可用 `ss` 或 `netstat` 查看监听 |
| 认证失败 | `XR:>` 执行 `userlist`。没有该用户则 `useradd`。密码错误则 `setpwd`。再看该来源地址是否处于认证失败封禁，封禁时长为 `auth_fail_ban_ms` |
| 后端不可用 | 在网关所在主机直连后端地址和端口。查看 `backends` 与主日志。熔断打开后，等待 `open_duration_ms` 再探测 |
| 连接数过高 | `conns` 查看空闲连接，必要时按连接断开。`max_clients` 的修改必须重启 |
| SQL 被防火墙拦截 | 看主日志或 `firewall_log_file` 中的命中记录。规则在 `security`，修改必须重启。临时只记录不拦截时，把 `sql_firewall_action` 设为 `log_only` 后重启 |
| 配置改坏 | 用同目录的 `conf.json.bak.*` 覆盖回去。可以热重载的字段执行 `reload`；监听、安全策略、包重组上限仍须重启 |

### 管理接口

管理接口只读，只接受 `GET`。控制类操作仍走 `XR:>`，不走这个接口。同一端口同时提供 HTTP 与 HTTPS：配置了 `admin_tls_pfx_file`，或同时配置了 `admin_tls_cert_file` 与 `admin_tls_key_file` 时，明文与加密都可以接入；未配置证书时只有 HTTP。证书已配置但加载失败时，进程启动失败。`admin_tls_*` 的修改必须重启。口令可用 `ENC2:`。

`admin_bearer_token` 非空时，每个端点都要带令牌。请求头使用 `Authorization: Bearer`，或使用 `X-Admin-Token`。仅监听本机且令牌为空时可以不带令牌。绑定 `0.0.0.0` 时必须设置令牌。

| 端点 | 用途 |
| --- | --- |
| `GET /health` | 运行状态、已运行时间、各后端健康与连接数 |
| `GET /metrics` | Prometheus 文本。监控系统抓取这一路径 |
| `GET /events` | 最近的通知事件 |
| `GET /audit/search` | 在审计导出中按关键词检索 |

其他路径返回 404。非 `GET` 返回 405。HTTP 与 HTTPS 的端点、内容和鉴权相同，差别只在传输是否加密。

```bash
curl -H "Authorization: Bearer <admin_bearer_token>" http://127.0.0.1:8080/health
curl -k -H "Authorization: Bearer <admin_bearer_token>" https://127.0.0.1:8080/metrics
```

上面的 `8080` 是快速入门示例。实际端口以这台机器的 `admin_listen_port` 为准。
