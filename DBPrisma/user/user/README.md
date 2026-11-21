# DBPrisma 用户文档中心

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
| 配置参考手册 | r19 | 2026-07-13 |
| 配置部署指南 | 无独立版本号 | 与现行配置参考同目录 |
| SSL/TLS 行为矩阵 | 文末 2026-06-28 | 2026-06-28 |
| 透明代理方案 B | 无独立版本号 | 现行稿密码示例为 `ENC2:` |
| 命令管道客户端开发手册 | r1 | 2026-06-17 |

---

## 相关文档（仓库内）

| 位置 | 用途 |
|------|------|
| [../study/README.md](../study/README.md) | 学习稿与术语 xlsx（同为开发证据；同题以本目录更高版本为准） |
| [../study/md/02-architecture-design.md](../study/md/02-architecture-design.md) | 架构设计摘要 |
| [../study/md/22-project-technical-glossary.md](../study/md/22-project-technical-glossary.md) | 技术术语表 |
| [../../_ingest/](../../_ingest/README.md) | 商业化话术/协议（非技术真身） |

产品导航：[../../README.md](../../README.md)

**修订**：v1.3（2026-06-17）— 新增[命令管道客户端开发手册](command-pipe-client-guide.md)（C#/Java/Python 对接控制管道与 capture 数据通道）及 [`examples/command-pipe/`](../../examples/command-pipe/) 示例；命令管道新增 `capture_queue_mb`/`capture_buf_kb` 缓冲配置（配置参考升至 r14）。  
v1.2（2026-06-10）— 配置项计数统一为 159；版本历史表对齐各文档实际版本（配置参考 r8、运维 v1.2、管理员/快速入门 v1.1）；修正 `start` 子命令须置于选项之前的示例（C-014）。  
v1.1（2026-06-06）— 紧急操作链至 Runbook §7A；修正技术文档路径；用户手册 v1.1 对齐 restart 语义。
