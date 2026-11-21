# DBPrisma Enterprise 安全审计专项
**日期：** 2026年11月

本文只说明息壤数据库串行审计安全网关的审计交付。记录、脱敏、导出、哈希链、校验和查询按开发手册中的配置键与命令书写，交付给客户现有平台采集。

## 记录范围

行为审计记录源 IP、身份、时间、SQL 和结果，并覆盖慢查询。

命令阶段把每条 SQL 写入 `capture.json`，并带上结果分类：OK、ERR 或未知。`log_config.log_include_sql` 为 true 时，日志行包含 SQL，并遵循 `capture_redact_literals`。

慢查询由 `log_config.slow_query_ms` 设定阈值，单位为毫秒，0 表示关闭。`log_config.slow_query_file` 指定慢查询文件；该路径为空时写入主日志。

SQL 防火墙的拦截另写入防火墙审计日志。配置 `log_config.firewall_log_file` 后，每次 `FIREWALL_BLOCK` 同时写入该文件。单行含时间、`pat` 与 `sql`。`firewall_log_max_size_mb` 与 `firewall_log_rotate_count` 控制滚动和保留份数。

## 落盘前脱敏

`proxy.capture_redact_literals` 在日志和 SQL 捕获落盘前替换字面量。取值为 true 时，字符串字面值替换为 `'?'`，数字字面值替换为 `?`，并作用于 `capture.json` 与相关日志。配置中含 `proxy` 段且省略本字段时，默认按开启处理。

`security.mask_client_ip_in_log` 为 true 时，日志中的客户端 IP 做部分脱敏：IPv4 记为 `A.B.x.x`，IPv6 记为 `a:b::x`。限流、踢出连接和捕获仍使用真实 IP。

## 导出格式

`log_config.audit_export_enabled` 为 true 时，写入独立于 `capture.json` 的审计导出文件。`log_config.audit_export_format` 取值为 json（NDJSON）、cef、syslog。`log_config.audit_export_file` 为导出路径，相对路径锚定 `paths.log_dir`；导出开启时该路径必填。上述三个键支持热重载。

## SHA-256 链

`log_config.audit_chain_enabled` 为 true 时，为 NDJSON 导出行附加 `prev_hash` 与 `line_hash`，形成 SHA-256 链。`log_config.audit_chain_key` 为哈希链盐值；留空时使用机器指纹派生默认值。这两个键支持热重载。

## 校验命令

`checkaudit` 不启动代理，校验 `log_config.audit_export_file` 导出的 NDJSON 哈希链，导出文件须含 `prev_hash` 与 `line_hash`。可直接指定导出文件，也可指定 `conf.json`。省略路径时，读取当前 `conf.json`，并解析 `paths.log_dir` 与 `audit_export_file`。没有哈希字段的行计为 `skipped`，不导致失败。

## 查询

管理接口提供 `GET /audit/search`。查询参数为 `q`、`event`、`user`、`tool`、`limit`、`offset`。`limit` 默认 20，最大 100。`offset` 跳过前 N 条匹配。响应含 `returned` 与 `has_more`。该接口为只读。配置了管理证书时，同一端口同时接受 HTTP 与 HTTPS。配置了 `admin_bearer_token` 时，请求须携带该令牌。其它路径返回 404，非 GET 返回 405。

## 交付

审计导出文件交给客户现有的安全信息与事件管理（SIEM），或交给客户现有的安全运营中心（SOC）。客户按所选的 json（NDJSON）、cef 或 syslog 采集 `audit_export_file`。

## POC 验收

| 核对项 | 核对什么 |
| --- | --- |
| 记录范围 | 抽查一条捕获记录，确认含源 IP、身份、时间、SQL 与结果分类（OK、ERR 或未知） |
| 慢查询 | 核对 `slow_query_ms` 与 `slow_query_file` 是否按阈值留下慢查询 |
| 防火墙审计日志 | 触发一次拦截，核对 `firewall_log_file` 写入带 `pat` 与 `sql` 的审计行 |
| 落盘前脱敏 | 核对 `capture_redact_literals` 开启后，落盘 SQL 的字面量已替换 |
| 导出格式 | 核对 `audit_export_format` 为 json、cef 或 syslog，且 `audit_export_file` 有输出 |
| SHA-256 链 | 核对 NDJSON 行含 `prev_hash` 与 `line_hash` |
| checkaudit | 执行 `checkaudit`，确认哈希链校验完成；无哈希字段的行记为 `skipped` |
| GET /audit/search | 用 `q`、`event`、`user`、`tool` 查询，核对响应中的 `returned` 与 `has_more` |
| 交付 | 确认导出文件可交给客户现有的安全信息与事件管理（SIEM）或安全运营中心（SOC） |
