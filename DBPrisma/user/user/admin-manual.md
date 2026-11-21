# DBPrisma 管理员手册

**版本**: v1.1 | **日期**: 2026-06-10 | **适用版本**: DBPrisma ≥ 1.0  
**受众**: 系统管理员、DBA、平台工程师

---

## 目录

1. [系统架构概览](#1-系统架构概览)
2. [安装与部署](#2-安装与部署)
3. [配置文件管理](#3-配置文件管理)
4. [代理用户管理](#4-代理用户管理)
5. [密码加密管理](#5-密码加密管理)
6. [TLS 证书管理](#6-tls-证书管理)
7. [Windows 服务管理](#7-windows-服务管理)
8. [Linux 守护进程管理](#8-linux-守护进程管理)
9. [完整 CLI 命令参考](#9-完整-cli-命令参考)
10. [Admin HTTP API](#10-admin-http-api)
11. [升级与回滚](#11-升级与回滚)
12. [多进程与集群部署](#12-多进程与集群部署)

---

## 1. 系统架构概览

```
客户端应用
    │  MySQL 协议（TCP/TLS）
    ▼
┌─────────────────────────────────────────┐
│              DBPrisma 代理              │
│  ┌──────────┐  ┌────────┐  ┌────────┐  │
│  │ 连接池   │  │ SQL    │  │ 路由   │  │
│  │ 管理     │  │ 防火墙 │  │ 引擎   │  │
│  └──────────┘  └────────┘  └────────┘  │
│  ┌──────────┐  ┌────────┐  ┌────────┐  │
│  │ 熔断器   │  │ 看门狗 │  │ 认证   │  │
│  └──────────┘  └────────┘  └────────┘  │
│         Admin HTTP :8080               │
└─────────────────────────────────────────┘
    │  MySQL 协议（TCP/TLS）
    ▼
后端 MySQL / MariaDB 集群
```

**核心组件**：
- **连接代理层**：多路复用客户端连接到后端连接池
- **路由引擎**：读写分离、主从分发、负载均衡
- **熔断器（Circuit Breaker）**：后端故障自动隔离与恢复
- **看门狗（Watchdog）**：进程心跳检测与自动重启
- **Admin HTTP/HTTPS**：实时状态查询、指标暴露、事件查询（同端口双协议，可选 TLS）

---

## 2. 安装与部署

### 2.1 目录规划（推荐）

```
/opt/dbprisma/                  （Linux 安装根目录）
├── bin/
│   └── DBPrisma                # 主程序
├── lib/
│   └── libDBPrisma.so          # 共享库
├── conf/
│   ├── conf.json               # 主配置
│   ├── proxy_users.json        # 代理用户
│   └── admin_users.json        # 命令管道管理员（启用 command_pipe 时）
├── certs/                      # TLS 证书目录（权限 700）
│   ├── server.crt
│   ├── server.key
│   └── ca.crt
├── logs/                       # 日志目录（权限 755）
└── backups/                    # 配置备份（自动生成）

C:\DBPrisma\                    （Windows 安装根目录）
├── DBPrisma.exe
├── DBPrisma.dll
├── conf\conf.json
├── conf\proxy_users.json
├── conf\admin_users.json
├── certs\
└── logs\
```

### 2.2 权限配置（Linux）

```bash
# 创建专用运行账号（不可登录）
useradd -r -s /sbin/nologin -d /opt/dbprisma dbprisma

# 设置目录权限
chown -R dbprisma:dbprisma /opt/dbprisma
chmod 750 /opt/dbprisma/config
chmod 700 /opt/dbprisma/certs
chmod 755 /opt/dbprisma/logs

# 证书文件权限
chmod 640 /opt/dbprisma/certs/*.key
```

### 2.3 构建参数

| 构建类型 | CMake 参数 | 用途 |
|----------|-----------|------|
| Release | `-DCMAKE_BUILD_TYPE=Release` | 生产部署 |
| Debug | `-DCMAKE_BUILD_TYPE=Debug` | 开发调试 |
| ASan | `-DENABLE_ASAN=ON` | 内存错误检测 |
| Fuzz | `-DENABLE_FUZZ=ON` | 模糊测试 |

```bash
# 生产 Release 构建
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/dbprisma
make -j$(nproc)
make install
```

#### 2.3.1 脚本化构建（推荐）

日常与发布默认用 **`buildall.bat`**（x64 全量：Windows 本地 + Linux host 37）。产物布局见 [bin/README.md](../../bin/README.md)。

**四平台 Release 按需全编**（ARM64 不在 `buildall` 默认矩阵内，需手动执行）：

```bat
REM 在仓库根目录
scripts\build\Windows\x64\build.bat
scripts\build\Windows\arm64\build.bat
python scripts\build\Linux\build_on_37.py --script scripts/build/Linux/x64/build.sh
python scripts\build\Linux\build_on_37.py --script scripts/build/Linux/arm64/build.sh
```

| 平台 | 产物目录 | 发布 |
|------|----------|------|
| Windows x64 | `bin/Windows/x64/` | 直接部署 |
| Windows arm64 | `bin/Windows/arm64/` | 部署到 ARM64 Windows（WoA）；本机 x64 无法运行 arm64 单测 |
| Linux x64 | `bin/Linux/x64/` | `python scripts/dev/_deploy_release_37.py` → `/opt/dbprisma` |
| Linux arm64 | `bin/Linux/arm64/` + `DBPrisma-linux-arm64.tar.gz` | 解压到 **ARM64 Linux 设备**（lab host 37 为 x86_64） |

授权工具（签发机，Windows x64）：`tools\licissue\build_windows.bat`、`tools\licreq\build_windows.bat`。

构建细节：[scripts/build/README.md](../../scripts/build/README.md)。

### 2.4 授权（License）部署

DBPrisma 启动受离线授权闸门控制，**无 `license.dat` 时默认 30 天试用**，到期或放入无效文件将拒绝启动。

```bash
# 0) （可选）先查看本机机器指纹，便于售后核对或与 licissue show 对比
licreq --print-fingerprint              # 默认：单行 64 位 hex
licreq --print-fingerprint --all        # hex + OpenSSH randomart 图案

# 1) 客户机上生成授权申请（含本机机器指纹与订单 SN），发给厂商
licreq                     # 交互式录入组织/申请人/订单 SN 等，产出 license.request（含请求摘要）
licreq --show license.request   # 生成后可随时回显核对

# 2) 厂商 licissue show 审核 → licissue issue --private-key ... --public-key ... 签发
#    将回传的 license.dat 放入 config 目录（与 conf.json 同级）
cp license.dat /opt/dbprisma/config/license.dat

# 3) 重启服务使授权生效
systemctl restart dbprisma     # 或 XR:> restart
```

启动后控制台/日志的 `[license] ...` 行表明授权状态与剩余天数。判定规则、文件位置（`config/license.dat`、`run/dbprisma.trial.state`）与启动失败排障详见 [运维手册 §13](operations-manual.md#13-授权与-license-管理)；工具用法见 [licreq](../../tools/licreq/README.md) / [licissue](../../tools/licissue/README.md)。

> **注意**：勿删 `run/dbprisma.trial.state`（删除会重置试用窗口）；该文件机器绑定封存，无法跨机复制。更换网卡/主机名/重装可能改变机器指纹，需重新申请换发 license。

---

## 3. 配置文件管理

### 3.1 配置文件清单

| 文件 | 用途 | 热更支持 |
|------|------|---------|
| `conf.json` | 主配置文件 | 部分字段支持（见热更矩阵） |
| `proxy_users.json` | 代理用户列表 | 是（`reload` 命令） |
| `admin_users.json` | 命令管道管理员凭据（`command_pipe.auth_store`） | 否（重启生效） |
| `license.dat` | 厂商签发的授权文件（缺失则 30 天试用） | 否（重启生效；见 §2.4） |

> 配置字段完整说明见 [配置参考手册](configuration-reference.md)（共 162 个配置项）

### 3.2 配置查看与在线动态调整

`setconfig` 采用**段式（section-based）语法**，用于编辑磁盘上的 `conf.json`，**服务运行时禁止写入**。  
实时运行时调整请使用交互命令（`loglevel`）或修改配置后执行 `reload`。

```bash
# 查看配置（只读，服务运行时也可使用）
./DBPrisma setconfig show logging        # 查看日志段
./DBPrisma setconfig show proxy          # 查看代理段
./DBPrisma setconfig show server 1       # 查看指定后端
./DBPrisma setconfig show               # 查看全部配置

# 编辑配置段（交互式向导，服务必须先停止）
./DBPrisma setconfig logging             # 修改日志配置（级别、文件、轮转）
./DBPrisma setconfig proxy               # 修改代理核心参数
./DBPrisma setconfig security            # 修改安全策略（改后须重启生效）

```

`loglevel` / `reload` 是 **XR:> 提示符下的运行时命令**（先运行 `./DBPrisma` 进入交互模式），不是独立的 Shell 子命令：

```text
# 运行时动态修改日志级别（无需停服，立即生效）
XR:> loglevel DEBUG                      # 切换为 DEBUG 级别
XR:> loglevel INFO                       # 恢复 INFO 级别

# 热更可热加载字段：先手动编辑 conf.json，再执行：
XR:> reload
```

### 3.3 热更整个配置文件

```text
# 修改 conf.json 后，在 XR:> 提示符下热更（不重启）
XR:> reload

# 热更结果：
#   成功：Reload applied. See DBPrisma.log for the change summary.
#   失败：Reload rejected by PA_Reload. ... Structural changes require 'restart'.
```

**热更矩阵**（主要字段；完整列表见 [configuration-reference.md](configuration-reference.md) §九）：

| 配置段 | 热更支持 | 说明 |
|--------|---------|------|
| `logging.level` / `max_size_mb` / `rotate_count` | ✅ | 日志级别与轮转 |
| `logging.file` | ❌ | 主日志路径须 restart |
| `security.*` | ❌ | 防火墙、限速、黑白名单等须 restart |
| `paths.*` | ❌ | 四目录布局须 restart |
| `proxy.idle_timeout_sec` / `connect_timeout_ms` | ✅ | 空闲/连接超时（idle 见 §九注） |
| `proxy.capture_redact_literals` / `allow_legacy_password` | ✅ | |
| `proxy.max_clients` / `worker_threads` | ❌ | 须 restart |
| `proxy.admin_bearer_token` / `admin_allow_ips` | ✅ | Admin 访问控制可热更 |
| `proxy.admin_listen_*` / `admin_tls_*` | ❌ | 监听与证书须 restart |
| `circuit_breaker.*` | ❌ | 须 restart |
| `servers[]`（含 `enabled`/`weight`/端口/凭据） | ❌ | 后端结构变更须 restart |
| `log_config.slow_query_*` / `firewall_log_*` | ✅ | 慢查询与防火墙审计日志可热更 |
| `log_config.log_format` / `log_include_sql` | ❌ | 须 restart |
| `resilience.*`（除 `backend_tcp_keepalive`） | ✅ | |
| `proxy.startup.*` / `proxy.shutdown.*` | ✅ | shutdown 在下次 stop 时生效 |
| `mode.read_*` / `dispatch_primary_only` | ✅ | read_write 子项可热更 |
| `tls.*` / `watchdog.*` / `capture.*` / `notify.*` | ❌ | 须 restart |

### 3.4 配置备份与回滚

当 `config_manage.backup_on_reload = true` 时，每次 `reload` 前自动备份 `conf.json` 到 **config 目录同级**（与 `conf.json` 同目录）：

```
config/
├── conf.json
├── conf.json.bak.20260608_093000
├── conf.json.bak.20260607_140000
└── ...（最多 config_manage.max_backup_count 份，按时间戳命名）
```

备份命名格式：`conf.json.bak.YYYYMMDD_HHMMSS`（`setconfig` 写盘使用相同 `.bak` 机制）

若新配置导致异常，可手动回滚：
```bash
cp config/conf.json.bak.20260607_140000 config/conf.json
# 在 XR:> 会话中执行：
# XR:> reload
```

或设置 `config_manage.rollback_on_reload_fail = true` 自动回滚。

---

## 4. 代理用户管理

DBPrisma 维护独立于 MySQL 后端的代理用户数据库，存储于 `proxy_users.json`。

### 4.1 用户操作命令

用户管理命令均为 **XR:> 提示符下的运行时命令**（先运行 `./DBPrisma` 进入交互模式）：

```text
# 添加用户（交互式密码提示，不回显，密码自动加密存储）
XR:> useradd <username>

# 删除用户
XR:> userdel <username>

# 列出所有用户
XR:> userlist

# 修改用户密码（交互式密码提示，不回显）
XR:> setpwd <username>
```

> **管理命令管道管理员**：上述 4 个命令加 `--admin` 旗标即操作 `admin_users.json`（命令管道登录账号，
> 见 §4.4），不加则操作 `proxy_users.json`。用旗标而非位置关键字，避免与名为 `admin` 的代理用户歧义。
>
> ```text
> XR:> useradd  --admin <username>     # 新建管道管理员
> XR:> setpwd   --admin <username>     # 改密码
> XR:> userlist --admin                # 列出管道管理员
> XR:> userdel  --admin <username>     # 删除
> ```

### 4.2 用户连接时的身份验证流程

```
客户端 --> [DBPrisma 代理认证] --> [连接后端 MySQL 使用配置中的账号]
```

- 客户端使用**代理用户**（`proxy_users.json` 中的账号）连接 DBPrisma
- DBPrisma 内部使用 `servers[].user` / `servers[].password` 连接后端 MySQL
- 两套账号完全独立，实现账号隔离和访问控制

### 4.3 用户认证安全策略

通过 `security` 段配置认证保护：

```json
"security": {
  "auth_fail_max": 5,
  "auth_fail_window_ms": 60000,
  "auth_fail_ban_ms": 300000
}
```

- 60 秒内失败 5 次，该 IP 封禁 5 分钟
- 封禁期间拒绝所有来自该 IP 的连接

### 4.4 命令管道远程管控（`command_pipe`）

带外管理控制通道：运维客户端经本地 IPC（Windows 命名管道 / Linux AF_UNIX 套接字）**登录并执行管理子命令**，
无需进入 `XR:>`。仅守护/服务模式生效（systemd / SCM / `--no-interactive`），与 `XR:>` 完全隔离。

```bash
# 1) 启用并按需编辑（改后须 restart）
./DBPrisma setconfig command_pipe        # enabled/pipe_name/local_only/auth_store/
                                         # capture_queue_mb/capture_buf_kb/allowed_commands

# 2) 准备管理员凭据 admin_users.json（独立于 proxy_users.json）
#    推荐用命令（见 §4.1，密码自动 ENC2: 加密落盘）：
./DBPrisma                               # 进入 XR:> 后：useradd --admin admin
#    或手工编辑 config/admin_users.json：
#    { "users": [ { "username": "admin", "password": "ENC2:..." } ] }  # ENC2 由 'passwd' 生成

# 3) 客户端连接（仓库自带测试客户端）
tools\cmdpipe_client.ps1 -User admin -Pass secret -Command "status"
tools/cmdpipe_client.py --sock <run_dir>/DBPrisma_ctrl.sock --user admin --pass secret --command status
```

- 子命令受 `allowed_commands` 白名单门控；完整命令与参数见 [命令管道命令参考手册](command-pipe-command-reference.md)。
- `checkconfig` 会校验 `local_only`、`auth_store` 是否存在/有用户。
- 支持 `capture` **流式 SQL 抓取**（白名单动词）：`capture` 另开数据通道并返回一次性 token，客户端握手后接收 NDJSON 流；
  停止抓取用 **§1A 管道原生 op** `capend`（不进 `XR:>`、不经白名单）。
- 抓取缓冲可调（Windows/Linux 通用）：`capture_queue_mb`（FIFO 内存上限，默认 256 MB，溢出丢最旧并 `{"_dropped":N}` 告知）、
  `capture_buf_kb`（数据通道传输缓冲，默认 64 KB；Win=命名管道收发缓冲，Linux=套接字 `SO_SNDBUF/SO_RCVBUF`）。
- 完整说明（协议帧、字段表、`-s/-c/-t`、背压、安全）见 [配置参考手册 §二十三](configuration-reference.md)；
  C#/Java/Python 客户端开发见 [命令管道客户端开发手册](command-pipe-client-guide.md)。

---

## 5. 密码加密管理

DBPrisma 支持加密存储密码（ENC2 格式），避免明文密码出现在配置文件中。

### 5.1 加密密码

`passwd` 是 XR:> 提示符下的运行时命令（先运行 `./DBPrisma` 进入交互模式）：

```text
# 交互式加密（推荐，不在 shell 历史中留下明文）
XR:> passwd

# 示例输出
Enter plain password: ******
Encrypted: ENC2:aGVsbG8gd29ybGQ=...

# 将输出的 ENC2:... 字符串填入 conf.json 对应的 password 字段
```

### 5.2 在配置文件中使用加密密码

```json
"servers": [
  {
    "user": "dbprisma",
    "password": "ENC2:aGVsbG8gd29ybGQ=..."
  }
]
```

DBPrisma 启动时自动识别 `ENC2:` 前缀并解密。

### 5.3 Token 轮换

Admin HTTP 令牌定期轮换：

```text
# 在 XR:> 提示符下生成新 token 并热更生效（无需重启）
XR:> token rotate
```

---

## 6. TLS 证书管理

DBPrisma 支持两段 TLS：
- **P1（客户端侧）**：客户端 → DBPrisma 的 TLS 加密
- **P2（后端侧）**：DBPrisma → MySQL 后端的 TLS 加密

### 6.1 P1 TLS 配置

**Linux（PEM 格式）**：
```json
"servers": [
  {
    "tls_termination": true,
    "tls_p1_cert_file": "/opt/dbprisma/certs/server.crt",
    "tls_p1_key_file": "/opt/dbprisma/certs/server.key",
    "tls_p1_cert_cn": "dbprisma.example.com"
  }
]
```

**Windows（PFX 格式）**：
```json
"servers": [
  {
    "tls_termination": true,
    "tls_p1_pfx_file": "C:\\DBPrisma\\certs\\server.pfx",
    "tls_p1_pfx_password": "ENC2:..."
  }
]
```

### 6.2 P2 TLS 配置

```json
"servers": [
  {
    "backend_ssl_mode": "verify_ca",
    "tls_p2_ca_file": "/opt/dbprisma/certs/mysql-ca.crt"
  }
]
```

`backend_ssl_mode` 取值（均为小写）：
- `disabled`：不使用 TLS（默认）
- `preferred`：优先 TLS，后端不支持则降级明文
- `required`：强制 TLS，失败则拒绝连接
- `verify_ca`：验证后端证书 CA
- `verify_identity`：验证 CA + 主机名（完整验证）

### 6.3 全局 TLS 参数

```json
"tls": {
  "min_tls_version": "TLSv1.2",
  "cipher_list": "ECDHE-RSA-AES256-GCM-SHA384:...",
  "prefer_server_ciphers": true,
  "session_ticket_enabled": false
}
```

### 6.4 证书轮换

TLS 证书不支持热更，需重启代理：

```bash
# 1. 替换证书文件
cp new-server.crt /opt/dbprisma/certs/server.crt
cp new-server.key /opt/dbprisma/certs/server.key

# 2. 验证配置（checkconfig 为 Shell 子命令）
./DBPrisma checkconfig
```

```text
# 3. 在 XR:> 提示符下优雅重启（等待现有连接处理完）
XR:> restart
```

---

## 7. Windows 服务管理

```powershell
# 安装为 Windows 系统服务
.\DBPrisma.exe service install

# 启动服务
.\DBPrisma.exe service start
# 或
Start-Service DBPrisma

# 停止服务
.\DBPrisma.exe service stop
# 或
Stop-Service DBPrisma

# 卸载服务
.\DBPrisma.exe service uninstall

# 查看服务状态
Get-Service DBPrisma
```

**服务配置建议**：
- 设置服务账号为最低权限账号（非 SYSTEM/Administrator）
- 启用"故障后自动重启"策略
- 配置 `watchdog.enabled = true` 配合系统服务恢复策略

---

## 8. Linux 守护进程管理

### 8.1 systemd 服务文件

创建 `/etc/systemd/system/dbprisma.service`：

```ini
[Unit]
Description=DBPrisma MySQL Proxy
After=network.target
Wants=network.target

[Service]
Type=simple
User=dbprisma
Group=dbprisma
WorkingDirectory=/opt/dbprisma
ExecStart=/opt/dbprisma/bin/DBPrisma start --no-interactive --config /opt/dbprisma/config/conf.json
KillSignal=SIGTERM
Restart=on-failure
RestartSec=5s
LimitNOFILE=65535
LimitNPROC=4096

[Install]
WantedBy=multi-user.target
```

> **注意**：
> - 自动生成的 systemd 单元不包含 `ExecStop=` 和 `ExecReload=`，进程退出依赖 `KillSignal=SIGTERM`。
> - `systemctl reload dbprisma` 对 DBPrisma **无效**（代理不响应 SIGHUP 作为重新加载信号）。热更配置请在 XR:> 会话中执行 `reload` 命令。
> - `DBPrisma stop` 和 `DBPrisma reload` **不是**有效的独立 Shell 命令，不能在 `ExecStop=` / `ExecReload=` 中使用。

```bash
# 启用并启动
systemctl daemon-reload
systemctl enable dbprisma
systemctl start dbprisma

# 查看状态
systemctl status dbprisma
journalctl -u dbprisma -f
```

### 8.2 文件描述符限制

高并发场景下需增大 fd 限制：

```bash
# /etc/security/limits.d/dbprisma.conf
dbprisma soft nofile 65535
dbprisma hard nofile 65535
```

---

## 9. 完整 CLI 命令参考

### 9.0 命令行启动参数

```
Usage: DBPrisma [OPTIONS] [SUBCOMMAND [args...]]
```

| 参数 | 说明 |
|------|------|
| `-c, --config <file>` | 配置文件路径（默认：`<exe>/config/conf.json`） |
| `--config-dir <dir>` | `conf.json`/`proxy_users.json`/备份目录（默认 `<exe>/config/`） |
| `--run-dir <dir>` | lock/pid/cli_token/cli_history/machine.key 目录（默认 `<exe>/run/`） |
| `--log-dir <dir>` | 主/防火墙/慢查询日志目录（默认 `<exe>/logs/`） |
| `--capture-dir <dir>` | `capture.json` 目录（默认 `<exe>/capture/`） |
| `--token <hex>` | 非交互式特权命令 CLI 认证令牌 |
| `--bg-capture` | 启动时开始后台 SQL 捕获到 `capture/capture.json` |
| `--no-interactive` | 守护进程模式：跳过 XR:> 提示符；Ctrl+C / stop 时退出 |
| `--single-instance` | 如果另一个 DBPrisma 实例正在运行则拒绝启动 |
| `--log-level <debug\|info\|warn\|error>` | 覆盖配置文件中的日志级别 |
| `--pid-file <path>` | 服务成功启动后将 PID 写入文件 |
| `--print-config` | 加载 conf.json，打印生效配置，然后退出 |
| `--warmup-only` | 运行预检 + 预热，然后退出（CI/就绪检查） |
| `-v, --version` | 打印版本信息并退出 |
| `-h, --help` | 打印此帮助并退出 |

> **参数语法**：`checkconfig`/`cc`、`setconfig`/`sc`、`service` 这三类子命令必须作为**第一个参数**（位于所有选项之前），否则报错退出（`subcommand '<x>' must come before options`）。`start` 子命令则不受此限——选项可置于 `start` 之前或之后（如 `DBPrisma start --no-interactive --config X` 与 `DBPrisma --no-interactive start` 均可）。带值参数同时接受 `--opt value` 与 `--opt=value` 两种写法；`--` 表示选项结束（其后均按位置参数处理）。无法识别的选项会**报错并退出**（给出最接近的拼写建议），拼错的子命令同样报错。

**子命令**（不进入 XR:> 提示符直接运行）：

| 子命令 | 说明 |
|------|------|
| `start [config_file]` | 自动启动代理服务并进入 XR:> 提示符 |
| `checkconfig \| cc [config_file]` | 验证配置语法和语义，然后退出 |
| `setconfig \| sc <verb> [args]` | 修改磁盘上的配置，然后退出（需要 CLI 认证） |
| `service <verb> [--config <f>]` | 管理 OS 服务（Windows SCM / Linux systemd） |

**环境变量**：

| 变量 | 说明 |
|------|------|
| `DBPRISMA_CLI_TOKEN` | 非交互式认证的替代 `--token` |
| `DBPRISMA_HOME` | 设置后作为默认根目录解析四目录布局（`$DBPRISMA_HOME/config\|run\|log\|capture`），优先级低于 `--config-dir` 与 `-c` 文件所在目录、高于 `<exe>` 同级目录 |
| `NO_COLOR` | 设置后（任意值）禁用彩色输出 |

### 9.1 进程生命周期

| 命令 | 说明 |
|------|------|
| `start [config]` | 启动代理服务（默认：conf.json） |
| `stop` | 优雅停止（等待连接断开） |
| `restart [config]` | 优雅重启（stop + 预检 + 预热启动） |
| `reload [file]` | 热更 conf.json（日志/空闲/弹性/admin/生命周期字段） |
| `shutdown [s] [force\|cancel]` | 停止服务并退出（s=倒计时秒数；force=立即；cancel=取消） |
| `checkconfig \| cc [file]` | 验证配置（语法 + 语义），然后退出 |
| `status [verbose]` | 显示运行时统计；`verbose` 增加 QPS 和计数器 |
| `version` | 打印版本、平台和构建信息 |

### 9.2 配置管理

| 命令 | 说明 |
|------|------|
| `setconfig \| sc show [section\|all]` | 查看配置（section：version/proxy/admin/resilience 等） |
| `setconfig \| sc <verb> [args...]` | 修改指定配置节（verb 为段名，如 `proxy`/`mode`/`logging`/`security`/`tls`/`notify` 等，共 22 个） |
| `setconfig \| sc server <id>\|add\|del` | 管理后端服务器配置（增删查） |
| `setconfig \| sc srvuser <id> <user>` | 设置指定后端服务器的登录用户名 |
| `setconfig \| sc srvpwd <id>` | 交互式修改指定后端服务器的登录密码（无回显） |
| `setconfig \| sc use <path>` | 切换活跃配置文件 |
| `setconfig \| sc -f <path>` | 单次命令覆盖文件路径 |
| `display` | 从 conf.json 显示配置 + 代理用户列表（读磁盘） |

### 9.3 用户管理

| 命令 | 说明 |
|------|------|
| `useradd [--admin] <username>` | 创建用户（交互式密码提示）；加 `--admin` 操作命令管道管理员库 `admin_users.json` |
| `userdel [--admin] <username>` | 删除用户（加 `--admin` 操作管理员库） |
| `userlist [--admin]` | 列出用户（加 `--admin` 列出命令管道管理员） |
| `setpwd [--admin] <username>` | 修改用户密码（交互式密码提示；加 `--admin` 操作管理员库） |
| `passwd` | 交互式加密后端密码 → ENC2:...（AES-256-GCM）；可选 `[明文]` 参数已弃用（会告警，明文会进 shell 历史/进程列表），推荐无参交互输入 |

> **注意**：`useradd` 和 `setpwd` 均使用交互式密码提示（不回显），不接受命令行密码参数。

### 9.4 连接管理

| 命令 | 说明 |
|------|------|
| `conns [ip]` | 列出活跃连接（含字节/查询/时间详情）；可选按客户端 IP 过滤 |
| `kick <ip> [port]` | 按 IP 强制断开客户端；port 0 = 所有端口 |
| `kick id <n>` | 断开 `conns` 列表中第 N 行连接（N 取自最近一次 `conns` 输出） |
| `drain <server_id>` | 标记后端为排水（不接受新连接） |
| `drain cancel <server_id>` | 取消排水 — 后端恢复接受新连接 |
| `pause` | 暂停接受所有新客户端连接 |
| `resume` | 暂停后恢复接受新连接 |
| `flush [server_id]` | 刷新某个后端的预热池；省略 server_id 则刷新全部 |

### 9.5 监控与诊断

| 命令 | 说明 |
|------|------|
| `status [verbose]` | 代理整体运行状态摘要；`verbose` 增加 QPS 和计数器 |
| `backends` | 显示后端健康摘要（状态、排水、探针） |
| `loglevel <level>` | 无需重启修改日志详细程度（debug\|info\|warn\|error） |
| `capture \| cap [opts] [mode]` | 实时 SQL 捕获（此窗口阻塞；P=暂停/恢复 E=退出） |
| | 选项：`-s IP:PORT` 按后端过滤；`-c IP:PORT` 按客户端过滤 |
| | 类型：`-t DDL,DML,DQL,DCL,TCL`；模式：`screen`（默认）\|`json`\|`all` |

### 9.6 安全管理

| 命令 | 说明 |
|------|------|
| `token rotate` | 重新生成 CLI 认证令牌（更新自动化脚本） |

### 9.7 服务管理（Windows SCM / Linux systemd）

| 命令 | 说明 |
|------|------|
| `service install [--config <f>]` | 注册为 OS 服务（Windows：SCM；Linux：systemd） |
| `service uninstall` | 停止并删除 OS 服务注册 |
| `service start` | 请求 OS 服务管理器启动 DBPrisma |
| `service stop` | 请求 OS 服务管理器停止 DBPrisma |
| `service status` | 查询 OS 服务管理器状态和 PID |

> **注意**：`service install/uninstall` 需要管理员权限（Windows）或 root（Linux）。

### 9.8 其他命令

| 命令 | 说明 |
|------|------|
| `clear \| cls` | 清屏 |
| `history \| hist` | 列出命令历史（128 条环形缓冲；持久化到 `<run>/dbprisma.cli_history`，跨会话保留；↑/↓ 调取） |
| `help \| ? [command]` | 显示全部命令；带命令名时显示该命令的详细用法 |
| `exit \| quit` | 退出程序（如服务正在运行则自动停止） |
| `(Tab)` | 自动补全命令名、标志和 setconfig 动词 |

---

## 10. Admin HTTP / HTTPS API

Admin 平面是**只读监控/可观测接口**（不做控制类操作；`stop`/`reload` 等仍走 CLI），默认监听 `127.0.0.1:8080`，仅接受 `GET`。

### 10.0 HTTP / HTTPS 双协议（同端口）

Admin 端口对每个连接**按首字节自动识别协议**（TLS 握手记录以 `0x16` 开头，HTTP 请求以方法名开头），无需为 HTTP/HTTPS 分别开端口：

| 是否配置证书 | 该端口行为 |
|---|---|
| 配了 `admin_tls_pfx_file` 或 `admin_tls_cert_file`+`admin_tls_key_file` | **同时**接受 HTTP（明文）与 HTTPS（加密） |
| 未配置证书 | 仅 HTTP（明文） |
| 配了证书但加载失败 | **启动失败（fail-closed）**，绝不静默降级为明文 |

启用 HTTPS（PFX 方式）：

```json
"proxy": {
  "admin_listen_ip": "127.0.0.1",
  "admin_listen_port": 8080,
  "admin_bearer_token": "your-secret-token",
  "admin_tls_pfx_file": "admin.pfx",
  "admin_tls_pfx_password": "ENC2:...."
}
```

或 PEM 证书+私钥方式（`pfx` 优先级更高）：

```json
"admin_tls_cert_file": "admin_cert.pem",
"admin_tls_key_file":  "admin_key.pem"
```

> 证书路径相对 conf.json 所在目录解析；PFX 口令支持 `ENC2:`（AES-256-GCM）。`admin_tls_*` 为非热更字段，修改后须重启。HTTPS 复用与数据平面 P1 相同的服务端 TLS 栈（Windows SChannel / Linux OpenSSL）。

### 10.1 认证方式

所有端点在 `admin_bearer_token` 非空时都需鉴权（loopback 且未设 token 时免鉴权；绑定 `0.0.0.0` 时必须设 token）。两种头任选其一：

```bash
# HTTPS（生产推荐；自签证书加 -k）
curl -k -H "Authorization: Bearer <admin_bearer_token>" https://127.0.0.1:8080/health

# HTTP（内网/loopback 排查）
curl -H "X-Admin-Token: <admin_bearer_token>" http://127.0.0.1:8080/metrics
```

### 10.2 API 端点

| 端点 | 方法 | 说明 |
|------|------|------|
| `/health` | GET | 健康/状态快照（JSON）：运行状态、uptime、mode、各 server 健康与连接数、health_indicators、performance、admin 自身计数 |
| `/metrics` | GET | Prometheus 文本格式指标（`dbprisma_up`、查询/错误计数、读写分离、CPU/内存/网络、按 server 维度指标、授权状态 `dbprisma_license_state`/`dbprisma_license_days_left` 等） |
| `/events` | GET | 最近 64 条通知事件（JSON 数组）：`backend_error` / `circuit_breaker` / `firewall_block` / `auth_fail` / `config_reload`，含时间戳与消息 |
| 其它路径 | GET | 返回 `404 Not Found` |
| 任意端点 | 非 GET | 返回 `405 Method Not Allowed` |

> HTTP 与 HTTPS 暴露的端点、返回内容、鉴权与访问控制**完全一致**，区别仅在于传输是否加密。

### 10.3 /health 响应示例

```json
{
  "status": "ok",
  "uptime_sec": 3600,
  "is_running": true,
  "mode": { "enabled": false, "strategy": 0, "dispatch_primary_only": false },
  "servers": [
    { "id": 1, "healthy": true, "active_clients": 42, "listening": true }
  ],
  "health_indicators": {
    "send_cas_busy_drops_c": 0, "send_cas_busy_drops_s": 0,
    "dispatch_defer_pending": 0, "dispatch_defer_drops": 0,
    "pool_in_use": 0, "shadow_send_drops": 0
  },
  "performance": {
    "total_queries": 15000, "total_errors": 0,
    "bytes_in_per_sec": 0.0, "bytes_out_per_sec": 0.0,
    "cpu_usage_pct": 4.2, "memory_bytes": 23064576
  },
  "admin": { "listening": true, "requests_total": 1, "auth_fail_total": 0, "rejected_total": 0 }
}
```

### 10.4 访问控制

```json
"proxy": {
  "admin_listen_ip": "127.0.0.1",
  "admin_listen_port": 8080,
  "admin_bearer_token": "your-secret-token",
  "admin_allow_ips": ["127.0.0.1", "10.0.0.1"],
  "admin_request_timeout_ms": 5000,
  "admin_tls_pfx_file": "admin.pfx"
}
```

**生产建议**：
- **启用 HTTPS**（配置 `admin_tls_*`），尤其在绑定非 loopback 时
- `admin_listen_ip` 设为 `127.0.0.1`（禁止外网访问）
- `admin_allow_ips` 仅允许运维堡垒机 IP（**仅支持精确 IP 字符串**，不支持 CIDR；客户端网段过滤请用 `security.ip_allowlist` / `ip_denylist`，支持 CIDR，须 **restart**）
- 使用随机 32+ 字符的 `admin_bearer_token`

---

## 11. 升级与回滚

### 11.1 升级步骤

```bash
# 1. 备份当前可执行文件和配置
cp /opt/dbprisma/bin/DBPrisma /opt/dbprisma/bin/DBPrisma.bak
cp /opt/dbprisma/config/conf.json /opt/dbprisma/config/conf.json.bak.$(date +%Y%m%d_%H%M%S)

# 2. 停止服务（XR:> 提示符下执行 stop，或用 systemd）
#    XR:> stop
systemctl stop dbprisma

# 3. 替换可执行文件
cp new-build/DBPrisma /opt/dbprisma/bin/

# 4. 验证配置兼容性（checkconfig 为 Shell 子命令）
./DBPrisma checkconfig

# 5. 启动新版本
systemctl start dbprisma
```

```text
# 6. 在 XR:> 提示符下验证运行状态
XR:> status
XR:> backends
```

### 11.2 回滚步骤

```bash
# 若升级后发现问题
systemctl stop dbprisma
cp /opt/dbprisma/bin/DBPrisma.bak /opt/dbprisma/bin/DBPrisma
cp /opt/dbprisma/config/conf.json.bak.20260607_140000 /opt/dbprisma/config/conf.json
systemctl start dbprisma
```

### 11.3 配置版本

`version` **必须为三段 semver**（如 `"0.0.1"`），与引擎 **`PA_VERSION_STRING` 的前三段**一致（引擎完整版如 `0.0.1.568` 含构建号，配置中不写第四段）。旧整数 `config_version` 不再支持；复制模板时请写入 `"version": "0.0.1"` 并执行 `checkconfig`。

---

## 12. 多进程与集群部署

### 12.1 多进程部署

DBPrisma 支持同一主机上运行多个实例（不同端口）：

```bash
# 实例1：主实例（start 子命令的选项顺序不限）
./DBPrisma start --config /opt/dbprisma/config/conf-primary.json

# 实例2：备用实例
./DBPrisma start --config /opt/dbprisma/config/conf-backup.json
```

每个实例使用独立的配置文件、日志目录和 PID 文件。

### 12.2 负载均衡集群

在 DBPrisma 前使用负载均衡器（Nginx / HAProxy / F5）：

```
应用层
  │
LB (Nginx/HAProxy)
  │         │
DBPrisma-1  DBPrisma-2  ...
  │         │
  └── MySQL 主从集群
```

### 12.3 高可用建议

1. **Watchdog**：启用 `watchdog.enabled = true`，自动检测并重启卡死进程
2. **Circuit Breaker**：启用熔断保护，后端故障时自动隔离
3. **多后端配置**：主库故障时经 `backends` + 手动标记 primary 切换（`auto_failover` 当前为**计划中/未实现**，见 [operations-manual §7.2](operations-manual.md)）；读写分离/分发可经 `mode.strategy` 配置
4. **Drain 优雅下线**：滚动重启时先 `drain`，再停止旧实例，启动新实例
5. **健康检查集成**：将 `/health` 端点集成到 LB 健康检查

---

*更多配置细节请参阅 [配置参考手册](configuration-reference.md)*  
*日常运维操作请参阅 [运维手册](operations-manual.md)*  
*安全加固请参阅 [安全加固指南](security-hardening-guide.md)*
