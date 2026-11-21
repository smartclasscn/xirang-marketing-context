# 方案 B（透传）与 MySQL 客户端兼容性说明

---

## 一、认证插件各版本支持情况

### 客户端支持的插件

| 插件名 | MySQL 5.x 客户端 | MySQL 8.x 客户端 | MySQL 9.x 客户端 |
|---|---|---|---|
| `mysql_old_password` | ✅ | ✅ | ❌ |
| `mysql_native_password` | ✅ | ✅ | ❌（9.0 删除） |
| `sha256_password` | ✅（5.6.6起） | ✅ | ✅ |
| `caching_sha2_password` | ❌ | ✅ | ✅ |

### 服务器支持的插件

| 插件名 | MySQL 5.x 服务器 | MySQL 8.x 服务器 | MySQL 9.x 服务器 |
|---|---|---|---|
| `mysql_old_password` | ✅ | ✅（8.0.36 移除） | ❌ |
| `mysql_native_password` | ✅ | ✅（8.0.34 废弃，仍可用） | ❌（9.0 删除） |
| `sha256_password` | ✅（5.6.6起） | ✅ | ✅ |
| `caching_sha2_password` | ❌ | ✅（8.0.4起，默认） | ✅（默认） |

### 客户端 → 服务器 能否连通（无代理）

| 客户端 \ 服务器 | MySQL 5.x | MySQL 8.x | MySQL 9.x |
|---|---|---|---|
| **5.x 客户端** | ✅ | ✅（账号需用 native_password） | ❌（5.x 无 caching_sha2） |
| **8.x 客户端** | ✅ | ✅ | ✅ |
| **9.x 客户端** | ❌（9.x 无 native_password） | ⚠️（账号需改为 caching_sha2） | ✅ |

---

## 二、认证流程基础

### 认证插件的工作方式

认证插件只有一个，同时决定握手和认证两个阶段的行为：

- **握手阶段**：服务器在握手包里声明使用哪个插件，并发出 nonce（随机挑战数）
- **认证阶段**：客户端用该插件规定的算法，把密码 + nonce 算成认证数据发给服务器

**主动权在服务器**，服务器先声明插件，客户端被动适应：

```
服务器握手 → 声明插件名
      ↓
客户端查找本地是否有该插件实现
      ↓
找到  → 继续发 HandshakeResponse，完成认证
找不到 → 直接断开连接，失败
```

### AuthSwitchRequest 机制

服务器在收到客户端的 HandshakeResponse 后，若需要切换插件，可发送 AuthSwitchRequest：

```
服务器握手: plugin = A
客户端发 HandshakeResponse（声明想用插件 B）
      ↓
服务器发 AuthSwitchRequest，同意切换
      ↓
客户端用插件 B 重新计算，再发一次认证数据
```

**注意**：AuthSwitchRequest 只能由服务器发起，客户端无法主动要求切换。

---

## 三、方案 B + MySQL 9.x 客户端 + MySQL 8.x 后端（mysql_native_password）失败过程

### 前提条件

- 代理配置：方案 B（servers 里 user 字段为空，纯透传）
- 后端：MySQL 8.x，账号使用 `mysql_native_password`
- 客户端：MySQL 9.x

### 失败步骤

**步骤 1：客户端发起 TCP 连接**
MySQL 9.x 客户端向代理端口发起 TCP 连接。

**步骤 2：代理接受连接，并连接后端**
代理接受客户端连接，同时向 MySQL 8.x 后端发起 TCP 连接。

**步骤 3：后端发握手包给代理**
MySQL 8.x 后端发出握手包：
- `plugin = mysql_native_password`
- `nonce = [20字节随机数]`

**步骤 4：代理原样转发握手包给客户端**
方案 B 不介入，握手包一字不改地转给 MySQL 9.x 客户端。

**步骤 5：客户端读到插件名 `mysql_native_password`**
MySQL 9.x 客户端解析握手包，得到插件名。

**步骤 6：客户端查找本地插件实现**
MySQL 9.0 已将 `mysql_native_password` 的客户端实现代码彻底删除，查找失败。

**步骤 7：客户端直接断开连接**
客户端不发任何认证数据，直接关闭 TCP 连接，报错：
```
ERROR 2013: Lost connection to MySQL server during handshake
```

**步骤 8：代理感知到断开，释放 SESSION**
代理检测到客户端 TCP 关闭，随即关闭与后端的连接。

### 根本原因

服务器声明 `mysql_native_password`，9.x 客户端不支持这个插件，在握手阶段就断开了，认证阶段根本没有启动。

---

## 四、为什么代理无法在方案 B 中解决这个问题

### 思路一：代理把握手包里的插件名改成 `caching_sha2_password`

```
后端握手: plugin = mysql_native_password（期待 SHA1）
代理改写: plugin = caching_sha2_password（转给客户端）
客户端发: SHA256(密码, nonce) ──> 代理
代理转发: SHA256 ──> 后端
后端期待: SHA1，收到 SHA256，认证失败 ✗
```

代理拿到 SHA256 结果，没有明文密码，无法反推原始密码，也无法重新算出 SHA1。

**结论**：改了握手，认证就对不上后端。

### 思路二：利用 AuthSwitchRequest

- AuthSwitchRequest 由服务器发给客户端，客户端无法主动发起
- 9.x 客户端在收到握手包后直接断开，根本没有发 HandshakeResponse
- 没有 HandshakeResponse，服务器就没有机会发 AuthSwitchRequest
- 即使强行触发，后端账号是 `mysql_native_password` 注册的，无法切换到 `caching_sha2_password`

