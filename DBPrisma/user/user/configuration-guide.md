# DBPrisma 配置手册

> 面向部署与运维的配置说明。字段逐项说明见 [configuration-reference.md](configuration-reference.md)；SSL 行为矩阵见 [ssl-behavior-matrix.md](ssl-behavior-matrix.md)。

| 文件 | 路径 | 作用 |
|------|------|------|
| 代理主配置 | `bin/<Platform>/<arch>/config/conf.json` | 监听端口、后端、TLS、日志、负载均衡 |
| 代理用户 | `bin/<Platform>/<arch>/config/proxy_users.json` | 方案 A 客户端登录代理的账号（与后端 MySQL 用户无关） |

修改配置后：

- **结构变更**（监听端口、TLS 证书路径、`mode`、新增/删除 server 等）→ **`stop` → `start`**
- **非结构字段**（日志级别、idle/timeout、脱敏开关等）→ 可用 **`reload`**（服务运行中）
- **`setconfig`** 只改磁盘上的 json，**不会**自动作用于运行中进程（改完后 `reload` 或重启）

Lab 模板：`config/windows/conf_lab.json`、`config/linux/conf_lab.json`（部署时复制到运行目录 `bin/<Platform>/<arch>/config/conf.json` 或 Linux `/opt/dbprisma/config/conf.json`）。

安全/防火墙参考模板：`config/conf_security.example.json`（含 `security` + `log_config.firewall_log_*` + CIDR 黑白名单示例；改后须 **restart**）。

---

## 1. 快速上手

### 1.1 目录与进程

```text
bin/                          # Windows 运行目录（<exe>）
  DBPrisma.exe                # Shell 子命令：start/checkconfig/setconfig/service；stop/reload 等为 XR:> 运行时命令
  PrismaAgent.dll             # 代理核心（由 EXE 加载）
  config/
    conf.json
    proxy_users.json
    conf.json.bak.*           # reload/setconfig 备份（同目录）
    certs/                    # 建议：P1 证书放此目录（路径相对 config/）
  run/                        # lock、pid、cli_token、cli_history、machine.key（运行时创建）
  logs/                       # DBPrisma.log、防火墙/慢查询日志
  capture/                    # capture.json

Linux 部署示例（/opt/dbprisma/）：
  DBPrisma
  libPrismaAgent.so           # 与 DBPrisma 同目录；RPATH $ORIGIN
  config/conf.json
  config/proxy_users.json
  run/  logs/  capture/       # 同上四目录布局
```

### 1.2 最小透明代理（方案 B）

客户端连 `listen_port`，代理原样转发到 `host:port`，不替换凭据。

```json
{
  "servers": [{
    "scheme": "transparent",
    "host": "127.0.0.1",
    "port": 3306,
    "listen_port": 33006
  }]
}
```

```bat
mysql -h 127.0.0.1 -P 33006 -u root -p -e "SELECT 1"
```

### 1.3 最小认证代理（方案 A）

客户端用 **proxy_users** 中的账号连代理；代理用 **servers** 里的 `user`/`password` 连后端。

`bin/<Platform>/<arch>/config/proxy_users.json`（**必须**含 `users` 数组）：

```json
{
  "users": [
    { "username": "app", "password": "app_secret" }
  ]
}
```

`conf.json`：

```json
{
  "servers": [{
    "scheme": "auth",
    "host": "127.0.0.1",
    "port": 3306,
    "listen_port": 33006,
    "user": "root",
    "password": "backend_secret"
  }]
}
```

```bat
mysql -h 127.0.0.1 -P 33006 -u app -papp_secret -e "SELECT 1"
```

### 1.4 启动前检查

```bat
cd /d F:\DBPrisma\bin
DBPrisma.exe checkconfig
DBPrisma.exe start
```

Linux：

```bash
cd /opt/dbprisma
./DBPrisma checkconfig config/conf.json
./DBPrisma start config/conf.json
```

存在 `[ERROR]` 时退出码为 1，**不要** `start`（证书路径错误、端口冲突等会在此暴露）。

---

## 2. 方案选择（A / B）

