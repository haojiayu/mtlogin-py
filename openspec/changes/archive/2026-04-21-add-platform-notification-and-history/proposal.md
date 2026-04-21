## Why

当前后台只有一份任务配置，M-Team 账号参数、Telegram 参数和执行状态集中在同一页面，无法按平台、通知渠道和登录账户分别管理，也无法查询历史执行记录。随着配置项增加，这种单配置模型已经限制了可扩展性，因此需要先把配置结构调整为列表化管理，再补齐执行记录查询能力。

## What Changes

- 新增平台配置列表，支持启用或停用平台，并为账户配置提供可选平台来源。
- 内置 `mt` 平台，`API Host` 和 `API Referer` 使用固定值且在界面中只读展示。
- 为后续非 `mt` 平台预留可扩展字段，允许其他平台使用自定义 `API Host` 和 `API Referer`。
- 新增通知管理配置列表，允许维护多个通知渠道配置。
- 首期只支持 `tg` 通知渠道，参数沿用当前页面中的 Telegram 相关字段。
- 新增登录账户管理列表，用于维护多个登录账户；每个账户必须绑定一个平台，并可勾选一个或多个通知渠道。
- 将当前单条运行状态扩展为执行记录集合，支持按账户、平台、状态和时间范围查询执行结果。
- 保留“立即执行一次”和定时执行能力，但执行入口切换为针对账户运行并写入历史记录。

## Capabilities

### New Capabilities
- `platform-config-management`: 管理平台配置列表，提供内置 `mt` 平台和未来自定义平台的配置约束。
- `notification-channel-management`: 管理通知渠道列表，首期支持 `tg` 渠道及其参数维护。
- `login-account-management`: 管理登录账户列表，支持账户绑定平台和多个通知渠道。
- `execution-history-query`: 保存并查询执行记录，支持多条件筛选与结果展示。

### Modified Capabilities
None.

## Impact

- 受影响代码主要包括 [app.py](/Users/haojiayu/Documents/me/github/mtlogin-py/app.py) 的存储、路由、调度与运行流程，以及 [templates/dashboard.html](/Users/haojiayu/Documents/me/github/mtlogin-py/templates/dashboard.html) 的页面结构。
- SQLite 持久化模型需要从单条 KV 配置扩展为平台、通知渠道、账户、账户通知关联、执行记录等多实体结构，并提供旧配置迁移。
- 调度逻辑需要从“读取一份任务配置执行”改为“按账户读取关联平台与通知配置执行”。
- 文档和默认初始化逻辑需要同步更新，说明内置 `mt` 平台、`tg` 通知渠道和旧数据迁移策略。
