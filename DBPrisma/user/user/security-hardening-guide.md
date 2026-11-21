# DBPrisma 安全加固指南

**版本**: v1.1 | **日期**: 2026-06-06 | **适用版本**: DBPrisma ≥ 1.0  
**受众**: 安全工程师、系统管理员、合规审计人员

> 可复制仓库模板 `config/conf_security.example.json`（同步到 `bin/<Platform>/<arch>/config/`）作为 `security` + 防火墙审计日志的起点；修改后须 **restart** 生效。

---

## 目录

1. [安全架构概述](#1-安全架构概述)
2. [密码安全](#2-密码安全)
3. [网络访问控制](#3-网络访问控制)
4. [连接限速与防暴力破解](#4-连接限速与防暴力破解)
5. [SQL 防火墙](#5-sql-防火墙)
6. [TLS 加密配置](#6-tls-加密配置)
7. [Admin HTTP 安全](#7-admin-http-安全)
8. [数据脱敏与隐私保护](#8-数据脱敏与隐私保护)
9. [文件系统安全](#9-文件系统安全)
10. [审计日志](#10-审计日志)
11. [生产环境安全检查清单](#11-生产环境安全检查清单)
12. [合规性说明（金融/电信级）](#12-合规性说明金融电信级)

---

## 1. 安全架构概述

DBPrisma 提供多层次纵深防御：

```
互联网 / 内网客户端
        │
    ┌───▼──────────────────────────────────┐
    │  Layer 1: IP 访问控制                │
    │  (ip_allowlist / ip_denylist)        │
    └───┬──────────────────────────────────┘
        │
    ┌───▼──────────────────────────────────┐
    │  Layer 2: TLS 加密（P1）            │
    │  (tls_termination)                   │
    └───┬──────────────────────────────────┘
        │
    ┌───▼──────────────────────────────────┐
    │  Layer 3: 认证 & 限速               │
    │  (proxy_users + auth_fail 封禁)      │
    └───┬──────────────────────────────────┘
        │
    ┌───▼──────────────────────────────────┐
    │  Layer 4: 连接配额                  │
    │  (max_connections_per_ip)            │
    └───┬──────────────────────────────────┘
        │
    ┌───▼──────────────────────────────────┐
    │  Layer 5: SQL 防火墙               │
    │  (sql_firewall + sql_block_patterns) │
    └───┬──────────────────────────────────┘
        │
    后端 MySQL（TLS P2 可选）
```

---

## 2. 密码安全

### 2.1 配置文件密码加密（ENC2）

**禁止**在 `conf.json` 中存储明文密码。所有密码字段均应使用 ENC2 加密格式：

`passwd` 是 XR:> 提示符下的运行时命令（先运行 `./DBPrisma` 进入交互模式）：

```text
./DBPrisma                       # 进入 XR:> 提示符
XR:> passwd                      # 交互式，不记录到 shell 历史
# Enter plain password: ██████████
# Encrypted: ENC2:c2VjcmV0MTIz...
```

在 `conf.json` 中使用：
```json
{
  "servers": [
    {
      "user": "dbprisma_user",
      "password": "ENC2:c2VjcmV0MTIz..."
    }
  ],
  "proxy": {
    "admin_bearer_token": "ENC2:..."
  }
}
```

> **规则**：任何 CI/CD 流水线或代码仓库中**绝对禁止**出现明文密码；`conf.json` 不应提交到版本控制系统，或必须使用 .gitignore 排除。

### 2.2 密码强度要求

代理用户密码建议满足：
- 长度 ≥ 12 位
- 包含大小写字母、数字和特殊字符（`!@#$%^&*`）
- 不使用常见弱密码（`password`、`123456`、`admin` 等）

```bash
# 生成强随机密码示例（Linux），先复制到剪贴板备用
openssl rand -base64 24
```

`useradd` 是 XR:> 运行时命令，且只接受用户名参数、密码通过交互式提示输入（不回显、不接受命令行密码参数）：

```text
./DBPrisma                       # 进入 XR:> 提示符
XR:> useradd appuser             # 随后在提示符粘贴上面生成的强密码
```

### 2.3 Admin Token 安全

```bash
# 生成高熵 Token
openssl rand -hex 32
# 输出：a8f5f167f44f4964e6c998dee827110c...
```

`token` 是 XR:> 运行时命令，周期性轮换（建议每 90 天）：

```text
./DBPrisma                       # 进入 XR:> 提示符
XR:> token rotate
```

**Admin Token 绝不应该**：
- 硬编码在应用代码中
- 通过 HTTP（非 HTTPS）明文传输
- 出现在日志文件中

---

## 3. 网络访问控制

### 3.1 IP 白名单（最严格，优先推荐）

允许指定**主机 IP 或 CIDR** 连接（如 `10.0.0.0/8`、`203.0.113.5`）：

```json
"security": {
  "ip_allowlist": [
    "192.168.1.100",
    "10.0.2.15",
    "172.16.0.88"
  ]
}
```

> **说明**：`ip_allowlist` 中的 IP 永不受 `auth_fail_*` 封禁与限速影响。修改 `security.*` 后须 **`stop` → `start`**（不支持 `PA_Reload`）。网段级控制请用上游防火墙/NACL。

### 3.2 IP 黑名单（拒绝已知恶意 IP）

```json
"security": {
  "ip_denylist": [
    "203.0.113.55",
    "10.20.30.40"
  ]
}
```

更新步骤（**须重启**）：
```bash
# 1. 编辑 conf.json 中 ip_denylist 数组（或用 setconfig 写盘，setconfig 为 Shell 子命令）
./DBPrisma setconfig security.ip_denylist append "203.0.113.55"
```

```text
# 2. 在 XR:> 提示符下使配置生效（security.* 需重启），并可选踢掉该 IP 已有连接
XR:> restart
XR:> kick 203.0.113.55
```

### 3.3 监听地址绑定

**不要监听** `0.0.0.0`（全网卡）除非业务必须。

推荐做法：
```json
"servers": [
  {
    "listen_ip": "10.0.1.50",
    "listen_port": 3307
  }
],
"proxy": {
  "admin_listen_ip": "127.0.0.1",
  "admin_listen_port": 8080
}
```

---

## 4. 连接限速与防暴力破解

### 4.1 认证失败封禁

```json
"security": {
  "auth_fail_max": 5,
  "auth_fail_window_ms": 60000,
  "auth_fail_ban_ms": 300000
}
```

**说明**：
- `auth_fail_max = 5`：60 秒内允许 5 次认证失败
- `auth_fail_ban_ms = 300000`：超过限制后，该 IP 被封禁 5 分钟
- 封禁期间的连接请求直接被拒绝，不返回认证失败原因（防止信息泄露）

**金融级推荐**：
```json
"auth_fail_max": 3,
"auth_fail_window_ms": 30000,
"auth_fail_ban_ms": 1800000
```

### 4.2 每 IP 最大连接数

```json
"security": {
  "max_connections_per_ip": 50
}
```

防止单个 IP 耗尽所有连接资源（DDoS 防护）。

### 4.3 新连接速率限制

```json
"security": {
  "new_conn_burst_limit": 200,
  "new_conn_burst_window_ms": 1000
}
```

**说明**：每 1000 毫秒内，来自单个 IP 的新连接数不超过 200 个，超过则拒绝。

---

## 5. SQL 防火墙

### 5.1 启用 SQL 防火墙

```json
"security": {
  "sql_firewall_enabled": true,
  "sql_firewall_action": "reject",
  "sql_firewall_log_blocked": true
},
"log_config": {
  "firewall_log_file": "logs/firewall.log",
  "firewall_log_max_size_mb": 100,
  "firewall_log_rotate_count": 10
}
```

`sql_firewall_action` 取值：
- `"reject"`：拦截并返回 MySQL 错误 1148（推荐生产使用）
- `"log_only"`：仅记录 `FIREWALL_BLOCK` WARN 日志，放行命令（适合初期规则调试）

#### 独立防火墙审计日志

`log_config.firewall_log_file` 配置后，每次 `FIREWALL_BLOCK` 事件**同时**写入该专用文件（主日志继续接收，二者互不干扰）。

日志格式为固定单行文本，便于 `grep` / SIEM 集成：
```
2026-06-06 14:00:00 [FIREWALL] sid=17 pat="DROP TABLE" sql="DROP TABLE users WHERE 1=1"
```

滚动行为与主日志相同：文件超过 `firewall_log_max_size_mb` MB 后旋转，保留 `firewall_log_rotate_count` 个历史备份（`.1`、`.2`…）。**`log_config.firewall_log_*` 三字段支持 `PA_Reload` 热更**（路径变更时自动重开文件，无需 stop→start）；`security.*`（规则/action/enabled）仍须 stop→start。客户端可见 ERR **1148** 时连接保持，见 [技术文档 ERR 矩阵](../technical/16-functional-spec-behavior.md#8a-mysql-err-包矩阵客户端可见)。

### 5.2 配置拦截规则

> **匹配方式**：大小写**不敏感子串匹配**（非正则）。规则字符串只要出现在 SQL 文本的任何位置即触发，首条命中优先。

```json
"security": {
  "sql_block_patterns": [
    "DROP TABLE",
    "DROP DATABASE",
    "TRUNCATE TABLE",
    "DELETE FROM",
    "ALTER TABLE",
    "GRANT ALL",
    "CREATE USER",
    "--",
    "/*",
    "UNION SELECT",
    "INTO OUTFILE",
    "LOAD DATA INFILE",
    "EXEC ",
    "xp_cmdshell"
  ]
}
```

**规则说明**：
| 规则 | 防护目标 |
|------|----------|
| `DROP TABLE` / `DROP DATABASE` | 防止删表/库 |
| `TRUNCATE TABLE` | 防止清空表 |
| `DELETE FROM` | 防止全表删除（包括有 WHERE 的删除，可按需精细化） |
| `UNION SELECT` | SQL 注入防护 |
| `INTO OUTFILE` | 防止文件写入 |
| `LOAD DATA INFILE` | 防止文件读取 |
| `--` / `/*` | SQL 注释注入 |

> **提示**：规则越短命中越宽泛。`DELETE FROM` 会匹配所有 DELETE 语句；生产部署时应根据业务 SQL 模式评估误拦截风险，先用 `log_only` 模式测试。

### 5.3 规则调试流程

1. 先设置 `sql_firewall_action = "log_only"`（只记录不拦截）
2. 运行正常业务，观察日志中的匹配记录
3. 确认无误拦截后，切换为 `"reject"`

```bash
# 1. 写盘：临时 log_only（setconfig 为 Shell 子命令；或直接编辑 conf.json）
./DBPrisma setconfig security.sql_firewall_action set log_only
```

```text
# 2. 在 XR:> 提示符下生效（security.* 必须重启）
XR:> restart
```

```bash
# 3. 观察命中
grep "FIREWALL_BLOCK" logs/dbprisma.log | tail -50
tail -50 logs/firewall.log    # 若配置了 firewall_log_file

# 4. 确认无误后改回 reject
./DBPrisma setconfig security.sql_firewall_action set reject
```

```text
# 然后在 XR:> 提示符下再次重启
XR:> restart
```

### 5.4 预编译语句（Prepared Statement）覆盖

DBPrisma SQL 防火墙对预编译语句实现**两阶段全覆盖**：

| 阶段 | 检测内容 |
|------|----------|
| `COM_STMT_PREPARE` | PREPARE 时用原始 SQL 文本检查，命中则立即拦截并拒绝 PREPARE |
| `COM_STMT_EXECUTE` | EXECUTE 时**同时检查**：① 合成记录文本 `EXECUTE stmt_id=N params(...)`；② 原始 PREPARE SQL（从 `fw_stmt_map` 查得）。即使 PREPARE 发生在规则收紧之前，EXECUTE 仍能被规则拦截 |
| `COM_STMT_CLOSE` | 清除 `fw_stmt_map` 中对应条目，无需防火墙介入 |

`fw_stmt_map` 每 session 最多缓存 32 条预编译语句（`FW_STMT_MAP_MAX = 32`），每条 SQL 保留前 256 字节。

### 5.5 已知限制

| 限制 | 说明 |
|------|------|
| `ssl_passthrough` 模式（方案 B 透明代理）| 流量全程加密转发，代理无法解密 SQL，防火墙**不可用** |
| `COM_STMT_SEND_LONG_DATA` | BLOB 等大参数碎片，无完整 SQL 文本，无法检查 |
| 规则类型 | 当前为子串匹配，**不支持正则表达式**；如需精细化规则请使用更精确的子串 |

### 5.6 紧急回滚（误拦生产 SQL）

| 目标 | 配置变更 | 生效方式 |
|------|----------|----------|
| 立即放行 | `sql_firewall_enabled: false` | **stop → start** |
| 仅停止拦截、保留审计 | `sql_firewall_action: "log_only"` | **stop → start** |
| 停用专用审计文件 | 清空 `log_config.firewall_log_file` | **`reload`**（热更，无需重启） |

详细步骤：[迁移指南 §7A](../technical/18-migration-upgrade-guide.md#7a-关闭或回滚-sql-防火墙)、[运维 Runbook §7A](../technical/09-operations-runbook.md#7a-事件响应剧本-playbooks) 剧本 1。

### 5.7 SQL 防火墙最佳实践

- 生产环境初始建议从 `log_only` 模式运行 1 周，积累真实 SQL 模式
- 谨慎使用宽泛子串（如 `DELETE FROM`），可能误拦截合法 SQL
- 定期审查拦截日志，更新规则
- 对于只读副本监听端口，可额外添加 `INSERT`/`UPDATE`/`DELETE` 拦截

---

## 6. TLS 加密配置

### 6.1 TLS 版本限制

禁止 TLS 1.0 和 TLS 1.1（已被弃用），最低要求 TLS 1.2：

```json
"tls": {
  "min_tls_version": "TLS1_2"
}
```

金融/高安全要求场景，强制 TLS 1.3：
```json
"tls": {
  "min_tls_version": "TLS1_3"
}
```

### 6.2 密码套件配置

禁用弱密码套件，推荐使用 Forward Secrecy 算法：

```json
"tls": {
  "cipher_list": "ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384",
  "prefer_server_ciphers": true,
  "session_ticket_enabled": false
}
```

> **平台差异**：`cipher_list`、`prefer_server_ciphers`、`session_ticket_enabled` **仅在 Linux/OpenSSL 后端生效**；Windows SChannel **忽略**这三项（精确套件控制需依赖 OS 组策略或 OpenSSL 构建路径）。`min_tls_version` 在 Windows/Linux **两端均生效**。

**禁用的弱密码**：
- RC4（已破解）
- 3DES（SWEET32 漏洞）
- NULL 加密
- MD5 哈希

### 6.3 证书安全要求

| 要求 | 说明 |
|------|------|
| 密钥长度 | RSA ≥ 2048 位（推荐 4096），ECDSA ≥ 256 位 |
| 有效期 | 不超过 2 年（推荐 1 年以内） |
| 签名算法 | SHA-256 或更强（禁止 MD5/SHA-1） |
| SAN | 必须包含实际域名/IP（禁止使用通配符证书于生产） |
| 私钥存储 | 文件权限 `640`，属主为代理运行用户 |

### 6.4 P1（客户端侧）TLS 配置

```json
"servers": [
  {
    "tls_termination": true,
    "tls_p1_cert_file": "/opt/dbprisma/certs/server.crt",
    "tls_p1_key_file": "/opt/dbprisma/certs/server.key",
    "tls_p1_cert_cn": "dbprisma.internal.example.com"
  }
]
```

### 6.5 P2（后端侧）TLS 配置

后端通信强制验证 CA（防中间人攻击）：

```json
"servers": [
  {
    "backend_ssl_mode": "verify_identity",
    "tls_p2_ca_file": "/opt/dbprisma/certs/mysql-ca.crt"
  }
]
```

---

## 7. Admin HTTP / HTTPS 安全

### 7.1 最小化暴露面

```json
"proxy": {
  "admin_listen_ip": "127.0.0.1",
  "admin_listen_port": 8080,
  "admin_bearer_token": "用-openssl-rand-hex-32-生成的随机值",
  "admin_allow_ips": ["127.0.0.1"],
  "admin_request_timeout_ms": 5000,
  "admin_max_header_bytes": 8192,
  "admin_tls_pfx_file": "admin.pfx",
  "admin_tls_pfx_password": "ENC2:..."
}
```

**关键点**：
- `admin_listen_ip = "127.0.0.1"`：Admin API 仅本机可访问
- `admin_allow_ips`：即使绑定本机，也额外限制白名单
- **启用 HTTPS**：配置 `admin_tls_pfx_file`（或 `admin_tls_cert_file`+`admin_tls_key_file`）后，Admin 端口在**同一端口**同时支持 HTTP 与 HTTPS（按首字节自动识别）。证书配错则**启动失败（fail-closed）**，不会静默降级为明文。绑定非 loopback 时**必须**启用 HTTPS。
- 若需远程访问，使用 SSH 隧道代替开放端口：
  ```bash
  ssh -L 8080:127.0.0.1:8080 user@dbprisma-host
  ```

### 7.2 Token 最佳实践

```bash
# 生成强 Token
TOKEN=$(openssl rand -hex 32)
echo "Generated token: $TOKEN"
```

```text
# 存入配置（加密）：passwd 为 XR:> 运行时命令，对 Token 进行 ENC2 加密
./DBPrisma                       # 进入 XR:> 提示符
XR:> passwd
# 在 conf.json 中配置 ENC2 加密后的 Token
```

### 7.3 定期轮换

```text
# 每季度轮换一次（token 为 XR:> 运行时命令）
./DBPrisma                       # 进入 XR:> 提示符
XR:> token rotate
# 轮换后更新所有使用 Token 的监控/运维脚本
```

### 7.4 命令管道控制通道（`command_pipe`）

`command_pipe` 是带外管理控制通道（Windows 命名管道 / Linux AF_UNIX 套接字），可远程执行管理子命令，
属高权限面，须按以下要求加固：

- **保持 `local_only: true`**（默认）：收紧管道 ACL/DACL 仅限本机；`checkconfig` 会对 `false` 给出安全 WARN。
- **加密管理员密码**：用 `useradd --admin <user>` / `setpwd --admin <user>` 命令管理（密码自动 `ENC2:`
  机器绑定 AES-256-GCM 落盘），勿留明文。该文件与 `proxy_users.json` 独立，应纳入 §9 文件权限管控
  （Linux 600、属主专属）。
- **最小化白名单**：`allowed_commands` 仅放行运维所需子命令；高危项（如 `stop`/`restart`）按需移除。
- **流式抓取鉴权**：`capture` 数据通道要求客户端**首帧回传一次性 token**握手，防同机其它进程窃取明文 SQL 流；
  流内含未脱敏 SQL，消费端应受同等访问控制。
- **抓取内存上限**：`capture_queue_mb`（默认 256 MB）为有界 FIFO 上限，消费端读取过慢时丢最旧记录而非无限占用内存，
  是抗内存型 DoS 的护栏；按物理内存与峰值流量评估，勿调过大。`capture_buf_kb`（默认 64 KB）控制数据通道传输缓冲。
- **默认禁用**：不需要时保持 `enabled: false`。详见 [配置参考手册 §二十三](configuration-reference.md)。

---

## 8. 数据脱敏与隐私保护

### 8.1 SQL 参数脱敏

在日志和 SQL 捕获中，自动将参数值替换为 `?`，防止敏感数据写入日志：

```json
"proxy": {
  "capture_redact_literals": true
}
```

开启后，`SELECT * FROM users WHERE password = 'secret'` 在日志中记录为 `SELECT * FROM users WHERE password = ?`。

### 8.2 隐藏服务器版本信息

防止攻击者获取 MySQL 版本进行针对性攻击：

```json
"security": {
  "hide_server_version": true
}
```

开启后，客户端握手包中的版本号被替换为 `5.7.0-DBPrisma`。

### 8.3 客户端属性过滤

MySQL 8.0 客户端在连接时会发送客户端属性（程序名、版本等），可过滤：

```json
"security": {
  "strip_client_attributes": true
}
```

### 8.4 IP 地址脱敏（日志）

在日志中对客户端 IP 进行脱敏（仅保留前三段）：

```json
"security": {
  "mask_client_ip_in_log": true
}
```

脱敏示例：`192.168.1.100` → `192.168.1.x`

---

## 9. 文件系统安全

### 9.1 目录权限（Linux）

```bash
# 安装目录
chmod 750 /opt/dbprisma

# 配置文件（含加密密码，严格限制）
chmod 640 /opt/dbprisma/config/conf.json
chmod 640 /opt/dbprisma/config/proxy_users.json
chown dbprisma:dbprisma /opt/dbprisma/config/*.json

# TLS 证书私钥（最高机密）
chmod 600 /opt/dbprisma/certs/*.key
chmod 644 /opt/dbprisma/certs/*.crt
chown dbprisma:dbprisma /opt/dbprisma/certs/

# 可执行文件
chmod 755 /opt/dbprisma/bin/DBPrisma

# 日志目录
chmod 755 /opt/dbprisma/logs
```

### 9.2 配置文件版本控制注意事项

- **禁止**将包含密码的 `conf.json` 提交到 Git
- 如需版本控制，使用 `conf.json.template`（密码字段留空或占位符）
- 使用密钥管理系统（Vault、AWS Secrets Manager）管理敏感配置

### 9.3 日志文件安全

日志可能包含敏感信息（用户名、SQL 语句等）：

```bash
# 日志文件权限（仅属主和日志组可读）
chmod 640 /opt/dbprisma/logs/*.log
chown dbprisma:log /opt/dbprisma/logs/*.log

# 日志轮转时保持权限
# 说明：DBPrisma 内置按 logging.max_size_mb / rotate_count 自轮转主日志；
#       若另用 OS logrotate，请使用 copytruncate（DBPrisma 持有文件句柄，
#       无"重开日志"命令；flush 仅清空预热连接池，与日志无关，切勿在此使用）。
# /etc/logrotate.d/dbprisma
/opt/dbprisma/logs/*.log {
    daily
    rotate 30
    compress
    copytruncate
    create 640 dbprisma log
}
```

---

## 10. 审计日志

### 10.1 开启详细安全日志

```json
"log_config": {
  "log_format": "json",
  "slow_query_ms": 1000,
  "slow_query_file": "logs/slow_query.log",
  "log_include_sql": false
}
```

> 合规要求较高场景：`log_include_sql = true` 并确保日志存储加密。

### 10.2 关键安全事件日志

DBPrisma 会记录以下安全事件（可通过 `grep` 监控）：

| 日志关键词 | 含义 |
|-----------|------|
| `AUTH_FAIL` | 认证失败 |
| `IP_BANNED` | IP 因认证失败被封禁 |
| `CONN_LIMIT` | 连接数超限被拒绝 |
| `FIREWALL_BLOCK` | SQL 被防火墙拦截 |
| `IP_DENIED` | IP 黑名单命中 |
| `RATELIMIT` | 连接速率限制触发 |
| `TLS_HANDSHAKE_FAIL` | TLS 握手失败 |

```bash
# 实时监控安全事件
tail -f logs/dbprisma.log | grep -E "AUTH_FAIL|IP_BANNED|FIREWALL_BLOCK|IP_DENIED"
```

### 10.3 日志集中收集（推荐）

将日志发送到集中日志平台（ELK、Splunk、Loki）：

```bash
# 使用 Filebeat 收集
# filebeat.yml 示例
filebeat.inputs:
  - type: log
    paths:
      - /opt/dbprisma/logs/dbprisma.log
    json.keys_under_root: true
    json.add_error_key: true
output.elasticsearch:
  hosts: ["elasticsearch:9200"]
```

---

## 11. 生产环境安全检查清单

### 11.1 部署前必须完成

- [ ] 所有密码已使用 `ENC2` 加密，无明文密码
- [ ] `admin_bearer_token` 已生成随机强 Token
- [ ] `admin_listen_ip` 设为 `127.0.0.1`（非 `0.0.0.0`）
- [ ] `admin_allow_ips` 仅允许运维 IP
- [ ] 如 Admin 需对外/跨主机访问，已配置 `admin_tls_*` 启用 HTTPS
- [ ] `hide_server_version` 已启用
- [ ] TLS P1 已配置（客户端侧加密）
- [ ] `min_tls_version` 设为 `TLS1_2` 或更高
- [ ] 弱密码套件已禁用（`cipher_list` 已配置；**仅 Linux/OpenSSL 生效**，Windows SChannel 依赖 `min_tls_version` + OS 组策略）
- [ ] SQL 防火墙已启用（`sql_firewall_enabled = true`）
- [ ] 认证失败封禁已配置（`auth_fail_max` / `auth_fail_ban_ms`）
- [ ] 每 IP 连接数已限制（`max_connections_per_ip`）
- [ ] `capture_redact_literals` 已启用（SQL 参数脱敏）
- [ ] 文件系统权限已按规范设置
- [ ] TLS 证书私钥权限 `600`

### 11.2 运行中定期检查

- [ ] 每 90 天轮换 `admin_bearer_token`
- [ ] 每年轮换 TLS 证书（或到期前 30 天）
- [ ] 每季度审查代理用户列表，删除废弃账号
- [ ] 每月审查 IP 黑白名单
- [ ] 每周检查安全事件日志（`AUTH_FAIL`、`FIREWALL_BLOCK` 频率）
- [ ] 定期测试 SQL 防火墙规则有效性（注入测试）

### 11.3 禁止事项

- ❌ 禁止在配置文件中使用明文密码
- ❌ 禁止将 `conf.json`（含密码）提交到版本控制
- ❌ 禁止 Admin HTTP 绑定公网地址
- ❌ 禁止使用 TLS 1.0/1.1
- ❌ 禁止关闭 SQL 防火墙而不记录原因和时间窗口
- ❌ 禁止使用 `admin_bearer_token = "test"` 等弱 Token 在生产使用

---

## 12. 合规性说明（金融/电信级）

DBPrisma 在以下方面满足金融和电信行业合规要求：

### 12.1 数据传输安全（符合 PCI DSS）

| 要求 | DBPrisma 支持方式 |
|------|-----------------|
| 禁止明文传输卡号等敏感数据 | TLS P1/P2 全链路加密 |
| 禁止 TLS 1.0/1.1 | `min_tls_version` 配置 |
| 强加密套件 | `cipher_list` 配置（仅 Linux/OpenSSL；Windows 依赖 `min_tls_version` + OS 策略） |
| 会话 Ticket 禁用 | `session_ticket_enabled = false`（仅 Linux/OpenSSL 生效；Windows SChannel 忽略） |

### 12.2 访问控制（符合等保三级）

| 要求 | DBPrisma 支持方式 |
|------|-----------------|
| 最小权限原则 | 代理用户独立账号体系 |
| IP 访问控制 | `ip_allowlist` / `ip_denylist` |
| 登录失败限制 | `auth_fail_max` 配置 |
| 管理接口保护 | Admin Token + IP 白名单 |

### 12.3 审计日志（符合 SOX/ISO 27001）

| 要求 | DBPrisma 支持方式 |
|------|-----------------|
| 操作审计日志 | 安全事件日志（`AUTH_FAIL`、`IP_BANNED` 等） |
| 慢查询审计 | `slow_query_ms` + `slow_query_file` |
| 日志完整性 | 日志文件权限保护 |
| 日志保留 | `rotate_count` 配置 |

### 12.4 安全渗透测试建议

每次大版本升级后，建议执行：
1. SQL 注入测试（验证防火墙规则）
2. 认证暴力破解测试（验证封禁机制）
3. TLS 协议测试（`testssl.sh` 或 Qualys SSL Labs）
4. 连接泛洪测试（验证速率限制）
5. Admin API 未授权访问测试

---

*配置详情参阅 [配置参考手册](configuration-reference.md)*  
*日常运维参阅 [运维手册](operations-manual.md)*（§8.5 防火墙排障、§9.2 紧急关闭）  
*技术全集索引参阅 [docs/technical/README.md](../technical/README.md)*（ERR 矩阵、双栈门禁、事件剧本）  
*安装部署参阅 [管理员手册](admin-manual.md)*

**修订历史**

| 日期 | 版本 | 说明 |
|------|------|------|
| 2026-06-05 | 1.0 | 初版 |
| 2026-06-06 | 1.1 | 修正 IP 列表（无 CIDR）、security/log_config 须 restart；§5.6 紧急回滚；链至 docs/technical |