| | 方案 A `scheme: "auth"` | 方案 B `scheme: "transparent"` |
|--|-------------------------|--------------------------------|
| 客户端连谁 | `listen_port`（代理） | `listen_port`（代理） |
| 客户端账号 | `proxy_users.json` | 后端 MySQL 真实账号 |
| 代理是否改握手/凭据 | 是（拦截认证） | 否（透传） |
| P1 TLS 终结 `tls_termination` | **支持** | 无效（忽略） |
| SQL 抓取/审计 | 支持 | 不支持加密通道内容 |

**`scheme` 为必填字段**，必须显式写 `"auth"` 或 `"transparent"`。省略或填写其他值，代理启动时将报错退出。

---

## 3. 证书与 TLS（重点）

### 3.1 两段连接

```text
  mysql.exe / 应用                DBPrisma 代理                 MySQL 服务器
  ─────────────────              ───────────────              ──────────────
        │    P1（客户端侧）              │    P2（后端侧）              │
        └──────── TLS? ─────────►└──────── TLS? ─────────►└──────── TLS? ──
```

| 段 | 谁出示服务端证书 | 在 json 里配什么 |
|----|------------------|------------------|
| **P1** | **代理**（你买的证通常配这里） | `tls_termination` + `tls_p1_*` |
| **P2** | **MySQL** | `backend_ssl_mode`（代理当 TLS 客户端，**不**使用 `tls_p1_*`） |

### 3.2 买的证书装在哪

| 安装位置 | 适用场景 |
|----------|----------|
| 代理 `tls_p1_pfx_file` 或 `tls_p1_cert_file` + `tls_p1_key_file` | 客户端连代理要走 HTTPS/TLS（方案 A + `tls_termination: true`） |
| MySQL 服务器自身 | 客户端直连 MySQL，或方案 B 端到端 TLS 透传 |

**私钥与 PFX 只放在服务端**（代理或 MySQL），不要下发到每台客户端。

### 3.3 P1：代理侧证书（方案 A）

**仅当** `"scheme": "auth"` 且 `"tls_termination": true` 时生效。

| 配置项 | 说明 |
|--------|------|
| `tls_p1_pfx_file` | **推荐**。PFX/P12，可含完整链 |
| `tls_p1_pfx_password` | PFX 密码，可用 `ENC2:`。**最大 128 字节**（含 `ENC2:` 前缀），超长会被截断导致解码失败 |
| `tls_p1_cert_file` + `tls_p1_key_file` | PEM；私钥支持 **`BEGIN RSA PRIVATE KEY`** 与 **`BEGIN PRIVATE KEY`（PKCS#8）**（Windows/Linux） |
| `tls_p1_cert_cn` | 未配证书文件时，运行时自签名证书的 CN（默认 `DBPrisma`） |

路径均**相对于 `conf.json` 所在目录**（一般为 `bin/<Platform>/<arch>/config/`）。

示例（正式证书）：

```json
{
  "scheme": "auth",
  "listen_port": 33006,
  "tls_termination": true,
  "tls_p1_pfx_file": "certs/proxy.pfx",
  "tls_p1_pfx_password": "ENC2:xxxx",
  "user": "root",
  "password": "ENC2:yyyy"
}
```

| 情况 | 行为 |
|------|------|
| 未配置 `tls_p1_*` | 每次启动自动生成 RSA 自签名（进程退出即销毁） |
| 配置了路径但加载失败 | **`start` 失败**（fail-closed） |
| `tls_p1_pfx_file` 与 PEM 成对同时配置 | **`checkconfig` ERROR** |
| 修改证书文件 | 必须 `stop` → `start` |

