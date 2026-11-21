# SSL 行为矩阵

> 本文档列出方案 A / B 在各种客户端 SSL 模式、`tls_termination`、`backend_ssl_mode` 配置和后端 SSL 支持状态下的完整行为。

---

## 术语说明

| 术语 | 含义 |
|---|---|
| P1 | 客户端 ↔ 代理 这一段连接 |
| P2 | 代理 ↔ 后端 MySQL 这一段连接 |
| TLS 终结 | 代理在 P1 侧解密 TLS，以明文处理 MySQL 协议（凭据替换等），再决定 P2 是否重加密 |
| TLS 重加密 | P2 侧代理以 TLS 客户端身份重新加密后发往后端（需 `backend_ssl_mode` ≠ `disabled`） |
| TLS 透传 | 代理不解密，直接将加密字节原样转发（方案 B 专用）|
| 自签名 | 代理启动时运行时自动生成的 RSA-2048 自签名证书 |
| 真实 CA | 由受信任 CA 签发的证书（配置 `tls_p1_pfx_file` 或 `tls_p1_cert_file` + `tls_p1_key_file`）|
| — | 不适用（连接在 P1 阶段已失败，P2 / SQL 捕获无从谈起）|

---

## P2 配置速查（`backend_ssl_mode`）

P2 行为由 **`backend_ssl_mode`** 独立控制，与客户端 `--ssl-mode` 无关。默认 `disabled`（明文）。

| 值 | 加密 | 验证证书链 | 验证主机名 | 说明 |
|---|---|---|---|---|
| `disabled` | ✗ | ✗ | ✗ | 明文（默认） |
| `required` | ✓ | ✗ | ✗ | 加密但不验证后端证书 |
| `verify_ca` | ✓ | ✓ | ✗ | 验证 CA 链；可配 `tls_p2_ca_file` / `tls_p2_ca_path`（否则 OS 信任库） |
| `verify_identity` | ✓ | ✓ | ✓ | 完整验证；hostname=`host`；可配 `tls_p2_ca_file` / `tls_p2_ca_path` |

---

## 客户端 SSL 模式速查（P1）

| 模式 | 含义 |
|---|---|
| `DISABLED` | 强制明文，不使用 TLS |
| `PREFERRED` | 优先 TLS，服务端不支持时自动降级明文（大多数客户端默认值）|
| `REQUIRED` | 强制 TLS，不验证服务端证书（自签名可接受）|
| `VERIFY_CA` | 强制 TLS + 验证 CA 链，不验证主机名 |
| `VERIFY_IDENTITY` | 强制 TLS + 验证 CA 链 + 验证主机名（CN/SAN 需匹配）|

安全性：`DISABLED` < `PREFERRED` < `REQUIRED` < `VERIFY_CA` < `VERIFY_IDENTITY`

---

## 方案 A — P1 行为矩阵

下表 P2 列均假设 **`backend_ssl_mode=disabled`（默认）**。P2 非默认行为见下一节。

