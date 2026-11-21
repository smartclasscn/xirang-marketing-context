# DBPrisma 用户文档综合手册

> **说明**：本文档由开发团队用户文档合并而成（不含《配置参考手册 configuration-reference.md》，该文件因体量较大单独维护）。  
> **合并日期**：2026年7月  
> **适用版本**：DBPrisma ≥ 1.0

---

## 综合目录

| 部分 | 来源文档 | 受众 |
|------|----------|------|
| 第一篇 | 文档导航与使用指引 | 所有用户 |
| 第二篇 | 快速入门指南 | 所有用户 |
| 第三篇 | 管理员手册 | 系统管理员、DBA |
| 第四篇 | 配置部署指南 | 管理员 |
| 第五篇 | 运维手册 | 运维工程师、SRE |
| 第六篇 | 安全加固指南 | 安全工程师、管理员 |
| 第七篇 | SSL/TLS 行为矩阵 | 管理员、安全工程师 |
| 第八篇 | 透明代理方案 B | 架构师、管理员 |
| 第九篇 | 命令管道客户端开发手册 | 开发者、集成方 |
| 第十篇 | 命令管道命令参考手册 | 开发者、运维、集成方 |

> 配置项逐项说明请参阅同目录下 [configuration-reference.md](configuration-reference.md)。

---


# 第一篇 文档导航与使用指引

> **来源**：`README.md`

---


**版本**: v1.2 | **日期**: 2026-06-10 | **适用版本**: DBPrisma ≥ 1.0

---

## 文档导航

### 快速入门

| 文档 | 受众 | 说明 |
|------|------|------|
| [快速入门指南](quick-start.md) | 所有用户 | 5 分钟完成安装、配置和首次运行 |

---

### 核心手册

| 文档 | 受众 | 说明 |
|------|------|------|
| [管理员手册](admin-manual.md) | 系统管理员、DBA | 安装部署、配置管理、用户管理、证书管理、CLI 完整参考 |
| [运维手册](operations-manual.md) | 运维工程师、SRE | 日常检查、故障排查、变更操作、监控告警配置 |
| [安全加固指南](security-hardening-guide.md) | 安全工程师、管理员 | 密码加密、访问控制、SQL 防火墙、TLS 配置、合规检查清单 |

---

### 配置参考

| 文档 | 受众 | 说明 |
|------|------|------|
| [配置参考手册](configuration-reference.md) | 所有用户 | `conf.json` 全部 162 个配置项的完整说明、类型、默认值和约束 |
| [配置部署指南](configuration-guide.md) | 管理员 | 生产部署场景配置示例（读写分离、主从、TLS 等） |

---

### 专题指南

| 文档 | 受众 | 说明 |
|------|------|------|
| [SSL/TLS 行为矩阵](ssl-behavior-matrix.md) | 管理员、安全工程师 | P1/P2 TLS 组合配置行为详细说明 |
| [透明代理方案 B](scheme-b-transparent.md) | 架构师、管理员 | 透明代理（无感知接入）部署方案 |
| [命令管道客户端开发手册](command-pipe-client-guide.md) | 开发者、集成方 | C#/Java/Python 对接命令管道与 capture 数据通道的协议规范与示例 |
| [命令管道命令参考手册](command-pipe-command-reference.md) | 开发者、运维、集成方 | **管道支持的全部命令与参数**（白名单、§1A 管道原生 op、语法、硬性禁止、`capture`） |

---

## 按使用场景快速定位

### 我是首次部署 DBPrisma

