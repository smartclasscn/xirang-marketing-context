# DBPrisma Enterprise 部署与 POC 要点
**日期：** 2026年11月

本文说明 DBPrisma Enterprise（息壤数据库串行审计安全网关）的部署与 POC。文中技术句来自开发手册：快速入门、管理员手册的安装与部署，以及配置手册的生产部署场景。

## 安装

### 前提

| 条件 | 说明 |
| --- | --- |
| 后端 | 当前支持 MySQL 和 PostgreSQL，以及兼容这两类数据库的信创产品。准备至少一个可访问实例 |
| 操作系统 | Windows 10+（x64），或 Linux（glibc ≥ 2.17，x64） |
| 账号权限 | 后端账号须有 SELECT、INSERT、UPDATE、DELETE |
| 端口 | 确认监听端口未被占用 |
| 安装物 | POC 使用预编译包。自行构建时才需要 CMake 3.16+ 和 C 编译器 |

### 预编译包目录

预编译包按下列目录放置。`config` 放主配置与代理用户，`run` 在运行时写入 lock、pid 与 cli_token，`logs` 放主日志，`capture` 在运行时写入 `capture.json`。

```
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

Linux 安装根目录可放在 `/opt/dbprisma`。正式授权文件 `license.dat` 与 `conf.json` 放在同一 `config` 目录。

### 首次启动核对

1. 在程序目录执行 `checkconfig`，确认输出含 `Configuration valid`。
2. 初次验证用 `start` 前台启动。后台、systemd 或脚本部署用 `start --no-interactive`。
3. 启动成功后，日志含监听地址，并含 Admin HTTP 地址。
4. 未放置 `license.dat` 时，自首次运行起默认 30 天试用。正式授权把 `license.dat` 放入 `config` 目录后重启，日志中的 `[license]` 行给出授权状态与剩余天数。
5. 若启动后立即退出，先确认 `logs` 目录存在。
6. 在 `XR:>` 下执行 `status`、`conns`、`backends`，分别核对运行状态、当前连接和后端健康。

## 配置

部署时改两个文件。

- `conf.json`：监听、后端、TLS 与日志等主配置。路径在预编译包的 `config/conf.json`。
- `proxy_users.json`：认证托管时，客户端登录网关的账号写在此文件，与后端账号分开。生产环境使用非空账号名单。

生产环境密码使用 `ENC2`。监听端口、监听地址和 `security` 段修改后，须停止进程再启动；这些项在重新启动后生效。

## POC 清单

| 核对项 | 核对什么 |
| --- | --- |
| 进程 | 执行 `status`，确认进程已监听 |
| 后端连通 | 执行 `backends`，确认后端健康；并直连后端地址与端口 |
| 认证托管或透明透传 | 认证托管：核对 `scheme` 为 `auth`，客户端使用 `proxy_users.json` 中的账号。透明透传：核对 `scheme` 为 `transparent`，客户端认证到达后端 |
| IP 访问控制 | 用来源地址核对 `ip_allowlist` 与 `ip_denylist` 的放行和拒绝 |
| TLS 加密（客户端到网关） | 核对客户端到网关的 `tls_termination` |
| 认证与限速 | 用代理用户登录，并核对认证失败封禁 |
| 连接配额 | 核对单 IP 连接数上限 |
| SQL 防火墙 | 用一条命中规则的语句核对拦截或放行 |
| 审计导出 | 打开 `audit_export_enabled`，核对 `audit_export_file` 按 `audit_export_format` 落盘 |
| GET /metrics | 携带管理令牌请求 `GET /metrics`，确认返回 Prometheus 文本 |