| 方案 | `tls_termination` | 证书类型 | 后端支持 SSL | 客户端 SSL 模式 | P1（Client ↔ 代理） | P2（代理 ↔ Server） | SQL 捕获 |
|---|---|---|---|---|---|---|---|
| A | false | — | 任意 | DISABLED | 明文 | 明文 | 正常 |
| A | false | — | 任意 | PREFERRED | 自动降级明文 | 明文 | 正常 |
| A | false | — | 任意 | REQUIRED | 连接失败 | — | — |
| A | false | — | 任意 | VERIFY_CA | 连接失败 | — | — |
| A | false | — | 任意 | VERIFY_IDENTITY | 连接失败 | — | — |
| A | true | 自签名 | 是 | DISABLED | 明文 | 明文 | 正常 |
| A | true | 自签名 | 是 | PREFERRED | TLS 终结 | 明文 | 正常 |
| A | true | 自签名 | 是 | REQUIRED | TLS 终结 | 明文 | 正常 |
| A | true | 自签名 | 是 | VERIFY_CA | 连接失败（自签名无法验证） | — | — |
| A | true | 自签名 | 是 | VERIFY_IDENTITY | 连接失败（自签名无法验证） | — | — |
| A | true | 自签名 | 否 | DISABLED | 明文 | 明文 | 正常 |
| A | true | 自签名 | 否 | PREFERRED | TLS 终结 | 明文 | 正常 |
| A | true | 自签名 | 否 | REQUIRED | TLS 终结 | 明文 | 正常 |
| A | true | 自签名 | 否 | VERIFY_CA | 连接失败（自签名无法验证） | — | — |
| A | true | 自签名 | 否 | VERIFY_IDENTITY | 连接失败（自签名无法验证） | — | — |
| A | true | 真实 CA ※ | 是 | DISABLED | 明文 | 明文 | 正常 |
| A | true | 真实 CA ※ | 是 | PREFERRED | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 是 | REQUIRED | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 是 | VERIFY_CA | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 是 | VERIFY_IDENTITY | TLS 终结（需 CN/SAN 匹配） | 明文 | 正常 |
| A | true | 真实 CA ※ | 否 | DISABLED | 明文 | 明文 | 正常 |
| A | true | 真实 CA ※ | 否 | PREFERRED | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 否 | REQUIRED | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 否 | VERIFY_CA | TLS 终结 | 明文 | 正常 |
| A | true | 真实 CA ※ | 否 | VERIFY_IDENTITY | TLS 终结（需 CN/SAN 匹配） | 明文 | 正常 |

---

## 方案 A — P2 行为矩阵（`backend_ssl_mode`）

P1 行为见上表；本节仅描述 **代理 ↔ 后端** 链路。适用于 `scheme=auth`（方案 A）。

| `backend_ssl_mode` | 后端支持 SSL | P2 结果 | 失败时行为 |
|---|---|---|---|
| `disabled` | 是 / 否 | 明文 | — |
| `required` | 是 | TLS 重加密 | P2 TLS 启动失败 → 降级明文 + WARN |
| `required` | 否 | 明文 | WARN |
| `verify_ca` | 是 | TLS 重加密 + 验链 | fail-closed（拒绝连接） |
| `verify_ca` | 否 | 连接失败 | fail-closed |
| `verify_identity` | 是 | TLS 重加密 + 验链 + hostname | fail-closed（拒绝连接） |
| `verify_identity` | 否 | 连接失败 | fail-closed |

> **组合示例：** `tls_termination=true` + 客户端 `REQUIRED` + `backend_ssl_mode=required` + 后端支持 SSL → P1 TLS 终结 + P2 TLS 重加密（自动化测试 profile `A_t1_p2`）。

> **P1+P2 双 TLS 认证时序：** 在强制后端 TLS 模式（`required` / `verify_ca` / `verify_identity`）下，客户端 HandshakeResponse（认证）会被**推迟**到 P2 后端 TLS 握手完成、链路激活之后，再以正确序号在 P2 TLS 上重放，而**不**会在 P2 TLS 就绪前以明文发往后端。这样可避免后端（正等待 TLS 握手）静默丢弃明文认证包，从而消除高并发 connect 下的偶发 `ERROR 2013`。此为代理侧建连时序处理，对客户端透明。

---

## 方案 B — 完整行为矩阵

方案 B 不使用 `backend_ssl_mode`（P2 由客户端与后端端到端协商，代理透传）。

| 方案 | `tls_termination` | 证书类型 | 后端支持 SSL | 客户端 SSL 模式 | P1（Client ↔ 代理） | P2（代理 ↔ Server） | SQL 捕获 |
|---|---|---|---|---|---|---|---|
| B | — | — | 任意 | DISABLED | 明文 | 明文 | 正常 |
| B | — | — | 否 | PREFERRED | 明文（Greeting 无 SSL 能力位） | 明文 | 正常 |
| B | — | — | 是 | PREFERRED | 常为 TLS 透传（保留后端 SSL 能力位；与 MySQL 客户端实现一致） | TLS 透传 | 不可用 |
| B | — | — | 是 | REQUIRED | TLS 透传 | TLS 透传（端到端） | 不可用 |
| B | — | — | 是 | VERIFY_CA | TLS 透传（校验**后端**证书） | TLS 透传 | 不可用 |
| B | — | — | 是 | VERIFY_IDENTITY | TLS 透传（校验后端证书 + 主机名） | TLS 透传 | 不可用 |
| B | — | — | 否 | REQUIRED | 连接失败（后端无 SSL） | — | — |
| B | — | — | 否 | VERIFY_CA | 连接失败（后端无 SSL） | — | — |
| B | — | — | 否 | VERIFY_IDENTITY | 连接失败（后端无 SSL） | — | — |