**结论**：三道墙都过不去。

### 核心矛盾

```
不改握手 → 9.x 客户端看到 mysql_native_password，直接断开
改了握手 → 客户端发 SHA256，后端期待 SHA1，代理无法转换（没有明文密码）
```

方案 B 不持有明文密码，这是透传模式的本质，无法绕过。

---

## 五、解决方案

### 方案一：迁移后端账号（推荐，最简单）

在后端 MySQL 8.x 执行：

```sql
ALTER USER 'root'@'%' IDENTIFIED WITH caching_sha2_password BY '你的密码';
FLUSH PRIVILEGES;
```

执行后：
- 后端握手包声明 `caching_sha2_password`
- 9.x 客户端支持该插件，正常发 HandshakeResponse
- 代理原样透传，完全没问题

**代理代码无需任何修改。**

### 方案二：改用方案 A（代理接管认证）

在 `conf.json` 里填写后端账号，并将 `scheme` 设为 `"auth"`（必填）：

```json
"servers": [{
    "scheme":   "auth",
    "user":     "root",
    "password": "ENC2:xxxxxxxx"
}]
```

> ⚠️ `scheme` 字段**必须显式填写**，省略时 `checkconfig` / `start` 会报错（`scheme` 不在合法值列表中）。

代理切换为方案 A，自动桥接 9.x 客户端（SHA256）↔ mysql_native_password 后端（SHA1）。

代价：客户端需要使用 `proxy_users.json` 里定义的代理账号登录，不能直接使用后端账号。

---

## 六、方案 A 如何解决这个问题（对比参考）

方案 A 在 `conf.json` 里配置了后端明文密码，所以代理可以：

```
后端握手: plugin = mysql_native_password（期待 SHA1）
代理改写: plugin = caching_sha2_password（转给 9.x 客户端）

客户端发: SHA256(密码, nonce) ──> 代理
代理验证: 用 proxy_users.json 里的密码校验 SHA256，通过

代理计算: 用 conf.json 里配置的明文密码重新算 SHA1(密码, nonce)
代理发送: SHA1 ──> 后端
后端验证: ✅ 通过
```

代理持有明文密码是方案 A 能桥接两种算法的根本原因。

---

## 七、插件协商机制补充说明

### 客户端可以在 HandshakeResponse 里声明不同的插件

当服务器在握手包里声明插件 A，客户端若不支持，可以在 HandshakeResponse 里声明自己想用的插件 B，服务器收到后发 AuthSwitchRequest 同意切换，客户端再用插件 B 完成认证。

```
服务器握手: plugin = A
客户端发 HandshakeResponse: auth_plugin_name = B（我想用这个）
服务器查账号: 账号支持 B，同意
服务器发 AuthSwitchRequest: 切换到插件 B
客户端用插件 B 认证，成功
```

**注意**：服务器发出的 AuthSwitchRequest 内容由账号注册时绑定的插件决定，服务器不能随意切换到账号不支持的插件。

---

### 各版本客户端对插件协商的支持情况

| 客户端版本 | 支持插件协商 | 说明 |
|---|---|---|
| **MySQL 5.x** | ✅ | 支持，可协商到 `mysql_native_password` |
| **MySQL 8.x** | ✅ | 支持，可协商到任意已支持的插件 |
| **MySQL 9.x** | ❌ | 硬编码删除 `mysql_native_password`，协商到它也无法使用，且遇到该插件直接断开不协商 |

---

### 实际案例

**5.x 客户端连 8.x 服务器（每天都在发生）**

```
8.x 服务器握手: plugin = caching_sha2_password（服务器全局默认）
5.x 客户端: 我没有 caching_sha2_password
5.x 客户端发 HandshakeResponse: auth_plugin_name = mysql_native_password
服务器查账号: 账号是 native_password，同意
服务器发 AuthSwitchRequest: 切换到 mysql_native_password
5.x 客户端用 SHA1 认证，成功 ✅
```

**9.x 客户端连 8.x 服务器（native_password 账号，无代理）**

```
8.x 服务器握手: plugin = mysql_native_password（账号绑定）
9.x 客户端: 硬编码拒绝，直接断开，不发 HandshakeResponse
失败 ✗
```

**9.x 客户端尝试协商（假设不硬编码拒绝）**

```
8.x 服务器握手: plugin = mysql_native_password
9.x 客户端发 HandshakeResponse: auth_plugin_name = caching_sha2_password
服务器查账号: 账号是 native_password，只能用这个
服务器发 AuthSwitchRequest: 切回 mysql_native_password
9.x 客户端: 我还是没有 mysql_native_password → 失败 ✗
```

即使 9.x 客户端愿意协商，账号绑定的插件是 `mysql_native_password`，服务器只会切回来，9.x 客户端仍然无法处理，结果一样失败。

---

### 对方案 B 的影响

方案 B 在**明文连接**下会剥除服务端/客户端握手包中的 `CLIENT_COMPRESS` 与部分扩展能力位（保留 `CLIENT_SSL`），以便 `capture` 能解析 SQL；**仅当客户端发送标准 32 字节 SSL Request** 时才进入 SSL 透传（与“HandshakeResponse 里带 CLIENT_SSL 位”区分）。真实 TLS 会话仍完全透明，但 `capture` 不可用。

方案 B 纯透传认证与插件协商，代理不替换凭据。因此：

- 5.x / 8.x 客户端的插件协商能力在方案 B 下**完全保留，正常工作**
- 9.x 客户端遇到 `mysql_native_password` 后端，**无论是否经过代理，结果相同**，代理无法改变这一行为
