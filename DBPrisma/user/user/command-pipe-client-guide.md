# 命令管道客户端开发手册（C# / Java / Python）

**版本**: r1 | **日期**: 2026-06-17 | **适用版本**: DBPrisma ≥ 0.0.1.568

本手册面向需要**编程对接** DBPrisma 命令管道（`command_pipe`）的开发者，给出两条管道的
**完整线缆协议（wire protocol）**，以及 C#、Java、Python 三种语言的**可运行客户端示例**。

> **命令与参数完整列表**见 **[命令管道命令参考手册](command-pipe-command-reference.md)**（默认白名单 16 动词、§1A 管道原生 op、各命令语法与管道差异）。  
> 运维侧的启用与配置说明见 [管理员手册 §4.4](admin-manual.md#44-命令管道远程管控command_pipe)
> 与 [配置参考手册 §二十三](configuration-reference.md)；安全加固见 [安全加固指南 §7.4](security-hardening-guide.md)。

示例源码位于仓库 [`examples/command-pipe/`](../../examples/command-pipe/)：

| 语言 | 文件 | 说明 |
|------|------|------|
| Python | [`examples/command-pipe/python/dbprisma_pipe.py`](../../examples/command-pipe/python/dbprisma_pipe.py) | 可复用客户端类 + CLI |
| C# (.NET 6+) | [`examples/command-pipe/csharp/DbPrismaPipeClient.cs`](../../examples/command-pipe/csharp/DbPrismaPipeClient.cs) | 跨平台客户端 + 演示程序 |
| Java (17+) | [`examples/command-pipe/java/DbPrismaPipeClient.java`](../../examples/command-pipe/java/DbPrismaPipeClient.java) | 跨平台客户端 + 演示程序 |

仓库还自带轻量测试客户端 [`tools/cmdpipe_client.py`](../../tools/cmdpipe_client.py) 与
[`tools/cmdpipe_client.ps1`](../../tools/cmdpipe_client.ps1)，可用于快速验证。

---

## 1. 总览：两条管道

DBPrisma 在**守护/服务模式**（Linux systemd / `--no-interactive`，Windows SCM / `--no-interactive`）下，
当 `command_pipe.enabled = true` 时暴露两条本机 IPC 管道：

| 管道 | 角色 | 生命周期 | 寻址 |
|------|------|----------|------|
| **控制管道** | 登录、执行管理子命令（`status`/`start`/`stop`…）、发起/停止抓取 | 服务运行期常驻 | 固定名（默认 `DBPrisma_ctrl`）|
| **capture 数据管道** | 流式推送实时 SQL 记录（NDJSON）| 每次 `capture` 临时创建，`capend`/断开即销毁 | 随机名 + 一次性 token，由控制管道返回 |

二者均与交互式 `XR:>` 提示符**完全隔离**，不影响其使用。

### 1.1 平台寻址

| 平台 | 控制管道地址 | capture 数据管道地址（示例）|
|------|--------------|------------------------------|
| **Windows** | `\\.\pipe\DBPrisma_ctrl` | `\\.\pipe\DBPrisma_cap_<随机十六进制>` |
| **Linux** | `<run_dir>/DBPrisma_ctrl.sock` | `<run_dir>/DBPrisma_cap_<随机十六进制>.sock` |

- Windows 为**命名管道**（字节模式 `PIPE_TYPE_BYTE`）。
- Linux 为 **AF_UNIX 流式套接字**（`SOCK_STREAM`），socket 文件权限 `0600`，仅属主可连。
- `<run_dir>` 即 `paths.run_dir`（默认 `<exe>/run/`）；capture 数据管道的**完整地址由控制管道在
  `capture` 响应里回传**，客户端无需自行拼接。

---

## 2. 控制管道协议

### 2.1 帧格式

控制管道上每个消息都是一帧：**4 字节小端（little-endian）无符号长度前缀** + **UTF-8 编码的 JSON 负载**。

```
+----------------+--------------------------------+
| len (uint32 LE)|  JSON payload (len bytes)      |
+----------------+--------------------------------+
```

- 长度仅指 JSON 字节数，不含 4 字节前缀本身。
- 单帧上限 **64 KiB**（`len` 为 0 或 > 65536 会被服务端判为协议错误并断开）。
- JSON 负载**不要求**以换行结尾。

### 2.2 请求与响应

所有响应都是一帧 JSON，统一含布尔字段 `ok`；失败时含 `error` 字符串。

#### (a) 登录 `login`

请求：
```json
{"op":"login","user":"admin","pass":"secret"}
```
成功响应（`token` 为本次会话令牌，后续所有 `cmd` 必须携带）：
```json
{"ok":true,"token":"<64 位十六进制>"}
```
失败响应（服务端会故意延迟约 300 ms 抑制爆破）：
```json
{"ok":false,"error":"authentication failed"}
```

#### (b) 执行命令 `cmd`

请求：
```json
{"op":"cmd","token":"<会话token>","line":"status"}
```
成功响应（`output` 为该命令打印到 stdout 的文本，含换行）：
```json
{"ok":true,"output":"...命令输出..."}
```
失败响应（`error` 取值见 §2.4）：
```json
{"ok":false,"error":"command not permitted"}
```

> `line` 即在 `XR:>` 下键入的整行命令；首个 token（动词）须在 `allowed_commands` 白名单内。
> **全部支持命令与参数**见 [命令管道命令参考手册](command-pipe-command-reference.md)。
> 白名单动词中仅 `capture` 有特殊语义（见 §3 与该参考手册 §4）；停止抓取用 §1A 的 `capend` op。

#### (b2) 管道原生 op（非 `cmd`）

仅管道、无 `XR:>` 对应：`ping`、`whoami`、`metrics`、`capstatus`、`capend`。完整请求/响应见 [命令管道命令参考手册 §1A](command-pipe-command-reference.md#1a-管道原生-op仅管道无-xr-对应)。

| op | 需要 token | 用途 |
|----|:----------:|------|
| `ping` | 否 | 探活：管道/代理是否在跑 |
| `whoami` | 是 | 当前 `admin_users.json` 登录名 |
| `metrics` | 是 | Prometheus 文本快照（同 `/metrics`） |
| `capstatus` | 是 | 流式 `capture` 是否在跑、channel、丢弃计数 |
| `capend` | 是 | 停止流式 `capture` 并销毁数据管道 |

#### (c) 注销 `logout` / `quit`

请求：
```json
{"op":"logout","token":"<会话token>"}
```
响应 `{"ok":true}` 后服务端关闭连接。`quit` 等价。

### 2.3 典型会话时序

```
client                          DBPrisma 控制管道
  | --- connect ----------------------> |
  | --- {op:login,user,pass} ---------> |
  | <-- {ok:true, token} -------------- |
  | --- {op:cmd, token, line:status} -> |
  | <-- {ok:true, output:"..."} ------- |
  | --- {op:logout, token} -----------> |
  | <-- {ok:true} --------------------- |
  | --- close ------------------------> |
```

### 2.4 错误字符串清单

| `error` | 触发条件 |
|---------|----------|
| `missing 'op'` | 帧里缺少 `op` 字段 |
| `unknown op` | `op` 非 login/cmd/logout/quit 及 §1A 管道原生 op |
| `authentication failed` | 用户名/密码校验失败 |
| `not authenticated` | 未登录就发 `cmd` |
| `invalid session token` | `cmd` 携带的 `token` 与会话不符 |
| `command not permitted` | 动词不在 `allowed_commands` 白名单 |
| `unknown command` | 动词不被 `dbp_exec_line` 识别 |
| `command execution failed` | 命令执行返回错误 |
| `server busy (max connections reached)` | 并发连接超过 `max_instances` |
| `capture already active` | 已有抓取会话在跑 |
| `service not running` | 代理未运行，无法 `capture` |
| `capture start failed` | 抓取数据通道/回调初始化失败 |

### 2.5 连接约束

- **空闲超时**：`session_idle_timeout_sec`（默认 300 s，0=禁用）。超时内无新帧，服务端关闭连接。
- **并发上限**：`max_instances`（默认 4，运行期钳制 1–64）。超限时新连接收到 `server busy` 后被关闭。
- **串行执行**：所有管理命令在服务端被全局命令锁串行化，多连接不会并发改状态。

---

## 3. capture 数据管道协议（流式 SQL 抓取）

`capture` 不通过控制管道回传数据，而是**另开一条专用数据管道**推送 NDJSON 流。

### 3.1 启动：在控制管道上发 `capture`

请求（过滤参数与交互式一致，`mode` 恒为屏幕流式、无需指定）：
```json
{"op":"cmd","token":"<会话token>","line":"capture -t DML,DQL -s 10.0.0.9:3306"}
```
成功响应：
```json
{"ok":true,"channel":"\\\\.\\pipe\\DBPrisma_cap_ab12cd34","token":"<一次性数据token>"}
```
- `channel`：**数据管道完整地址**（Windows 管道路径 / Linux socket 路径）。
- `token`：**一次性数据令牌**，连接数据管道后用于首行握手。

过滤参数：

| 参数 | 含义 |
|------|------|
| `-s ip[:port]` | 按**后端**（server）IP/端口过滤 |
| `-c ip[:port]` | 按**客户端**（client）IP/端口过滤 |
| `-t <类型>` | SQL 类型：`DML,DQL,DDL,DCL,TCL` 任意组合或 `ALL`（默认 `ALL`）|

### 3.2 握手：连接数据管道并回传 token

1. 连接 `channel`。
2. **首行**发送 `token` + `\n`（仅令牌，无 JSON）。
3. 服务端回一行 JSON：
   - 成功：`{"ok":true}\n`，随后开始推流。
   - 失败：`{"ok":false,"error":"auth"}\n`，随后断开。

> 握手用**常量时间比较**校验，且首行就必须正确，可防同机其它进程窃取明文 SQL 流。

### 3.3 数据流：NDJSON 逐行

握手成功后，服务端按行推送 **NDJSON**（每行一条 SQL 记录，以 `\n` 结尾），字段与
`capture.json` 逐字节一致：

| 字段 | 类型 | 说明 |
|------|------|------|
| `serial` | int | 本次会话内自增序号 |
| `ts` | string | ISO-8601 本地时间 |
| `latency_ms` | number | 语句耗时（毫秒）|
| `session_id` | int | 会话 ID |
| `server_id` | int | 后端 ID |
| `is_shadow` | bool | 是否影子流量 |
| `client` | object | `{"ip":"...","port":N}` |
| `server` | object | `{"ip":"...","port":N}` |
| `user` | string | 登录用户 |
| `db` | string | 数据库名 |
| `sql_type` | string | `DML`/`DQL`/`DDL`/`DCL`/`TCL` |
| `statement` | string | SQL 文本（若开启脱敏则字面量已被替换）|
| `sql_bytes` | int | SQL 字节数 |
| `result_code` | int | 执行结果码 |
| `mysql_errno` | uint | MySQL 错误号（0 表示无错）|
| `error_message` | string | 仅在出错时出现 |

示例行：
```json
{"serial":1,"ts":"2026-06-17T21:30:00","latency_ms":0.812,"session_id":7,"server_id":1,"is_shadow":false,"client":{"ip":"127.0.0.1","port":50512},"server":{"ip":"10.0.0.9","port":3306},"user":"app","db":"shop","sql_type":"DQL","statement":"SELECT * FROM orders WHERE id=?","sql_bytes":31,"result_code":0,"mysql_errno":0}
```

#### 背压与丢弃标记

服务端用**有界 FIFO**缓冲（上限 `capture_queue_mb`，默认 256 MB）。当消费端读取过慢导致积压
超限时，**丢弃最旧记录**（而非阻塞代理热路径），并向消费端插入一行标记告知累计丢弃数：
```json
{"_dropped":128}
```
客户端应识别该 `_dropped` 行（它不是 SQL 记录），用于监控/告警。

### 3.4 停止：在控制管道上发 `capend` op

```json
{"op":"capend","token":"<会话token>"}
```
响应 `{"ok":true}`。`capend` 会停止抓取并销毁数据管道。

- `capend` 是 **§1A 管道原生 op**，不出现在 `XR:>` 提示符，也不经 `allowed_commands` 白名单。
- 此外，以下任一情况也会自动结束抓取：客户端断开数据管道、执行 `stop`、进程退出。

### 3.5 capture 数据管道时序

```
control pipe                         data pipe
  | --{cmd: capture -t DML}-->          |
  | <--{ok, channel, token}--           |
  |                            connect-->|  (到 channel)
  |                          token+"\n"->|
  |                          <--{ok:true}\n
  |                          <--NDJSON 行 ...（持续）
  |                          <--{"_dropped":N}（按需）
  | --{op: capend, token}---->           |
  | <--{ok:true}-----------              | (数据管道关闭)
```

---

## 4. 鉴权与凭据（`admin_users.json`）

- 命令管道登录用户存放在 `command_pipe.auth_store`（默认 `config/admin_users.json`），
  **与 `proxy_users.json` 完全独立**——管道管理员**不是**代理用户，也不是后端 MySQL 用户。
- 推荐用命令化管理（密码自动 `ENC2:` 机器绑定 AES-256-GCM 加密落盘）：在 `XR:>` 下
  `useradd --admin <user>` / `setpwd --admin <user>` / `userdel --admin <user>` / `userlist --admin`。
- 客户端发送的是**明文密码**，由服务端在本机 IPC 上校验。请确保管道为 `local_only:true`（默认）。

---

## 5. 跨平台实现要点

| 关注点 | Windows | Linux |
|--------|---------|-------|
| 控制管道连接 | 打开命名管道 `\\.\pipe\<name>`（C#: `NamedPipeClientStream`；Java: `RandomAccessFile`；Python: `open(path,'r+b')`）| 连接 AF_UNIX 套接字 `<run_dir>/<name>.sock` |
| 数据管道连接 | 同上，地址取 `channel` | 连接 `channel`（socket 路径）|
| 字节序 | 长度前缀一律 **小端** | 同 |
| 读取 | 必须**读满** 4 字节再读 `len` 字节（IPC 可能分片）| 同 |
| NDJSON | 按 `\n` 切分，注意跨 read 边界的半行需缓存 | 同 |

> **务必“读满”**：命名管道/套接字的单次读可能返回不足请求的字节数，封装一个
> `readFully(n)` 循环读取。示例代码均已处理。

---

## 6. 运行示例

前置：在 DBPrisma 主机上启用并准备凭据（详见 [管理员手册 §4.4](admin-manual.md#44-命令管道远程管控command_pipe)）：

```text
# 1) 启用命令管道（改后须 restart / 重启服务）
DBPrisma setconfig command_pipe          # enabled=true

# 2) 准备管理员凭据（XR:> 下，密码自动 ENC2 加密）
DBPrisma                                  # 进入 XR:>，执行：useradd --admin admin

# 3) 以服务/守护模式启动（Linux systemd / Windows SCM / --no-interactive）
```

### 6.1 Python

```bash
cd examples/command-pipe/python
# Windows（命名管道）
python dbprisma_pipe.py --pipe DBPrisma_ctrl --user admin --pass secret status
# Linux（Unix socket）
python3 dbprisma_pipe.py --sock /opt/dbprisma/run/DBPrisma_ctrl.sock --user admin --pass secret status
# 流式抓取 30 秒
python3 dbprisma_pipe.py --sock /opt/dbprisma/run/DBPrisma_ctrl.sock --user admin --pass secret \
    capture --filter "-t DML,DQL" --seconds 30
```

### 6.2 C# (.NET 6+)

```bash
cd examples/command-pipe/csharp
dotnet run -- --user admin --pass secret status                       # Windows 默认连 DBPrisma_ctrl
dotnet run -- --sock /opt/dbprisma/run/DBPrisma_ctrl.sock --user admin --pass secret status
dotnet run -- --user admin --pass secret capture --filter "-t DML" --seconds 30
```

### 6.3 Java (17+)

```bash
cd examples/command-pipe/java
javac DbPrismaPipeClient.java
java DbPrismaPipeClient --user admin --pass secret status
java DbPrismaPipeClient --sock /opt/dbprisma/run/DBPrisma_ctrl.sock --user admin --pass secret status
java DbPrismaPipeClient --user admin --pass secret capture --filter "-t DML" --seconds 30
```

---

## 7. 常见问题（FAQ）

- **连不上 / 找不到管道**：确认服务以守护/服务模式运行、`command_pipe.enabled=true`、且已 `restart`。
  Windows 确认管道名一致；Linux 确认 socket 路径（`<run_dir>/<name>.sock`）与权限（属主、0600）。
- **`authentication failed`**：`admin_users.json` 不存在或无该用户。用 `useradd --admin` 创建。
- **`server busy`**：并发连接超过 `max_instances`，调大或复用连接。
- **`capture already active`**：同一时刻仅允许一个抓取会话，先 `capend`。
- **看到 `{"_dropped":N}`**：消费端处理太慢导致丢记录，提升消费速度或调大 `capture_queue_mb`。
- **半行 / 乱码**：未做“读满 + 按 `\n` 缓存切分”，参见 §5 与示例代码。

---

## 8. 版本

- r1（2026-06-17）：首版。覆盖控制管道帧协议、capture 数据管道握手与 NDJSON 流、`capture_queue_mb`/
  `capture_buf_kb` 缓冲配置、C#/Java/Python 客户端示例。