1. [快速入门指南](quick-start.md) → 基础安装和验证
2. [管理员手册 §2](admin-manual.md#2-安装与部署) → 生产级目录规划和权限配置
3. [配置参考手册](configuration-reference.md) → 配置所有选项
4. [安全加固指南 §11](security-hardening-guide.md#11-生产环境安全检查清单) → 上线前安全检查

### 我要配置读写分离

→ [配置部署指南](configuration-guide.md) 中的"读写分离"章节

### 我要配置 TLS

→ [管理员手册 §6](admin-manual.md#6-tls-证书管理) 和 [SSL/TLS 行为矩阵](ssl-behavior-matrix.md)

### 我要加固安全

→ [安全加固指南](security-hardening-guide.md)（全文）

### 我遇到故障，需要排查

→ [运维手册 §8](operations-manual.md#8-故障排查手册) 故障排查手册

### 我需要紧急操作

→ [运维手册 §9](operations-manual.md#9-紧急操作手册soe) 紧急操作手册  
→ [技术 Runbook §7A](../technical/09-operations-runbook.md#7a-事件响应剧本-playbooks) 事件响应剧本（1148 / 1129 / gate 137）

### SQL 防火墙误拦 / 紧急回滚

→ [安全加固指南 §5.6](security-hardening-guide.md#56-紧急回滚误拦生产-sql)  
→ [迁移指南 §7A](../technical/18-migration-upgrade-guide.md#7a-关闭或回滚-sql-防火墙)

### 我要查某个配置项的含义

→ [配置参考手册](configuration-reference.md)（按 Ctrl+F 搜索字段名）

### 我要用代码对接命令管道（C#/Java/Python）

→ [命令管道命令参考手册](command-pipe-command-reference.md)（**全部命令与参数**）  
→ [命令管道客户端开发手册](command-pipe-client-guide.md) + 示例目录 [`examples/command-pipe/`](../../examples/command-pipe/)

---

## 文档版本历史

| 文档 | 当前版本 | 最后更新 |
|------|---------|---------|
| 快速入门指南 | v1.1 | 2026-06-10 |
| 管理员手册 | v1.1 | 2026-06-10 |
| 运维手册 | v1.2 | 2026-06-10 |
| 安全加固指南 | v1.1 | 2026-06-06 |
| 配置参考手册 | r14 | 2026-06-17 |
| 配置部署指南 | — | 2026-06-05 |
| SSL/TLS 行为矩阵 | — | — |
| 透明代理方案 B | — | — |
| 命令管道客户端开发手册 | r1 | 2026-06-17 |

---

## 相关技术文档

技术设计文档位于 [`docs/technical/`](../technical/README.md)（全集 v2.3+）：

- [架构设计](../technical/02-architecture-design.md)
- [功能规格 ERR 矩阵](../technical/16-functional-spec-behavior.md#8a-mysql-err-包矩阵客户端可见)
- [运维 Runbook §7A 事件剧本](../technical/09-operations-runbook.md#7a-事件响应剧本-playbooks)
- [ADR 决策记录](../technical/15-adr-collection.md)
- [C 编码规范](../technical/23-c-coding-standard.md)
- [技术术语表](../technical/22-project-technical-glossary.md)

**修订**：v1.3（2026-06-17）— 新增[命令管道客户端开发手册](command-pipe-client-guide.md)（C#/Java/Python 对接控制管道与 capture 数据通道）及 [`examples/command-pipe/`](../../examples/command-pipe/) 示例；命令管道新增 `capture_queue_mb`/`capture_buf_kb` 缓冲配置（配置参考升至 r14）。  
v1.2（2026-06-10）— 配置项计数统一为 159；版本历史表对齐各文档实际版本（配置参考 r8、运维 v1.2、管理员/快速入门 v1.1）；修正 `start` 子命令须置于选项之前的示例（C-014）。  
v1.1（2026-06-06）— 紧急操作链至 Runbook §7A；修正技术文档路径；用户手册 v1.1 对齐 restart 语义。


---

# 第二篇 快速入门指南

> **来源**：`quick-start.md`

---


**版本**: v1.1 | **日期**: 2026-06-10 | **适用版本**: DBPrisma ≥ 1.0

---

## 概述

本指南帮助你在 5 分钟内完成 DBPrisma MySQL 代理的安装、配置和首次运行。DBPrisma 支持 Windows 和 Linux 双平台，输出 DLL/SO 形式的共享库及配套命令行程序。

---

## 前提条件

| 条件 | 说明 |
|------|------|
| MySQL 后端 | 至少一个可访问的 MySQL 5.7+ 或 8.0+ 实例 |
| 操作系统 | Windows 10+（x64）或 Linux（glibc ≥ 2.17，x64） |
| 权限 | MySQL 后端账号须有 `SELECT, INSERT, UPDATE, DELETE` 权限 |
| 端口 | 确认监听端口（默认 `3307`）未被占用 |
| 可选 | CMake 3.16+ 和 C 编译器（仅自行构建时需要） |

---

## 第一步：获取 DBPrisma

### 使用预编译包（推荐）

```
DBPrisma/
├── DBPrisma.exe / DBPrisma          # 主程序
├── PrismaAgent.dll / libPrismaAgent.so
├── config/
│   ├── conf.json                    # 主配置
│   └── proxy_users.json             # 代理用户
├── run/                             # lock、pid、cli_token（运行时创建）
├── logs/                            # 主日志（运行时创建）
└── capture/                         # capture.json（运行时创建）
```

### 自行构建

```bash
# Linux
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make -j$(nproc)

# Windows (PowerShell / Developer Command Prompt)
mkdir build && cd build
cmake .. -G "Visual Studio 17 2022"
cmake --build . --config Release
```

构建产物位于 `build/Release/`（Windows）或 `build/`（Linux）。

---

## 第二步：创建最小配置

在 `<exe>/config/` 创建 `conf.json`（或复制模板后修改）：

```json
{
  "version": "0.0.1",
  "proxy": {
    "worker_threads": 4,
    "max_clients": 1000,
    "admin_listen_port": 8080,
    "admin_bearer_token": "change-me-in-production"
  },
  "servers": [
    {
      "id": 1,
      "scheme": "auth",
      "host": "127.0.0.1",
      "port": 3306,
      "listen_ip": "0.0.0.0",
      "listen_port": 3307,
      "enabled": true,
      "user": "dbprisma",
      "password": "yourpassword"
    }
  ],
  "logging": {
    "file": "DBPrisma.log",
    "level": "info",
    "max_size_mb": 100,
    "rotate_count": 7
  },
  "paths": {
    "run_dir": "run",
    "log_dir": "logs",
    "capture_dir": "capture"
  }
}
```

> `logging.file` 为相对路径时锚定 `log_dir`（默认 `<exe>/logs/`）。四目录可通过 `paths` 段、`--config-dir/--run-dir/--log-dir/--capture-dir` 或 `DBPRISMA_HOME` 覆盖。

> **关键字段说明**
> - `servers[].host` / `port`：MySQL 后端地址
> - `servers[].listen_port`：代理监听端口（客户端连接此端口）
> - `proxy.admin_bearer_token`：Admin HTTP API 鉴权令牌，**生产环境必须修改**

---

## 第三步：创建代理用户

DBPrisma 维护独立的代理用户列表，与 MySQL 后端账号相互独立。`useradd` / `userlist` 等用户管理命令是 **XR:> 提示符下的运行时命令**，不是独立的 Shell 子命令——先运行 `./DBPrisma` 进入 XR:> 提示符再执行：

```text
./DBPrisma                 # 进入 XR:> 提示符（交互模式）

XR:> useradd appuser       # 添加代理用户（交互式密码提示，不回显，不接受命令行密码参数）
XR:> userlist              # 查看用户列表
```

用户信息存储在 `config/proxy_users.json`。

---

## 第四步：验证配置

```bash
./DBPrisma checkconfig
```

正常输出：
```
[OK] version = 0.0.1 (engine 0.0.1.568)
[OK] servers[0]: primary -> 127.0.0.1:3306  listen 0.0.0.0:3307
[OK] Configuration valid.
```

---

## 第五步：启动代理

```bash
# 前台启动（调试/初次验证用）
./DBPrisma start

# 无交互式提示符启动（适合后台/systemd/脚本部署）
# 注意：start 子命令的选项可前可后；而 checkconfig/setconfig/service 须作为第一个参数
./DBPrisma start --no-interactive

# Windows 注册为系统服务
./DBPrisma service install
./DBPrisma service start
```

启动成功后控制台（或日志）输出：
```
[license] TRIAL - no license found - trial started (30 days).
[INFO] DBPrisma started. Listening on 0.0.0.0:3307
[INFO] Admin HTTP on 127.0.0.1:8080
```

> **关于授权**：DBPrisma 启动受 License 闸门控制。
> - 未放置 `license.dat` 时，自首次运行起**默认 30 天试用**；试用到期且无 license 将拒绝启动。
> - 正式授权需把厂商签发的 `license.dat` 放入 **config 目录**（与 `conf.json` 同级）。
> - 申请前可用 `licreq --print-fingerprint` 查看本机机器指纹；生成后可用 `licreq --show license.request` 核对（详见 [licreq README](../../tools/licreq/README.md)）。
> - 若看到 `[license] Startup blocked: ...` 则表示启动被拦截，详见 [运维手册 §13](operations-manual.md#13-授权与-license-管理)。

---

## 第六步：测试连接

使用任意 MySQL 客户端通过代理端口连接：

```bash
# mysql 命令行
mysql -h 127.0.0.1 -P 3307 -u appuser -p

# Python（pymysql）
import pymysql
conn = pymysql.connect(host='127.0.0.1', port=3307,
                       user='appuser', password='AppPass123',
                       database='yourdb')
```

---

## 第七步：验证代理状态

`status` / `conns` / `backends` 是 XR:> 提示符下的运行时命令，在已运行的实例中执行：

```text
XR:> status        # 查看代理运行状态
XR:> conns         # 查看当前活跃连接
XR:> backends      # 查看后端健康状态
```

Admin HTTP 接口：
```bash
# 健康检查
curl -H "Authorization: Bearer change-me-in-production" http://127.0.0.1:8080/health

# Prometheus 指标
curl -H "Authorization: Bearer change-me-in-production" http://127.0.0.1:8080/metrics
```

---

## 常见问题

| 现象 | 原因 | 解决方法 |
|------|------|----------|
| `Connection refused` on 3307 | 代理未启动或端口被占用 | XR:> 提示符下 `status`；检查端口 `netstat -an \| grep 3307` |
| `Access denied for user 'appuser'` | 代理用户不存在 | XR:> 提示符下 `userlist`，若无则 `useradd` |
| `checkconfig` 报错 | conf.json 格式错误 | 用 JSON 验证器检查；参考 [配置参考手册](configuration-reference.md) |
| 后端连接失败 | MySQL 地址/端口/账号错误 | 直连后端验证；检查防火墙规则 |
| 启动后立即退出 | 缺少 `logs/` 目录 | `mkdir logs` |

---

## 下一步

| 目标 | 文档 |
|------|------|
| 完整配置所有选项 | [配置参考手册](configuration-reference.md) |
| 生产环境部署 | [管理员手册](admin-manual.md) |
| 日常运维操作 | [运维手册](operations-manual.md) |
| 安全加固 | [安全加固指南](security-hardening-guide.md) |
| 读写分离/主从配置 | [配置部署指南](configuration-guide.md) |


---

# 第三篇 管理员手册

> **来源**：`admin-manual.md`

---


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


---

# 第四篇 配置部署指南

> **来源**：`configuration-guide.md`

---


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


---

# 第五篇 运维手册

> **来源**：`operations-manual.md`

---


**版本**: v1.2 | **日期**: 2026-06-10 | **适用版本**: DBPrisma ≥ 1.0  
**受众**: 系统运维工程师、值班 SRE、NOC 工程师

---

## ⚠️ CLI 调用模型（重要，请先阅读）

DBPrisma 的命令分为两类，混淆将导致操作无效甚至误操作：

| 类型 | 可直接在 Shell 中单次运行的命令 |
|------|---------------------------------|
| **独立子命令**（Shell 直接执行） | `start`、`checkconfig`（别名 `cc`）、`setconfig`（别名 `sc`）、`service`、`--help`、`--version`；其中 `checkconfig`/`setconfig`/`service` 必须为第一个参数（位于选项之前），`start` 选项可前可后 |
| **交互会话命令**（仅限 XR:> 提示符内） | `status`、`stop`、`reload`、`restart`、`shutdown`、`backends`、`conns`、`kick`、`drain`、`pause`、`resume`、`flush`、`capture`、`display`、`loglevel`、`history`、`useradd`、`userdel`、`userlist`、`setpwd`、`token rotate` 等所有其他命令 |

**关键说明**：
- 运行 `./DBPrisma status` 会报 `unknown command 'status'`（`status` 既非子命令、又非存在的配置文件），**不会**显示状态。
- 运行 `./DBPrisma stop` 同理报 `unknown command 'stop'`，**不会**停止服务。
- 所有操作命令必须在 **XR:> 交互会话**中执行（通过 `./DBPrisma` 启动交互模式；守护进程模式请用 `./DBPrisma start --no-interactive`，注意必须带 `start`，否则代理不会启动且进程随即退出）。
- 本手册中的 `XR:> <cmd>` 格式表示在交互提示符下输入的命令；`./DBPrisma <cmd>` 格式仅用于独立子命令。
- `DBPrisma stop` 和 `DBPrisma reload` 也**不是**有效的独立 Shell 命令，不能在 systemd `ExecStop=`/`ExecReload=` 中使用。

---

## 目录

1. [日常检查清单](#1-日常检查清单)
2. [启动、停止、重启操作](#2-启动停止重启操作)
3. [热更配置（不停服）](#3-热更配置不停服)
4. [连接管理操作](#4-连接管理操作)
5. [监控与状态查询](#5-监控与状态查询)
6. [日志管理](#6-日志管理)
7. [后端数据库运维协同](#7-后端数据库运维协同)
8. [故障排查手册](#8-故障排查手册)
9. [紧急操作手册（SOE）](#9-紧急操作手册soe)
10. [变更操作规程](#10-变更操作规程)
11. [监控告警配置参考](#11-监控告警配置参考)
12. [运维命令速查表](#12-运维命令速查表)
13. [授权与 License 管理](#13-授权与-license-管理)

---

## 1. 日常检查清单

### 1.1 每日检查（建议每天 9:00 执行）

```bash
# 4. 检查日志中是否有 ERROR/WARN
grep -E "^.{23}(ERROR|WARN)" logs/dbprisma.log | tail -50

# 5. 检查 Admin HTTP 可达性
curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health | python -m json.tool

# 6. 检查磁盘空间（日志磁盘）
df -h /opt/dbprisma/logs
```

以下命令须在 **XR:> 会话**中执行（启动 `./DBPrisma` 后输入）：

```
# 1. 确认代理运行状态
XR:> status

# 2. 查看后端健康状态
XR:> backends

# 3. 检查活跃连接数是否异常
XR:> conns
```

### 1.2 每周检查

- 检查日志文件轮转是否正常（`logs/` 目录文件数和大小）
- 检查 `backups/` 配置备份文件数量，清理过期备份
- 复查 Prometheus `/metrics` 中 `connection_errors_total` 趋势
- 验证 TLS 证书有效期（距到期 < 30 天需提前轮换）

### 1.3 每月检查

- 轮换 Admin HTTP Bearer Token：在 XR:> 会话中执行 `token rotate`
- 核查代理用户列表是否有废弃账号：在 XR:> 会话中执行 `userlist`
- 核查 IP 黑白名单是否需要更新
- 对照 `docs/user/configuration-reference.md` 核查运行配置

---

## 2. 启动、停止、重启操作

### 2.1 正常启动

```bash
# 前台启动（进入 XR:> 提示符）
./DBPrisma

# 守护进程模式（无交互式提示符；SIGTERM 或 XR:> stop 命令退出）
./DBPrisma start --no-interactive --config config/conf.json

# systemd（推荐生产部署）
systemctl start dbprisma

# Windows 服务
.\DBPrisma.exe service start
# 或
Start-Service DBPrisma
```

> **授权前置**：启动时会先经过 License 闸门，控制台/日志会打印 `[license] ...` 一行：
> - `[license] licensed to ...` 或 `[license] TRIAL - ...` → 放行，继续启动；
> - `[license] Startup blocked: ...` + `Startup aborted by license gate.` → **不会启动**。
> 详见 [§13 授权与 License 管理](#13-授权与-license-管理)。

**启动后验证**（在 XR:> 会话中执行）：
```
XR:> status          # 期望输出: RUNNING
XR:> backends        # 期望后端状态: UP
```

或通过 Admin HTTP API：
```bash
curl http://127.0.0.1:8080/health -H "Authorization: Bearer $TOKEN"
```

### 2.2 优雅停止

在 XR:> 会话中执行：
```
# 优雅停止（等待连接处理完，最长 graceful_drain_ms 毫秒）
XR:> stop
```

或通过 OS 服务管理器：
```bash
# systemd（发送 SIGTERM，触发优雅关闭）
systemctl stop dbprisma

# Windows 服务
.\DBPrisma.exe service stop
```

优雅停止流程：
1. 停止接受新连接
2. 等待现有连接完成当前事务（最长 `txn_drain_ms` ms）
3. 强制关闭剩余连接（最长 `stop_deadline_ms` ms）
4. 清理资源退出

### 2.3 优雅重启（零停机滚动重启）

在 XR:> 会话中执行：
```
# 方式1：直接重启命令（stop + 预检 + 预热启动）
XR:> restart
```

> **注意**：`systemctl reload dbprisma` **无效**——代理不响应 SIGHUP 作为重新加载信号（systemd 单元不含 `ExecReload=`）。热更配置请用 `XR:> reload`，进程级重启请用 `XR:> restart` 或 systemctl restart。

配合负载均衡的滚动重启（在 XR:> 会话中）：
```
# 步骤1：标记指定后端为排水（停止接受新连接）
XR:> drain <server_id>

# 步骤2：等待 conns 降为 0 或业务低峰后停止
XR:> stop
```
```bash
# 步骤3：替换可执行文件或修改配置，然后重新启动
./DBPrisma start --no-interactive --config config/conf.json
```

### 2.4 强制停止（紧急）

在 XR:> 会话中执行：
```
# 优雅停止（带倒计时，默认等待现有连接完成事务）
XR:> shutdown

# 立即强制停止（跳过排水等待）
XR:> shutdown 0 force
```

```bash
# 或直接 kill（最后手段）
kill -15 $(cat /var/run/dbprisma.pid)
```

> **注意**：`shutdown` 默认是**优雅停止**（等待连接排空，受 `graceful_drain_ms` / `stop_deadline_ms` 控制）。`shutdown 0 force` 才是立即强停，可能导致正在执行的事务回滚，生产环境仅在紧急情况使用。

---

## 3. 热更配置（不停服）

### 3.1 单项配置热更

在 XR:> 会话中执行：
```
# 运行时动态修改日志级别（立即生效，无需停服）
XR:> loglevel DEBUG
XR:> loglevel INFO     # 恢复
```

修改安全配置（`security.*` 不支持热更，须停服 → 修改文件 → 重启）：
```
# 步骤1：在 XR:> 会话中停止服务
XR:> stop
```
```bash
# 步骤2：手动编辑 conf.json 的 ip_denylist / sql_firewall_enabled 等字段
# 或使用交互式向导（会校验配置）：
./DBPrisma setconfig security
# 步骤3：重新启动
./DBPrisma start --no-interactive --config config/conf.json
```

### 3.2 整体配置热更

```bash
# 步骤1：备份当前配置
cp conf/conf.json conf/conf.json.manual-$(date +%Y%m%d-%H%M%S)

# 步骤2：修改 conf.json

# 步骤3：验证配置（独立子命令，可直接在 Shell 中运行）
./DBPrisma checkconfig
```

在 XR:> 会话中执行：
```
# 步骤4：热更（仅对可热更字段生效，不可热更字段须 stop → start）
XR:> reload
```

**热更后验证**：
```bash
# 查看当前磁盘配置（独立子命令）
./DBPrisma setconfig show logging
./DBPrisma setconfig show proxy
```

在 XR:> 会话中执行：
```
# 查看运行时状态
XR:> status
```

### 3.3 热更注意事项

- 修改 `proxy.worker_threads`、`servers[].listen_port`、`tls.*` 等字段需要重启才能生效
- `reload` 命令会跳过需要重启的字段（显示 `SKIP: RESTART_REQUIRED`），不会报错
- 若 `config_manage.rollback_on_reload_fail = true`，热更失败会自动回滚

---

## 4. 连接管理操作

### 4.1 查看连接详情

在 XR:> 会话中执行：
```
# 显示所有活跃连接
XR:> conns

# 仅显示某客户端 IP 的连接（可选过滤）
XR:> conns 192.168.1.100

# 输出示例
# ID    CLIENT_IP       USER        DB       STATE     DURATION
# 1001  192.168.1.100   appuser     mydb     EXECUTING  0.5s
# 1002  192.168.1.101   readuser    mydb     IDLE        120s
```

### 4.2 踢出问题连接

`kick` 命令可按客户端 IP，或按 `conns` 列表行号强制断开连接。在 XR:> 会话中执行：
```
# 断开指定 IP 的所有连接（port=0 表示所有端口）
XR:> kick 192.168.1.100

# 断开指定 IP 的指定端口
XR:> kick 192.168.1.100 54321

# 按 conns 列表的第 N 行（# 列）断开单条连接
XR:> conns          # 先查看，记下目标行的 # 序号
XR:> kick id 3      # 断开第 3 行连接（实时解析为其 IP:端口）
```

> **注意**：`kick` 不接受 `--user` / `--ip` 标志。参数为客户端 IP（可选端口），或 `id <n>`。`id <n>` 的 N 取自**最近一次** `conns` 输出；列表变动后需重新 `conns` 确认。

### 4.3 暂停/恢复接受连接

在 XR:> 会话中执行：
```
# 暂停接受新连接（不断开现有连接）
XR:> pause
# 场景：后端维护前暂停，避免新连接建立

# 恢复接受新连接
XR:> resume
```

### 4.4 排水（Drain）操作

在 XR:> 会话中执行：
```
# 标记指定后端为排水（停止向其路由新连接，现有连接自然结束）
XR:> drain <server_id>
# 场景：滚动升级、计划维护；server_id 为 servers[].id 的值

# 取消排水，恢复接受新连接
XR:> drain cancel <server_id>
```

```bash
# 监控 drain 进度（通过 Admin HTTP）
watch -n 2 'curl -s -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health | python -m json.tool'
```

---

## 5. 监控与状态查询

### 5.1 代理整体状态

在 XR:> 会话中执行：
```
XR:> status

# 期望输出示例
# Status:       RUNNING
# Uptime:       3d 2h 15m
# Connections:  Active=42  Total=152340
# Workers:      8 threads
# Memory:       RSS=128MB
```

### 5.2 后端健康状态

在 XR:> 会话中执行：
```
XR:> backends

# 输出示例
# ID         HOST              PORT   STATUS  WEIGHT  POOL  LATENCY
# primary    mysql-primary     3306   UP      100     10/50  1.2ms
# replica1   mysql-replica1    3306   UP      50      5/20   2.1ms
# replica2   mysql-replica2    3306   DOWN    50      0/0    -
```

后端状态说明：
- `UP`：健康，正常接受请求
- `DOWN`：不可用，探测失败超过阈值
- `HALF_OPEN`：熔断恢复探测中
- `DISABLED`：管理员手动禁用（`servers[].enabled = false`）

### 5.3 Admin HTTP 指标

```bash
# 获取 Prometheus 格式指标
curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/metrics

# 关键指标
# dbprisma_connections_active         当前活跃连接数
# dbprisma_connections_total          历史总连接数
# dbprisma_backend_status{id="..."}   后端状态（1=UP, 0=DOWN）
# dbprisma_backend_latency_ms{id="..."} 后端延迟
# dbprisma_query_duration_seconds     查询耗时分布
# dbprisma_errors_total               错误总计数
# dbprisma_cb_state{id="..."}         熔断器状态
```

### 5.4 状态一次性显示

在 XR:> 会话中执行：
```
# 一次性读取 conf.json 和代理用户列表并显示（非持续刷新）
XR:> display
```

> **注意**：`display` 是**一次性读取**磁盘上的 `conf.json` + 代理用户列表并输出，不会持续刷新。

---

## 6. 日志管理

### 6.1 日志级别

| 级别 | 适用场景 |
|------|----------|
| `DEBUG` | 开发调试、问题定位（**勿在生产长期开启，性能影响大**） |
| `INFO` | 正常生产运行（推荐） |
| `WARN` | 需关注的异常，但不影响服务 |
| `ERROR` | 影响服务的错误，需立即处理 |

在 XR:> 会话中执行：
```
# 在线切换日志级别（立即生效，无需重启）
XR:> loglevel DEBUG
XR:> loglevel INFO
XR:> loglevel WARN
```

### 6.2 日志文件管理

配置参考：
```json
"logging": {
  "file": "logs/dbprisma.log",
  "level": "INFO",
  "max_size_mb": 100,
  "rotate_count": 7
}
```

- `max_size_mb`：单个日志文件最大大小，超过后轮转
- `rotate_count`：保留的历史日志文件数量

在 XR:> 会话中执行：
```
# 刷新后端预热连接池（重建 warmup pool；省略 server_id 则刷新全部）
XR:> flush [server_id]
```

```bash
# 查看最新日志
tail -f logs/dbprisma.log

# 过滤错误日志
grep "ERROR" logs/dbprisma.log | tail -100

# 查看慢查询日志（若 log_config.slow_query_file 已配置）
tail -f logs/slow_query.log
```

### 6.3 SQL 流量捕获（调试用）

`capture` 命令进入**阻塞捕获模式**（当前窗口显示实时 SQL 流）。在 XR:> 会话中执行：
```
# 进入实时 SQL 捕获（阻塞模式；P=暂停/恢复打印，E=退出捕获）
XR:> capture [screen|json|all]

# 按后端过滤
XR:> capture -s 192.168.1.10:3306

# 按客户端过滤
XR:> capture -c 10.0.0.5:0

# 按 SQL 类型过滤（DDL,DML,DQL,DCL,TCL）
XR:> capture -t DML,DDL
```

> **注意**：
> - `capture` **没有** `start`/`stop` 子命令；进入后按 **P** 暂停/恢复，按 **E** 退出捕获模式。
> - `capture_redact_literals = true` 时，SQL 参数值会被 `?` 替换（脱敏）。
> - capture 开启时会降低性能，仅用于短期问题定位。

---

## 7. 后端数据库运维协同

### 7.1 后端计划维护（不停代理服务）

在 XR:> 会话中执行后，再执行独立命令：
```
# 步骤1：停止服务
XR:> stop
```
```bash
# 步骤1（续）：通过 setconfig server <id> 向导禁用该后端
./DBPrisma setconfig server 1     # 交互式向导，将 enabled 改为 false
# 重新启动
./DBPrisma start --no-interactive --config config/conf.json
```
```
# 步骤2：等待该后端连接池清空（在 XR:> 会话中查看）
XR:> backends    # 查看 POOL 列变为 0
```

```
# 步骤3：对该 MySQL 实例做维护操作（略）

# 步骤4：维护完成后重新启用
XR:> stop
```
```bash
./DBPrisma setconfig server 1     # 交互式向导，将 enabled 改回 true
./DBPrisma start --no-interactive --config config/conf.json
```
```
# 步骤5：验证后端恢复
XR:> backends
```

### 7.2 后端主从切换

> 注：`auto_failover`（自动故障转移）当前为**计划中/未实现**（TD-012 清查的保留键，仅解析与 `checkconfig` 校验，引擎侧不切换）。主库故障请按下述**手动切换**流程处理。

主库故障时手动切换：

在 XR:> 会话中执行：
```
# 查看当前路由状态
XR:> backends

# 若自动故障转移未触发，手动将新主库标记为 primary
# 方法：修改 conf.json 中 servers 的 primary 字段后 reload
XR:> reload
```

### 7.3 后端探测与熔断

在 XR:> 会话中执行：
```
# 查看熔断器状态
XR:> status    # 输出中包含 CB 状态

# 后端状态探测配置
# resilience.probe_interval_sec：健康探测间隔
# resilience.probe_failure_threshold：连续失败多少次触发熔断
# resilience.probe_recovery_threshold：连续成功多少次关闭熔断
# circuit_breaker.error_rate_threshold_pct：错误率阈值
```

### 7.4 防止后端 MySQL 封禁代理 IP（`max_connect_errors` / Error 1129）

> **背景**：MySQL 会按**来源 IP** 累计「连接建立后、认证完成前」的失败次数（`max_connect_errors`，默认 100）。超过阈值后，该 IP 被封锁，返回：
>
> `Error 1129: Host 'x.x.x.x' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'`
>
> DBPrisma 连后端时，所有连接对 MySQL 而言都来自**代理机同一 IP**。压测、错误凭据探测、频繁重启/暖机失败等场景容易在代理 IP 上累积计数，导致**整台代理对后端不可用**。
>
> 注意：这与 DBPrisma 自身对客户端的 IP 封禁（`security.auth_fail_max` → 客户端见 1129）是**两套机制**，勿混淆。

> **代理侧已根治（无需额外配置）**：自 2026-06 起，代理对后端的两个「认证完成前断开」来源均已在代码层消除——(1) 认证代理（scheme A）的健康检查不再用「裸 TCP 连接后立即关闭」的探测（改由已认证 `COM_PING`/真实业务连接近期上报即跳过裸探测）；(2) **裸 TCP 预热池彻底退役**（曾对后端建立未认证连接后丢弃，是 `ERROR 1129` 的主要残留源，尤其 scheme B）——现 `warmup_tcp_connections` 已弃用，方案 A+明文后端折算为认证预热，其余场景用冷连接。因此代理**不再向后端产生「认证前断开」的连接错误**。下列后端配置作为**纵深防御**仍然推荐。

**按方案选择（先看这里）：**

| 方案 | 代理是否持有后端账号 | 首选手段 | 说明 |
|------|----------------------|----------|------|
| **scheme A（认证代理）** | 是（`conf.json` `servers[].user`） | 代理已根治 + 方向一/方向三 | 用固定后端专用账号，可精准授予 `CONNECTION_ADMIN`（方向三） |
| **scheme B（透明代理）** | **否**（客户端凭据直传后端） | **方向二（`host_cache_size=0`）** | 代理无凭据，无法做认证级探测；`CONNECTION_ADMIN` **不适用**（每个客户端账号各异），只能在后端关闭封禁机制 |

#### 7.4.1 方向一：调大后端 `max_connect_errors`（推荐，所有 MySQL 版本）

在**每台后端 MySQL** 上调高阈值，降低误封概率。生产环境建议设为较大值（如 `1000000`），并在变更单中记录。

**临时生效（重启 MySQL 后失效）：**

```sql
-- 在 MySQL 上执行（需 SUPER 或 SYSTEM_VARIABLES_ADMIN）
SET GLOBAL max_connect_errors = 1000000;

-- 验证
SHOW GLOBAL VARIABLES LIKE 'max_connect_errors';
```

**永久生效（`my.cnf` / `my.ini`，改后重启 MySQL）：**

```ini
[mysqld]
max_connect_errors = 1000000
```

**已封禁时的应急解封：**

```bash
# 在能连上该 MySQL 的机器上执行（通常从 DBA 跳板机，而非已被封的代理机）
mysqladmin -h <backend_host> -P <port> -u <admin_user> -p flush-hosts
```

> **运维建议**：将 `max_connect_errors` 纳入后端 MySQL 基线配置；代理上线前在每台后端核查一次。压测/长 soak 前从**未被封禁的跳板机**执行 `flush-hosts`，勿依赖代理机自身执行（代理 IP 被封时 `mysqladmin` 也会失败）。

#### 7.4.2 方向二：禁用后端 host 缓存（`host_cache_size=0`，**scheme B 首选 / 最彻底**）

后端封禁计数保存在 MySQL 的 **host 缓存**里；把缓存容量设为 `0` 会**关闭 host 缓存，连带禁用 host 封禁机制**——`Error 1129 (ER_HOST_IS_BLOCKED)` 从此**永不再发**，无论代理产生多少「认证前断开」。这正是 **MySQL 官方推荐的「前置代理 / 负载均衡」场景**配置：所有连接都来自代理同一 IP，按 IP 封禁本就无意义且有害（一个坏客户端会连累全部代理流量）。

同时建议开启 `skip_name_resolve`，免去反向 DNS 解析失败造成的连接错误，并加快连接建立。

**临时生效（运行时即时生效，重启 MySQL 后失效）：**

```sql
-- 需 SUPER 或 SYSTEM_VARIABLES_ADMIN
SET GLOBAL host_cache_size = 0;

-- 验证（应为 0）
SHOW GLOBAL VARIABLES LIKE 'host_cache_size';
```

> `skip_name_resolve` 为只读变量，**不能**用 `SET GLOBAL` 在线修改，只能写 `my.cnf` 后重启。

**永久生效（`my.cnf` / `my.ini`，改后重启 MySQL）：**

```ini
[mysqld]
host_cache_size  = 0
skip_name_resolve = ON
```

> **适用性**：所有 MySQL 版本（5.7 / 8.0 / 8.4 / 9.x）均支持。对 **scheme B（透明代理）是首选**，因为代理不持后端账号、无法走方向三；对 scheme A 亦可作为方向一/三之外的兜底，三者不冲突。
> **权衡**：禁用 host 缓存后，后端不再缓存主机名解析与 per-host 统计；在「代理前置」架构下这恰好是期望行为（无副作用）。若后端**同时**还直接面向大量非代理客户端，则评估后再决定（可改用方向一把 `max_connect_errors` 调到极大值替代）。

#### 7.4.3 方向三：授予后端账号 `CONNECTION_ADMIN`（MySQL 8.0.26+，**仅 scheme A**）

MySQL 8.0.26 起，持有 `CONNECTION_ADMIN` 全局权限的账号，其连接**不受** `max_connect_errors` 限制。适合将 DBPrisma 在 `conf.json` 中配置的**后端专用账号**设为豁免，与方向一可叠加使用。

**前提**：后端 MySQL ≥ 8.0.26；`conf.json` 中 `servers[].user` 使用专用账号（勿与临时测试账号混用）。

**授权示例（在每台后端执行，按实际账号名替换）：**

```sql
-- 查看当前后端账号（与 conf.json servers[].user 一致）
-- 示例账号：dbprisma_backend

GRANT CONNECTION_ADMIN ON *.* TO 'dbprisma_backend'@'%';
FLUSH PRIVILEGES;

-- 验证（应含 CONNECTION_ADMIN）
SHOW GRANTS FOR 'dbprisma_backend'@'%';
```

**与 conf.json 的对应关系：**

| conf.json 字段 | 说明 |
|----------------|------|
| `servers[].user` | 应使用已授予 `CONNECTION_ADMIN` 的后端专用账号 |
| `servers[].password` | 明文或 `ENC2:` 加密；生产建议用 XR:> `passwd`（REPL 运行时命令）或 Shell 子命令 `DBPrisma setconfig srvpwd <id>` 生成密文 |
| `servers[].host` / `port` | 每台后端均需单独执行 `GRANT`（多实例部署时逐台操作） |

> **注意**：`CONNECTION_ADMIN` 为较高权限，仅授予代理专用后端账号，勿授予应用直连账号。MySQL 5.7 / 8.0.25 及以下无此权限，请改用方向一或方向二。
> **scheme B 不适用**：透明代理不持有固定后端账号（客户端各自的凭据直传后端），无法对「代理账号」授权，请改用**方向二（`host_cache_size=0`）**。

#### 7.4.4 上线与压测检查清单

通用：
- [ ] 已区分客户端 1129 与后端 1129：前者查 `security.auth_fail_*` 与代理日志；后者查后端 host 封禁与代理机 IP 是否被封
- [ ] 长 soak / 全量回归前已从跳板机 `flush-hosts`（若此前有压测失败累积）

**scheme A（认证代理）**：
- [ ] 已确认代理为 2026-06 之后版本（裸 TCP 探测已根治；`XR:> backends` 正常）
- [ ] 每台后端 `max_connect_errors` 已按基线调高（方向一）
- [ ] （可选，MySQL 8.0.26+）后端专用账号已授予 `CONNECTION_ADMIN`（方向三）
- [ ] `conf.json` 后端 `user`/`password` 与授权账号一致，暖机可成功

**scheme B（透明代理）**：
- [ ] 每台后端已设 `host_cache_size = 0`（方向二，首选）
- [ ] 每台后端已设 `skip_name_resolve = ON`
- [ ] `servers[].warmup_tcp_connections` 已弃用：裸 TCP 预热已退役（曾是 `ERROR 1129` 根因）。透明代理下该值被忽略并改用冷连接，无需再手动置 0；如仍配置 >0，`checkconfig` 会给出弃用 WARN

> **`ERROR 1129 (Host 'x' is blocked because of many connection errors)` 根因已修复**：旧版「裸 TCP 预热池」对后端建立未认证连接后丢弃，被 MySQL 计入 host 级连接错误并最终拉黑代理。现已退役裸 TCP 预热——认证代理 + 明文后端改用「认证预热」（`warmup_db_connections`，或由 `warmup_tcp_connections` 折算），其余场景用冷连接。残留封禁仍可用 `mysqladmin flush-hosts` 或 `FLUSH HOSTS;` 临时解除。

---

## 8. 故障排查手册

### 8.1 故障排查总流程

```
故障发生
  │
  ├── XR:> status            → 确认代理是否运行（XR:> 会话命令）
  ├── XR:> backends          → 确认后端是否可达（XR:> 会话命令）
  ├── grep ERROR logs/...    → 查看最新错误日志（Shell 命令）
  └── curl /health           → 确认 Admin HTTP 正常（Shell 命令）

  ┌── 代理未运行 ──────────────> 重启代理
  ├── 后端 DOWN ─────────────> 排查后端 MySQL
  ├── 连接耗尽 ──────────────> 调整 max_clients / 清理空闲连接
  └── 高延迟 ────────────────> 查看慢查询日志 / 后端负载
```

### 8.2 客户端无法连接代理

**现象**：`mysql: ERROR 2003 (HY000): Can't connect to MySQL server on '127.0.0.1' (111)`

排查步骤：
```bash
# 1. 确认代理进程是否运行（Shell 命令）
ps -ef | grep DBPrisma

# 2. 确认端口是否监听
netstat -tlnp | grep 3307        # Linux
netstat -an | findstr 3307       # Windows

# 3. 检查日志启动错误
tail -50 logs/dbprisma.log

# 4. 检查防火墙
iptables -L INPUT -n             # Linux
netsh advfirewall show all       # Windows
```

若代理运行中，可在 XR:> 会话中执行：
```
XR:> status
```

### 8.3 认证失败 Access Denied

**现象**：`ERROR 1045 (28000): Access denied for user 'xxx'@'xxx'`

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 确认代理用户存在
XR:> userlist

# 2. 若用户不存在，添加用户（交互式密码提示，不回显）
XR:> useradd xxx

# 3. 若存在但密码错误（交互式密码提示，不回显）
XR:> setpwd xxx
```
```bash
# 4. 检查 IP 是否被封禁
grep "ban\|blocked\|denied" logs/dbprisma.log | tail -20

# 5. 若 IP 被封禁，需等待 auth_fail_ban_ms 毫秒后自动解封
# 或重启代理（会清空封禁状态）
```

### 8.4 连接数耗尽

**现象**：`ERROR 1040 (08004): Too many connections`

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 查看当前连接数
XR:> conns

# 3. 踢出长时间空闲连接（找出 STATE=IDLE 且 DURATION 很长的连接，然后踢出）
XR:> conns
XR:> kick <client_ip> [port]
```
```bash
# 2. 检查 max_connections_per_ip 是否触发
grep "max_connections_per_ip" logs/dbprisma.log | tail -10

# 5. 检查应用连接池是否有泄漏
```
```
# 4. 提升 max_clients（须停服 → setconfig proxy 向导修改 → 重启）
XR:> stop
```
```bash
./DBPrisma setconfig proxy && ./DBPrisma start --no-interactive --config config/conf.json
```

### 8.5 SQL 被防火墙拦截

**现象**：客户端收到 `MySQL Error 1148: SQL blocked by DBPrisma firewall`

**原理**：防火墙对以下 SQL 执行大小写不敏感子串匹配：
- `COM_QUERY` 及 `COM_STMT_PREPARE` — 检查原始 SQL 文本
- `COM_STMT_EXECUTE` — **双重检查**：① 合成记录文本；② 原始 PREPARE SQL（来自 `fw_stmt_map`）

**注意**：`reject` 模式下只拒绝该条命令并返回 1148 错误，**连接不会断开**，应用可继续发送其他 SQL。

排查步骤：
```bash
# 1. 确认 SQL 防火墙是否开启及当前模式（查看磁盘配置，独立子命令）
./DBPrisma setconfig show security

# 2. 查看被拦截的 SQL 和命中规则
#    主日志（当 sql_firewall_log_blocked=true 时）
grep "FIREWALL_BLOCK" logs/dbprisma.log | tail -20
#    或专用防火墙审计日志（当 firewall_log_file 已配置时）
tail -50 logs/firewall.log
```

若是误拦截，须停服 → setconfig security 向导修改 action → 重启：
```
# 3. 在 XR:> 会话中停止服务
XR:> stop
```
```bash
# 通过 setconfig 修改（security.* 须重启生效）
./DBPrisma setconfig security    # 向导中将 sql_firewall_action 改为 log_only
./DBPrisma start --no-interactive --config config/conf.json

# 4. 调整 sql_block_patterns 后切回 reject（同样须 restart）
./DBPrisma setconfig security    # 向导中将 sql_firewall_action 改回 reject
./DBPrisma start --no-interactive --config config/conf.json

# 5. 若需彻底关闭（生产须记录原因和时间窗口）
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改为 false
./DBPrisma start --no-interactive --config config/conf.json
```

> **提示**：`log_config.firewall_log_file` 写入独立审计流；`log_config.firewall_log_file`/`firewall_log_max_size_mb`/`firewall_log_rotate_count` **支持 `reload` 热更**（路径变更时自动重开文件）；**`security.*`（规则/action/enabled）须 stop→start**（不支持 `reload`）。完整剧本见 [技术 Runbook §7A](../technical/09-operations-runbook.md#7a-事件响应剧本-playbooks)、[迁移指南 §7A](../technical/18-migration-upgrade-guide.md#7a-关闭或回滚-sql-防火墙)。

### 8.6 后端连接失败

**现象**：`Backend connection failed: 192.168.1.10:3306`，或客户端/日志中出现：

`Error 1129: Host '<proxy_ip>' is blocked because of many connection errors`

> 若报文含 `flush-hosts`，且被封 IP 为**代理机地址**，属于后端 MySQL `max_connect_errors` 封禁，见 [§7.4](#74-防止后端-mysql-封禁代理-ipmax_connect_errors--error-1129)。若 1129 出现在**客户端 IP** 且与认证失败相关，则为 DBPrisma 自身 IP 封禁，见 [§8.3](#83-认证失败-access-denied)。

排查步骤：
```bash
# 1. 直接测试后端连通性（在代理机上执行；若代理 IP 已被封，此步会复现 1129）
mysql -h 192.168.1.10 -P 3306 -u dbprisma -p

# 2. 若为后端 1129：从跳板机解封并落实 §7.4 预防措施
mysqladmin -h 192.168.1.10 -P 3306 -u <admin> -p flush-hosts
# 长期：调高 max_connect_errors（§7.4.1）；MySQL 8.0.26+ 可为后端账号授予 CONNECTION_ADMIN（§7.4.2）

# 4. 检查日志中的后端错误
grep "backend\|connect\|1129" logs/dbprisma.log | grep ERROR | tail -20
```

在 XR:> 会话中执行：
```
# 3. 检查后端 MySQL 状态
XR:> backends    # 查看具体后端状态

# 5. 检查熔断器是否触发（状态为 OPEN）
XR:> status      # 查看 CB 状态

# 6. 若熔断器 OPEN，等待 open_duration_ms 后自动进入 HALF_OPEN 探测
```

### 8.7 高延迟 / 慢查询

排查步骤：
```bash
# 1. 查看慢查询日志
tail -100 logs/slow_query.log
```

在 XR:> 会话中执行：
```
# 2. 开启 DEBUG 日志短暂抓取
XR:> loglevel DEBUG
# 等待几分钟
XR:> loglevel INFO

# 3. 开启 SQL 捕获（进入阻塞捕获模式；P=暂停，E=退出）
XR:> capture

# 4. 查看后端延迟
XR:> backends     # 查看 LATENCY 列
```

```sql
-- 5. 检查后端 MySQL 负载（在 MySQL 端执行）
SHOW FULL PROCESSLIST;
SHOW STATUS LIKE 'Threads%';
```

### 8.8 内存持续增长

排查步骤（以下命令在 XR:> 会话中执行）：
```
# 1. 查看代理内存使用
XR:> status       # 查看 RSS

# 3. 检查是否有大量 IDLE 连接未清理（idle_timeout_sec 是否合理）
XR:> conns
```
```bash
# 2. 查看连接是否有泄漏
grep "session_leak\|LEAK" logs/dbprisma.log | tail -20
# 配置 resilience.session_leak_detect = true 可自动检测
```

---

## 9. 紧急操作手册（SOE）

### 9.1 全量封禁某 IP 段（安全事件）

在 XR:> 会话中执行：
```
# 先踢出该 IP 已有连接（kick 接受 IP 字符串）
XR:> kick 10.20.30.40

# 停服
XR:> stop
```
```bash
# 封禁 IP 或 CIDR（如 203.0.113.0/24）；security.* 须停服 → 修改 → 重启才生效
# 手动在 conf.json security.ip_denylist 数组中追加 "10.20.30.40"
# （或使用 setconfig security 向导，数组字段须手动编辑 conf.json）
./DBPrisma setconfig security
./DBPrisma start --no-interactive --config config/conf.json

# 记录操作日志
echo "$(date) Banned IP 10.20.30.40 by operator $(whoami)" >> /opt/dbprisma/logs/ops-audit.log
```

### 9.2 紧急关闭 SQL 防火墙（误拦截生产 SQL）

在 XR:> 会话中停服，然后用独立命令修改配置：
```
# 1. 停服
XR:> stop
```
```bash
# 通过 setconfig security 向导关闭（或改 log_only 保留审计）
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改为 false
                                 # 或将 sql_firewall_action 改为 log_only

# 2. 重启使安全配置生效
./DBPrisma start --no-interactive --config config/conf.json

# 3. 记录变更原因与时间窗口；通知安全团队
# 4. 修复规则后重开并 restart
./DBPrisma setconfig security    # 向导中将 sql_firewall_enabled 改回 true
./DBPrisma start --no-interactive --config config/conf.json
```

### 9.3 代理进程失响（假死）

```bash
# 1. 确认进程存活但不响应（Shell 命令）
ps -ef | grep DBPrisma

# 2. 查看是否有线程卡死
cat /proc/$(pgrep DBPrisma)/status | grep Threads
```

在 XR:> 会话中执行（若进程能响应）：
```
# 3. 尝试优雅重启
XR:> restart
```

```bash
# 4. 若重启失败，强制 kill 后重启
kill -9 $(pgrep DBPrisma)
./DBPrisma start --no-interactive --config config/conf.json

# 5. 启用 Watchdog 防止再次发生（在 conf.json 中配置）
# watchdog.enabled = true
# watchdog.action = "restart"
```

### 9.4 配置误操作回滚

```bash
# 方法1：使用自动备份回滚（备份与 conf.json 同目录；命名 conf.json.bak.YYYYMMDD_HHMMSS）
ls -lt config/conf.json.bak.*
cp config/conf.json.bak.20260605_093000 config/conf.json
```

在 XR:> 会话中执行：
```
# 热更（仅对可热更字段生效）
XR:> reload
```

```bash
# 方法2：使用手动备份
cp conf/conf.json.manual-20260605-093000 conf/conf.json
```

在 XR:> 会话中执行：
```
XR:> reload
```

### 9.5 磁盘空间告急（日志暴涨）

```bash
# 立即压缩旧日志
gzip logs/dbprisma.log.2 logs/dbprisma.log.3

# 清理超期日志（保留最近 3 天）
find logs/ -name "*.log.*" -mtime +3 -delete
```

在 XR:> 会话中执行：
```
# 刷新后端预热连接池
XR:> flush

# 临时降低日志级别
XR:> loglevel WARN
```

---

## 10. 变更操作规程

### 10.1 变更操作原则

1. **先备份后操作**：任何配置变更前备份 `conf.json`
2. **先 `checkconfig` 后 `reload`**：验证通过再热更
3. **灰度测试**：重大变更先在测试环境验证
4. **操作记录**：所有变更操作记录到运维日志
5. **回滚预案**：操作前确认回滚方案

### 10.2 标准变更流程

```
1. 申请变更时间窗口（通知相关业务方）
2. 备份当前配置：cp conf.json conf.json.backup-$(date +%Y%m%d)
3. 修改配置文件
4. 验证（独立子命令）：./DBPrisma checkconfig
5. 执行热更（XR:> 会话）：XR:> reload
6. 验证（XR:> 会话）：XR:> status 然后 XR:> backends
7. 观察 5-10 分钟（检查错误日志、业务指标）
8. 填写变更记录
```

### 10.3 紧急变更流程

```
1. 通知直属负责人
2. 备份当前配置
3. 执行最小变更（仅修改必要项）
4. 立即验证
5. 变更完成后 30 分钟内补填变更记录
```

---

## 11. 监控告警配置参考

### 11.1 推荐告警规则（Prometheus/Alertmanager）

```yaml
# 代理宕机
- alert: DBPrismaDown
  expr: up{job="dbprisma"} == 0
  for: 30s
  labels:
    severity: critical
  annotations:
    summary: "DBPrisma proxy is down"

# 后端 DOWN
- alert: DBPrismaBackendDown
  expr: dbprisma_backend_status == 0
  for: 1m
  labels:
    severity: warning

# 活跃连接数超阈值
- alert: DBPrismaConnectionHigh
  expr: dbprisma_connections_active > 800
  for: 2m
  labels:
    severity: warning

# 错误率过高
- alert: DBPrismaErrorRateHigh
  expr: rate(dbprisma_errors_total[5m]) > 10
  for: 2m
  labels:
    severity: critical

# 后端延迟过高
- alert: DBPrismaBackendLatencyHigh
  expr: dbprisma_backend_latency_ms > 100
  for: 3m
  labels:
    severity: warning
```

### 11.2 健康检查集成（Nginx upstream）

```nginx
upstream dbprisma {
    server 127.0.0.1:3307 max_fails=3 fail_timeout=30s;
}

# 健康检查（需 nginx_upstream_check_module）
check interval=3000 rise=2 fall=3 timeout=1000 type=http;
check_http_send "GET /health HTTP/1.0\r\nAuthorization: Bearer TOKEN\r\n\r\n";
check_http_expect_alive http_2xx;
```

---

## 12. 运维命令速查表

> **说明**：标注「XR:>」的命令须在交互会话中执行；标注「Shell」的可直接从终端调用。

| 场景 | 命令 | 类型 |
|------|------|------|
| 查看代理状态 | `XR:> status` | XR:> |
| 查看后端健康 | `XR:> backends` | XR:> |
| 查看活跃连接 | `XR:> conns` | XR:> |
| 踢出连接 | `XR:> kick <client_ip> [port]` | XR:> |
| 热更配置 | `XR:> reload` | XR:> |
| 修改单项配置（磁盘） | `./DBPrisma setconfig <verb> [args]`（如 `setconfig srvpwd 1`） | Shell |
| 修改日志级别 | `XR:> loglevel <level>` | XR:> |
| 封禁 IP | 停服 → 手动编辑 conf.json `security.ip_denylist` → 重启 | Shell |
| 暂停新连接 | `XR:> pause` | XR:> |
| 恢复新连接 | `XR:> resume` | XR:> |
| 排水（drain） | `XR:> drain <server_id>` | XR:> |
| 优雅重启 | `XR:> restart` | XR:> |
| 优雅停止 | `XR:> stop` | XR:> |
| 带倒计时停止 | `XR:> shutdown [秒] [force\|cancel]` | XR:> |
| 验证配置 | `./DBPrisma checkconfig` 或 `./DBPrisma cc` | Shell |
| 轮换 Token | `XR:> token rotate` | XR:> |
| 刷新预热池 | `XR:> flush [server_id]` | XR:> |
| SQL 捕获（阻塞模式；P=暂停，E=退出） | `XR:> capture [opts] [screen\|json\|all]` | XR:> |
| 后台 SQL 捕获（启动时） | `./DBPrisma start --bg-capture` | Shell |
| 状态一次性显示（读磁盘） | `XR:> display` | XR:> |
| 查看代理用户 | `XR:> userlist` | XR:> |
| 添加代理用户 | `XR:> useradd <username>`（交互式密码提示） | XR:> |
| Admin 健康检查 | `curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/health` | Shell |
| Admin 指标 | `curl -H "Authorization: Bearer $TOKEN" http://127.0.0.1:8080/metrics` | Shell |

---

## 13. 授权与 License 管理

DBPrisma 启动时由 License 闸门（`DBPrisma/license_gate.c`）控制是否放行，逻辑完全离线，不联网。

### 13.1 判定规则

| 情况 | 结果 | 行为 |
|------|------|------|
| **config 目录无 `license.dat`** | 试用 | 自首次运行起 30 天内放行；到期后拒绝启动 |
| `license.dat` 验签通过、`key_id` 与内嵌 `VENDOR_KEY_ID` 一致、绑定本机、已到 `activate_utc`、未过期（或在 `grace_days` 宽限期内） | 授权 | 放行 |
| `license.dat` 签名无效 / 文件损坏 / 格式错误 / `key_id` 与内嵌公钥不匹配 | 拒绝 | **报错中止，不回落试用** |
| `license.dat` 绑定到其它机器 | 拒绝 | 报错中止 |
| `license.dat` 未到 `activate_utc`（预签发尚未生效） | 拒绝 | 报错中止 |
| `license.dat` 已过期且超出 `grace_days` 宽限期 | 拒绝 | 报错中止 |
| 试用状态相对系统时钟回拨 | 拒绝 | 报错中止 |

> 关键约定：**只要存在 `license.dat`，它就必须有效**；只有「完全没有该文件」时才走 30 天试用。宽限期内仍放行，但 `Detail` 会提示已进入 grace period。

### 13.2 相关文件与位置

| 文件 | 位置 | 说明 |
|------|------|------|
| `license.dat` | **config 目录**（与 `conf.json` 同级） | 厂商签发的授权文件，由客户放置 |
| `dbprisma.trial.state` | **run 目录** | 试用起点（机器绑定 AES-GCM 封存，**勿删**，删除会重置 30 天） |

> 启动时控制台/日志会打印一行 `[license] ...` 表明当前授权状态与剩余天数。
> 守护进程/服务模式下每小时复检一次，license/试用到期会自动停止服务。

**到期前提示（`license.warn_days`）**：当 license **或**试用剩余天数 ≤ `license.warn_days`（默认 14 天，见 [配置参考](configuration-reference.md)）时，启动会打印 `[license] License/Trial expires in N day(s); renew ...`，守护模式下每小时复检也会重复该提醒；`status verbose` 的 `Days left` 在此阈值内转黄。设 `warn_days=0` 关闭提醒。此提示对**正式 license 与试用同时生效**（对称）。

**监控指标**：`/metrics`（及命令管道 `metrics` op）输出两项授权指标，便于到期前自动告警：
> - `dbprisma_license_state`：`2`=授权、`1`=试用、`0`=拒绝、`-1`=未知
> - `dbprisma_license_days_left`：剩余整天数（`-1`=未知）

示例告警规则：`dbprisma_license_state >= 1 and dbprisma_license_days_left <= 14`。

**随时查看授权状态**：在 `XR:>` 会话执行 `status verbose`，输出末尾的 `---- License ----` 段会显示 `Mode`（LICENSED / TRIAL / DENIED）、`Days left`（剩余天数）与 `Detail`（如 `licensed to <组织>, expires ...` 或 `trial: N day(s) left`）。这是除启动行外随时可查的入口（不会重置试用）。

### 13.3 申请与部署流程

```
客户: licreq --print-fingerprint（可选，查看本机指纹）
客户: licreq 生成 license.request（含本机机器指纹与订单 SN；月数由 SN 解析）→ 核对生成摘要或 licreq --show <file> → 发给厂商
厂商: licissue show 审核（核对 machine_fp；可加 --art/--all 打印 randomart）
      → licissue issue --private-key ... --public-key ... 签发 license.dat（可 --months / --days / --activate-utc / --grace-days；台账 issued-ledger.jsonl）
      → licissue inspect -i license.dat 复核（可选）
客户: 将 license.dat 放入 config 目录 → 重启 DBPrisma
```

**客户侧常用命令**：

```bash
licreq --print-fingerprint              # 默认：单行 hex，便于脚本捕获
licreq --print-fingerprint --art        # 仅 randomart 图案（人工核验）
licreq --print-fingerprint --all        # hex + 图案
licreq -o license.request               # 交互式生成申请（成功后会打印请求摘要）
licreq --show license.request           # 解密并回显已生成的申请文件
# 机器指纹：SHA256(CPU序列号 | hostname | salt)，与 license.dat 绑定字段同源
```

**厂商侧签发示例**（须同时提供私钥与公钥）：

```bash
licissue show -i license.request
licissue issue -i license.request -o license.dat \
  --private-key vendor.key --public-key vendor.pub -y
# 按天签发（覆盖申请月数）：--days 28
licissue inspect -i license.dat         # 查看序列号、四时间字段、宽限天数等
```

**跨平台私钥（portable key）**：自 2026-06-22 起，`licissue keygen` 生成的私钥
（`DBPVK1`，150 字节 = `d‖X‖Y`）与公钥一样是跨平台格式，**同一个私钥文件可在
Windows 与 Linux 上签发**，签出的 `license.dat` 在所有 DBPrisma（任意 OS/架构）上
均可验证。公钥与 `key_id` 不变，**无需重新 keygen、无需重编 DBPrisma**。

历史上 Windows 私钥是 BCrypt blob（158 字节）、Linux 是裸标量（54 字节），互不通用。
两种旧格式仍可被新 `licissue` 读取（向后兼容）；将旧私钥一次性迁移为跨平台格式：

```bash
licissue convert-key -i vendor.old.key -o vendor.key   # 公钥/ key_id 不变
```

> 迁移 Windows 旧 blob 在任意平台均可；迁移 Linux 旧裸标量需在 Linux 上执行
> （需用曲线运算推导公钥点）。

详见 `tools/licreq/README.md`、`tools/licissue/README.md`。

### 13.4 启动失败排障（`[license] Startup blocked: ...`）

| 提示 | 原因 | 处置 |
|------|------|------|
| `trial period has ended` | 30 天试用到期且无 license | 申请并放入有效 `license.dat` |
| `trial state invalid or tampered` | `run/dbprisma.trial.state` 被改动/损坏/清空，或机器身份变化（CPU 标识、主机名变化、整目录跨机拷贝）导致机器绑定 AES-GCM 解封失败 | 正解：放入有效 `license.dat`（不再走试用）。开发/测试环境如确需重置试用：删除 `run/dbprisma.trial.state` 后重启（**等于重置 30 天**，受授权策略约束） |
| `license.dat is invalid or corrupt` | 文件被改动/损坏，或 `key_id` 与内嵌公钥不匹配（密钥轮换后旧证） | 重新获取与当前产品版本公钥匹配的 `license.dat`；确认已 `keygen --emit-header` 并重编译 |
| `license.dat is bound to a different machine` | license 绑定的机器指纹与本机不符（换机/重装/CPU 或主机名变化） | 本机 `licreq --print-fingerprint` 与 `licissue show` 中的 `machine_fp` 对比；不符则重新 `licreq` 申请并换发 |
| `license not active until ...` | 预签发的 license 尚未到 `activate_utc` | 等到生效日，或请厂商重签（调整 `--activate-utc`） |
| `license expired at ...` | license 到期且超出 `grace_days` 宽限期 | 续期换发新的 `license.dat` |
| `system clock rolled back` | 系统时钟相对试用起点回拨 | 校正系统时间（NTP） |

> 机器指纹来源与 ENC2 密钥同源（`SHA256(CPU序列号|hostname|salt)`）。更换 CPU、主机名或重装可能导致指纹变化、需换发 license。

---

*配置详情参阅 [配置参考手册](configuration-reference.md)*  
*安装部署参阅 [管理员手册](admin-manual.md)*  
*安全加固参阅 [安全加固指南](security-hardening-guide.md)*  
*授权工具参阅 [licreq](../../tools/licreq/README.md) / [licissue](../../tools/licissue/README.md)*


---

# 第六篇 安全加固指南

> **来源**：`security-hardening-guide.md`

---


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


---

# 第七篇 SSL/TLS 行为矩阵

> **来源**：`ssl-behavior-matrix.md`

---


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


---

# 第八篇 透明代理方案 B

> **来源**：`scheme-b-transparent.md`

---


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


---

# 第九篇 命令管道客户端开发手册

> **来源**：`command-pipe-client-guide.md`

---


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


---

# 第十篇 命令管道命令参考手册

> **来源**：`command-pipe-command-reference.md`

---


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


---
