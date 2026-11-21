# DBPrisma 快速入门指南

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