**P1 / P2 证书格式速查**（与 [configuration-reference.md §2.3](configuration-reference.md#23-tls-终结字段scheme-a-专属) 一致）：

| 格式 | P1 | P2 |
|------|:--:|:--:|
| PFX / P12 | ✓ 服务端证+私钥 | ✗ |
| PEM 证书 + PEM 私钥 | ✓（含 PKCS#8） | ✗ |
| PEM CA 文件 | ✗ | ✓ `tls_p2_ca_file` |
| PEM/DER CA 目录 | ✗ | ✓ `tls_p2_ca_path` |
| OS 信任库 | ✗ | ✓（verify 模式且未配自定义 CA） |
| 运行时自签名 | ✓ | ✗ |

### 3.4 P2：代理 → MySQL

| 配置项 | 默认 | 可选值 | 说明 |
|--------|------|--------|------|
| `backend_ssl_mode` | `"disabled"` | `disabled` / `required` / `verify_ca` / `verify_identity` | P2 TLS 策略（镜像 MySQL 客户端 `--ssl-mode`） |
| `tls_p2_ca_file` | `""` | PEM 或 DER 路径 | 仅 `verify_ca` / `verify_identity`；类似 `mysql --ssl-ca`；路径相对 `conf.json` |
| `tls_p2_ca_path` | `""` | CA 目录路径 | 仅 verify 模式；目录内 PEM/DER（**非递归**）；类似 `mysql --ssl-capath` |

| 值 | 加密 | 验证证书链 | 验证主机名 | 适用场景 |
|---|------|-----------|-----------|---------|
| `disabled` | ✗ | ✗ | ✗ | 明文（默认） |
| `required` | ✓ | ✗ | ✗ | 加密但不验证后端证书（实验室） |
| `verify_ca` | ✓ | ✓ | ✗ | 验证 CA 链，不要求 hostname 匹配（内网 IP 直连） |
| `verify_identity` | ✓ | ✓ | ✓ | 完整验证（生产推荐） |

- 代理**不会**把 P1 的 PFX/PEM 当作客户端证书发给 MySQL（当前无 mTLS）。
- `verify_ca` / `verify_identity` 默认依赖 OS 信任库。自签 MySQL CA 可配 **`tls_p2_ca_file`** 和/或 **`tls_p2_ca_path`**（推荐，无需导入 OS）；二者可同时配置（合并信任）。
- 配了 `tls_p2_ca_*` 时**仅信任自定义来源**（不再用 OS 库）；在 `disabled` / `required` 下设置会 **WARN 并忽略**。
- `tls_p2_ca_file` / `tls_p2_ca_path` 修改后须 `stop` → `start`（不可 `reload`）。
- `verify_identity` 使用 `host` 字段做主机名匹配；若后端是 IP，请改用 `verify_ca`。

示例（P1 正式证 + P2 加密但不校验 MySQL 证）：

```json
{
  "scheme": "auth",
  "tls_termination": true,
  "tls_p1_pfx_file": "certs/proxy.pfx",
  "backend_ssl_mode": "required",
  "host": "127.0.0.1",
  "port": 3306,
  "listen_port": 33006,
  "user": "root",
  "password": "secret"
}
```

示例（P2 完整验证 + 自定义 CA，类似 `mysql --ssl-ca`）：

```json
{
  "scheme": "auth",
  "tls_termination": true,
  "tls_p1_pfx_file": "certs/proxy.pfx",
  "backend_ssl_mode": "verify_ca",
  "tls_p2_ca_file": "certs/mysql-ca.pem",
  "host": "192.168.10.37",
  "port": 3306,
  "listen_port": 33006,
  "user": "root",
  "password": "secret"
}
```

示例（P2 verify_ca + 自定义 CA 文件与目录）：

```json
{
  "scheme": "auth",
  "tls_termination": true,
  "tls_p1_pfx_file": "certs/proxy.pfx",
  "backend_ssl_mode": "verify_ca",
  "tls_p2_ca_file": "certs/mysql-ca.pem",
  "tls_p2_ca_path": "certs/ca-dir",
  "host": "192.168.10.34",
  "port": 3306,
  "listen_port": 33006,
  "user": "root",
  "password": "secret"
}
```

示例（P2 verify_identity + OS 信任库）：

```json
{
  "scheme": "auth",
  "tls_termination": true,
  "tls_p1_pfx_file": "certs/proxy.pfx",
  "backend_ssl_mode": "verify_identity",
  "host": "mysql.example.com",
  "port": 3306,
  "listen_port": 33006,
  "user": "root",
  "password": "secret"
}
```

### 3.5 客户端（mysql.exe / 应用）要不要证书文件

| 问题 | 答案 |
|------|------|
| 要不要在客户端装买的 **服务端 PFX/私钥**？ | **不要** |
| 要不要 `--ssl-cert` / `--ssl-key`？ | 一般**不要**（除非 MySQL 本身要求客户端证书登录，与 DBPrisma 方案 A 无关） |
| 什么时候要带文件？ | 仅 `VERIFY_CA` / `VERIFY_IDENTITY` 时可选 **`--ssl-ca=ca.pem`**（信任链），或依赖系统已信任该 CA |

客户端校验的是**连接对象**在 TLS 握手中出示的证：连代理验代理的证（P1），方案 B 透传时验 MySQL 的证。

---

## 4. mysql 客户端连接示例

以下均连 **`listen_port`**，用户名为 **proxy_users** 中的账号（方案 A）。

### 4.1 代理自签名或未校验（测试常用）

**MySQL 8.0+：**

```bat
mysql -h 127.0.0.1 -P 33006 -u app -p ^
  --ssl-mode=REQUIRED ^
  -e "SELECT 1"
```

**MySQL 5.7**（`-h` 与 `-P` 分开写；关闭证书校验）：

```bat
mysql -h 127.0.0.1 -P 33006 -u app -p ^
  --ssl-mode=REQUIRED ^
  --ssl-verify-server-cert=0 ^
  -e "SELECT 1"
```

### 4.2 代理已配置 CA 签发的 `tls_p1_*`

```bat
mysql -h dbproxy.example.com -P 33006 -u app -p ^
  --ssl-mode=VERIFY_IDENTITY ^
  -e "SELECT 1"
```

`-h` 必须与证书 CN/SAN 一致。CA 已在系统信任库时可省略 `--ssl-ca`。

自签但希望 VERIFY：导出 CA 后：

```bat
mysql -h 127.0.0.1 -P 33006 -u app -p ^
  --ssl-mode=VERIFY_CA ^
  --ssl-ca=C:\path\to\ca.pem ^
  -e "SELECT 1"
```

### 4.3 明文（未开 P1 或未要求 TLS）

```bat
mysql -h 127.0.0.1 -P 33006 -u app -p --ssl-mode=DISABLED -e "SELECT 1"
```

5.7 在 `tls_termination: false` 时建议显式 `DISABLED`，避免默认尝试 SSL 导致 `2013`。

### 4.4 客户端 ssl-mode 与代理配置对照

| 代理 P1 | 客户端模式 | 结果 |
|---------|------------|------|
| `tls_termination: false` | `REQUIRED` / `VERIFY_*` | 失败 |
| `tls_termination: true` + 自签名 | `REQUIRED` | 正常 |
| `tls_termination: true` + 自签名 | `VERIFY_CA` / `VERIFY_IDENTITY` | 失败（除非导入信任或 `--ssl-ca`） |
| `tls_termination: true` + 真实 CA 证书 | `VERIFY_CA` / `VERIFY_IDENTITY` | 正常（IDENTITY 需主机名匹配） |

完整矩阵见 [ssl-behavior-matrix.md](ssl-behavior-matrix.md)。

---

## 5. 代理用户与密码

### 5.1 两套账号

```text
客户端 ──(proxy_users 用户名/密码)──► 代理 ──(servers[].user/password)──► MySQL
```

| 存储位置 | 用途 |
|----------|------|
| `proxy_users.json` | 客户端登录代理 |
| `servers[].user` / `password` | 代理连接后端（可 `ENC2:`） |

`proxy_users.json` **非空**时，仅列表内用户可连；**空文件或不存在**时方案 A 允许任意客户端名（仍会用 servers 后端凭据），生产环境建议配置白名单。

### 5.2 加密密码

`passwd` 是 XR:> 提示符下的运行时命令；推荐**不带参数**交互输入（避免明文进入 shell 历史/进程列表）：

```text
DBPrisma.exe          # 进入 XR:> 提示符
XR:> passwd
```

输出形如 `ENC2:...`（AES-256-GCM，机器绑定，唯一加密格式），写入 json：

```json
"password": "ENC2:rJ3kLmP9..."
```

后端密码：`setconfig srvpwd <id>` 或 `passwd`。代理用户密码：`setpwd <username>`（写入 `proxy_users.json` 并加密）。

### 5.3 遗留认证插件 `allow_legacy_password`

`conf.json` → `proxy` 段：

```json
"proxy": {
  "allow_legacy_password": false
}
```

| 值 | 行为 |
|----|------|
| `false`（**默认**） | 拒绝 8 字节 legacy 握手（推荐，生产默认） |
| `true` | 允许参与 `mysql_old_password`（scramble323）；仅对 MySQL 4.1 前遗留客户端才需要 |

此字段支持 **`reload`**（无需 `stop` → `start`）。

### 5.4 IP 认证失败限流（可配置，须重启）

方案 A 下，代理对同一客户端 IP 的认证失败次数进行计数；超阈值后封禁，返回 MySQL 错误 **1129**。成功登录后计数清零；表满时打 WARN 日志 `ip_ratelimit: table full`，不误封已有 ban。

上述阈值和窗口可通过 `conf.json` 的 `security` 段配置（**默认值**如下）：

```json
"security": {
  "auth_fail_max":        5,
  "auth_fail_window_ms":  60000,
  "auth_fail_ban_ms":     300000
}
```

| 字段 | 默认 | 说明 |
|------|------|------|
| `auth_fail_max` | `5` | 触发封禁的失败次数（1–100） |
| `auth_fail_window_ms` | `60000` | 滑动时间窗口（ms，≥1000） |
| `auth_fail_ban_ms` | `300000` | 封禁持续时长（ms，0=永久） |

> ⚠️ `security` 段修改后须 **`stop` → `start`** 才能生效，**不支持** `reload`。完整 security 字段说明见 [configuration-reference.md §十三](configuration-reference.md#十三安全配置security)。

---

## 6. 负载均衡 `mode`（可选）

```json
"mode": {
  "enabled": true,
  "listen_ip": "0.0.0.0",
  "listen_port": 3300,
  "strategy": "round_robin"
}
```

| strategy | 说明 |
|----------|------|
| `round_robin` | 轮询（默认） |
| `weighted` | 按 `servers[].weight` |
| `least_conn` | 最少连接 |
| `dispatch` | 写广播到多后端；可设某一 server `"primary": true`（详见 reference §三） |
| `read_write` | **（Phase 1–3）** 写走 `role=write`、读走 `role=read` 池；见 [read-write-split 设计](../design/read-write-split.md) |

**`read_write` 补充（Phase 2–3）**

| 字段 | 说明 |
|------|------|
| `read_after_write_sticky_ms` | 写后 N ms 内 SELECT 强制 primary（0=关） |
| `read_fallback_to_primary` | 读池全失败时回退 primary |
| `read_health_skip_failures` | 读副本连续失败 N 次后暂时跳过 |
| SQL 提示 | `/* master */` / `/* slave */` / `/* read */` |

`read_strategy`、`read_after_write_sticky_ms`、`read_fallback_to_primary`、`read_health_skip_failures` 支持 **`reload`**；`mode.strategy`、`servers[].role` 等结构字段须 **`stop` → `start`**。字段逐项见 [configuration-reference.md §三](configuration-reference.md#三mode-负载均衡可选)。

自动化：`tests/quality/test_proxy_read_write.py`（QRW-01～08）。

- `mode.listen_port` 与各 `servers[].listen_port` **并行**，互不影响。
- **仅 scheme A 后端参与 mode 路由**（`round_robin` / `weighted` / `least_conn` / `dispatch` / `read_write` 均如此）；`scheme: "transparent"` 会被排除并 Warning，无 auth 后端时 mode 无法启动。
- `dispatch` 写 SQL 会等待影子后端就绪，超时返回 **45000**（详见 [configuration-reference.md](configuration-reference.md)）。

---

## 7. 预热连接

| 字段 | 方案 A | 方案 B |
|------|--------|--------|
| `warmup_tcp_connections` | 支持 | 支持 |
| `warmup_db_connections` | 支持 | **忽略** |

与 `tls_termination: true` 可同时使用；预热握手的合成 Greeting 会带 `CLIENT_SSL`，便于客户端升级 TLS。

---

## 8. 日志

```json
"logging": {
  "file": "DBPrisma.log",
  "level": "info",
  "max_size_mb": 50,
  "rotate_count": 3
}
```

- 日志路径相对于 `bin/`（或绝对路径）。
- `start` 时 EXE 会把 `max_size_mb` / `rotate_count` 同步到内部抓取日志轮转（与 `DBPrisma.log` 策略一致）。
- 省略 `max_size_mb` 时真实默认为 **10 MB**（示例用 50 是模板选值）；`rotate_count` 默认 **3**。

---

## 8A. SQL 捕获与字面值脱敏

代理在认证模式（Scheme A）下可在命令阶段捕获每条 SQL（含其结果分类：OK / ERR / 未知）并写入 `capture.json`，同时在 `info` 及以上日志级别打印。涉及预处理语句：

- `COM_STMT_PREPARE` 捕获完整 SQL 文本；
- `COM_STMT_EXECUTE` 捕获为 `EXECUTE stmt_id=N params=(v1, v2, …)`，按已知参数类型解码；未知类型以 `?` 标记后安全截断；未知 `stmt_id` 标记 `?unknown_stmt`；
- `COM_STMT_CLOSE` 仅从内部映射表中删除条目，不产生捕获行；
- 单条 SQL 最长保留 **4096** 字节（`PA_MAX_SQL_LEN`），超长截断。

```json
"proxy": {
  "capture_redact_literals": true
}
```

| 取值 | 行为 |
|------|------|
| 省略且存在 `proxy` 段 | 默认 **`true`**（字面量脱敏） |
| 显式 `false` | 原样记录 SQL 字面值，便于排错（仅限受控 lab） |
| 显式 `true` | 字符串字面值替换为 `'?'`、数字字面值替换为 `?`；**同时作用于** `capture.json`、UI、影子后端 WARN 日志 |

> 生产环境**强烈建议**置 `true`。未开启时，凡含密码/令牌/PII 的 SQL 都会原文进入日志与捕获文件。

---

## 9. 常用场景配置片段

### 9.1 生产：方案 A + 购买的证书 + 仅加密到 MySQL

```json
{
  "servers": [{
    "id": 1,
    "scheme": "auth",
    "host": "10.0.0.10",
    "port": 3306,
    "listen_ip": "0.0.0.0",
    "listen_port": 33006,
    "enabled": true,
    "tls_termination": true,
    "tls_p1_pfx_file": "certs/proxy.example.com.pfx",
    "tls_p1_pfx_password": "ENC2:...",
    "backend_ssl_mode": "required",
    "user": "proxy_backend",
    "password": "ENC2:..."
  }],
  "logging": { "level": "info", "max_size_mb": 100, "rotate_count": 5 }
}
```

客户端：`--ssl-mode=VERIFY_IDENTITY`，`-h` 为证书域名。

### 9.2 开发：自签名 P1，mysql 不校验

不配 `tls_p1_*`，保持 `tls_termination: true`。客户端 `--ssl-mode=REQUIRED`（5.7 加 `--ssl-verify-server-cert=0`）。

### 9.3 方案 B：客户端直连 MySQL 的 TLS（透传）

不在代理配 `tls_p1_*`。MySQL 侧启用 SSL；客户端：

```bat
mysql -h 127.0.0.1 -P 33006 -u root -p ^
  --ssl-mode=VERIFY_IDENTITY ^
  --ssl-ca=C:\path\to\mysql-ca.pem ^
  -e "SELECT 1"
```

此处 `--ssl-ca` 是 **MySQL 的 CA**，不是代理证书。

### 9.4 多版本后端（测试矩阵）

每个后端一个 `servers[]` 项，不同 `port` / `listen_port`（如 33006–33009），同一 `proxy_users.json`。

---

## 10. 命令行配置维护

不启动服务时编辑 json（**运行中** `setconfig` 写盘会被拒绝；已运行实例用 `reload` 或 `restart`）。

```text
setconfig show server 1
setconfig server 1
setconfig srvuser 1 root
setconfig srvpwd 1
setconfig use D:\backup\conf.json
```

代理用户：

```text
useradd alice
setpwd alice
userdel alice
userlist
```

**热加载与重启：**

```text
reload              # 应用可热更字段（日志级别、idle/timeout、connect_timeout、脱敏、allow_legacy_password、admin、resilience 等）；结构不变
reload D:\conf.json
restart             # stop + start（证书/端口/mode/security 等变更时用此）
```

`reload` **不能**改：监听端口、`mode.strategy`/`mode.listen_*`、`tls_p1_*`、`backend_ssl_mode`、`tls_p2_ca_file`、`tls_p2_ca_path`、`max_clients`、`worker_threads`、`max_reassemble_bytes`、server 增删、`resilience.backend_tcp_keepalive`、`admin_listen_ip`/`admin_listen_port`、**`security.*`**、`capture.*`、`circuit_breaker.*`、`watchdog.*`、`tls.*`、`log_config.log_format`/`slow_query_ms`/`slow_query_file`/`log_include_sql`、`config_manage.*` → 必须 **`stop` → `start`**（详见 [configuration-reference.md §九](configuration-reference.md#九热加载-reload)）。`log_config.firewall_log_file`/`firewall_log_max_size_mb`/`firewall_log_rotate_count` **支持 `reload`** 热更。

**运维操作命令（XR:> 提示符下）：**

```text
status              # 运行时统计摘要
status verbose      # 详细统计（版本、模式、后端列表、连接池）
backends            # 后端服务器健康状态（含熔断器状态）
shutdown            # 优雅停止（同 stop）
shutdown 30         # 30 秒倒计时后停止
shutdown 0 force    # 立即强制停止
shutdown cancel     # 取消等待中的 shutdown 计划
drain <server_id>        # 按后端停止新路由，等现有会话结束
drain cancel <server_id> # 取消该后端 drain，恢复路由
pause               # 暂停接受新连接（不断开已有连接）
resume              # 恢复接受新连接
flush [server_id]   # 清空预热连接池并后台重建（省略=全部）
```

**CLI 管理面 token**（`stop` / `reload` / `setconfig` / `useradd` / `passwd` 等）：

- 首次 `start` 在 `run/dbprisma.cli_token` 生成 32 字节 hex token（仅属主可读；默认 `<exe>/run/`，可由 `--run-dir`/`DBPRISMA_HOME`/`paths.run_dir` 覆盖）
- 交互式控制台（TTY）可省略 token
- 脚本/管道（非 TTY）须提供 token，且这些命令经 stdin 管道送入（非 Shell 子命令），例如：`echo stop | DBPrisma.exe --token <hex>` 或先 `export DBPRISMA_CLI_TOKEN=<hex>` 再 `echo stop | DBPrisma.exe`

详见 [configuration-reference.md §七–§十二](configuration-reference.md#七配置编辑命令-setconfig)。

---

## 11. 排错清单

| 现象 | 可能原因 | 处理 |
|------|----------|------|
| `start` 失败，日志 TLS / PFX | 路径错、密码错、PFX 与 PEM 同时配、PEM 加载失败 | `checkconfig`；优先 PFX；Windows exotic 密钥用 PFX |
| `REQUIRED` 连不上 | 未开 `tls_termination` | 方案 A 设 `tls_termination: true` 或客户端 `DISABLED` |
| 5.7 错误 `2013` | 未开 P1 但客户端默认 SSL | `--ssl-mode=DISABLED` |
| 高并发 connect 偶发 `2013`（双 TLS：`tls_termination=true` + `backend_ssl_mode` 强制 TLS） | **已修复**（无需配置变更）：旧版在 P2 后端 TLS 就绪前把明文认证发给了等待 TLS 握手的后端，被静默丢弃。现已将客户端认证推迟至 P2 TLS 激活后再以正确序号重放 | 升级到含本修复的版本即可；若仍偶发，区分是否为 5.7 后端 TLS 协议版本互通问题（见下条/[14](../technical/14-risk-technical-debt-register.md) TD-020） |
| MySQL 5.7 后端 + 全链路 SSL + dispatch 偶发 `2013` | 5.7 TLS 栈在并发握手压力下拒绝 OpenSSL-3 `ClientHello`（`tlsv1 alert protocol version`）——**后端互通问题，可重试**，根因不同于上一条的代理建连时序 | 重试；或评估后端 5.7 升级 / TLS 版本配置（详见 [14](../technical/14-risk-technical-debt-register.md) TD-020） |
| `certificate verify failed` | 自签名 + `VERIFY_CA` | 改 `REQUIRED` 或 `--ssl-ca` / 导入信任 |
| `Hostname mismatch` | `VERIFY_IDENTITY` 与 `-h` 不一致 | `-h` 改为证书域名 |
| 认证失败 | proxy 用户错 | 查 `proxy_users.json` 与 `userlist` |
| 连上但后端拒绝 | 后端凭据错 | 查 `servers[].user` / `password` |
| `45000` dispatch 写失败 | 影子后端未就绪 | 检查各 shadow `enabled`、网络、`connect_timeout_ms` |
| 错误 **1129** | IP 认证失败过多被限流 | 等待 5 分钟或从其它 IP 重试；查 brute-force 日志 |

---

## 11A. 生产部署加固（建议）

| 项 | 建议 |
|----|------|
| P2 链路 | `backend_ssl_mode` ≥ `required`（生产推荐 `verify_identity`）；自签 MySQL CA 用 `tls_p2_ca_file` / `tls_p2_ca_path` |
| 方案 B | 勿将 `listen_ip: 0.0.0.0` 暴露公网；绑内网 + 防火墙 |
| Legacy 密码 | `allow_legacy_password: false` |
| SQL 捕获脱敏 | `capture_redact_literals: true` |
| 代理用户 | `proxy_users.json` 非空白名单；密码用 `ENC2:` |
| 明文后端密码 | lab 允许；生产用 `ENC2:` 加密 |
| IP 安全策略 | 配置 `security.max_connections_per_ip` 防止单 IP 过载；`security.ip_denylist` 屏蔽已知恶意 IP；`security.hide_server_version: true` 防版本探测 |
| 熔断器 | 生产环境建议启用 `circuit_breaker.enabled: true`；`error_rate_threshold_pct=50`，`min_requests=20` 作为起点 |
| 看门狗 | 建议启用 `watchdog.enabled: true`，`action: "restart"` 配合外部 supervisor 自动恢复 |
| Admin HTTP | 若启用，**必须**设置 `admin_bearer_token` 且监听 `127.0.0.1`（或通过 `admin_allow_ips` 限制访问） |

---

## 12. 相关文档

| 文档 | 内容 |
|------|------|
| [configuration-reference.md](configuration-reference.md) | 全字段默认值、类型、约束 |
| [ssl-behavior-matrix.md](ssl-behavior-matrix.md) | P1/P2 × 客户端 ssl-mode 完整表 |
| [scheme-b-transparent.md](scheme-b-transparent.md) | 透明代理与 MySQL 9 客户端 |
| [../README.md](../README.md) | 文档索引与构建测试 |

---

*文档版本：2026-06-13。§11 排错清单新增双 TLS 高并发 connect 偶发 `2013` 已修复说明（客户端认证推迟至 P2 TLS 激活后重放，无需配置变更），并与 5.7 后端 TLS 协议版本互通问题（TD-020）区分；保留原 5.7/`DISABLED` 指引。（2026-06-05）`allow_legacy_password` 默认值更正为 `false`（可 reload）；reload 命令描述补全 admin/resilience 可热更字段；截面引用链接更新至新章节编号。字段逐项见 [configuration-reference.md](configuration-reference.md)。*