---

## 注意事项

- ※ **真实 CA 证书**：在 `servers[]` 中配置 `tls_p1_pfx_file`（推荐）或 `tls_p1_cert_file` + `tls_p1_key_file`（RSA PRIVATE KEY PEM）。未配置时仍使用运行时自签名证书（CN=`DBPrisma` 或 `tls_p1_cert_cn`）。证书路径相对 `conf.json` 所在目录。加载失败时 `start` 整体失败。
- **P1 与 P2 独立配置：** `tls_termination` 控制 P1；`backend_ssl_mode` 控制 P2。二者可任意组合（例如 P1 TLS 终结 + P2 明文，或 P1 明文 + P2 TLS）。
- 方案 A + `tls_termination=true` + `PREFERRED`：代理向客户端声明支持 SSL，`PREFERRED` 模式的客户端会主动选择 TLS，实际行为等同于 `REQUIRED`。
- 方案 B：仅当**后端 Initial Handshake 带 SSL 能力位**时，代理向客户端保留 `CLIENT_SSL`（否则 `REQUIRED` / `VERIFY_*` 无法发起 SSL Request）。后端不支持 SSL 时仍剥除该标志，行为与「明文」行一致。
- 方案 B + 后端支持 SSL + `PREFERRED`：多数客户端会协商 TLS 并进入 **TLS 透传**（与 MySQL 官方语义一致）；若需强制明文请用 `DISABLED`。
- 方案 B 的 `tls_termination` 字段填了也无效，代码自动忽略。
- 自动化：`python tests\ssl\test_ssl_behavior_matrix.py` 按本文矩阵抽样验证（profile `A_t0` / `A_t1` / `A_t1_p2` / `B`）。
- 自签名证书若导入客户端信任库，`VERIFY_CA` 可以通过；`VERIFY_IDENTITY` 还需证书 CN/SAN 与连接地址匹配。

---

## P2 TLS 的后端角色范围（方案 A）

上面的 P2 矩阵描述的是**会话承载后端**（客户端会话直连的那个后端）。在多后端策略下，P2 TLS 按后端**角色**生效：

| 后端角色 | 模式 | P2 TLS | 说明 |
| --- | --- | --- | --- |
| 会话主后端 | 所有 | ✓ | `dispatch` 主、`read_write` `role=write` 主，走会话 `server_sock` 的 P2 路径（`start_p2_tls`）|
| 只读从库 `role=read` | `read_write` | ✓ | **2026-06 新增**：各从库按自己的 `backend_ssl_mode` / CA 建 P2 TLS（阻塞握手 + 命令阶段透明加解密）|
| 影子库 shadow | `dispatch` | ✓ | **2026-06 新增**：各影子按自己的 `backend_ssl_mode` / CA 建 P2 TLS。TLS 影子绕过异步发送队列，改为 `send_lock` 下加密+阻塞发送（保证 TLS 记录序），drain 线程透明解密 |

> 因此「`dispatch` + 全后端强制 TLS」现已对**主后端与影子链路**端到端加密。注意：TLS 影子不复用 DB 预热池（预热 socket 为明文），改为冷连接 + TLS 握手，建连略慢于明文影子；明文影子（`backend_ssl_mode=disabled`）行为与发送路径不变。

---

*文档版本：2026-06-28。P2：`backend_ssl_mode` + 可选 `tls_p2_ca_file` / `tls_p2_ca_path`；后端角色范围已覆盖会话主后端、`read_write` 从库与 `dispatch` 影子（三者均支持 P2 TLS）；强制后端 TLS 下客户端认证推迟至 P2 TLS 激活并重放（修复双 TLS 高并发偶发 ERROR 2013）。*