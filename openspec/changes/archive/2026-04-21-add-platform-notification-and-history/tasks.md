## 1. Data Model And Migration

- [x] 1.1 Extend the SQLite storage layer with tables and indexes for platforms, notification channels, accounts, account-notification bindings, and execution records.
- [x] 1.2 Seed the built-in `mt` platform with fixed API settings and add startup migration logic that converts existing single-task settings into default account and `tg` channel records when old data exists.
- [x] 1.3 Add storage-layer CRUD helpers and validation rules for enabled state, secret-field preservation, and account/channel/platform relationships.

## 2. Management UI And Routes

- [x] 2.1 Refactor the dashboard routes and templates to expose separate management sections for platform configuration, notification channels, login accounts, and execution history queries.
- [x] 2.2 Implement the platform list UI with built-in `mt` display, fixed read-only API fields, and enabled-state handling.
- [x] 2.3 Implement notification channel create/edit flows for `tg` entries, including masked secret handling and enabled-state filtering.
- [x] 2.4 Implement login account create/edit/list flows with platform selection, multi-select notification binding, current task execution fields, and account enable/disable handling.

## 3. Scheduling, Execution, And History

- [x] 3.1 Refactor the scheduler to load enabled accounts, compute per-account schedules, and support manual execution at the account level.
- [x] 3.2 Extract or gate the current notification logic so Web executions can dispatch result notifications through selected `tg` channels without duplicate sends.
- [x] 3.3 Persist an execution record for every manual or scheduled account run, including trigger mode, status, message, timestamps, and returned metrics.
- [x] 3.4 Implement execution history query filters for account, platform, status, and time range, ordered by newest runs first.

## 4. Verification And Documentation

- [x] 4.1 Add focused tests or verification coverage for migration, account/channel binding rules, scheduler execution writes, and execution history filtering.
- [x] 4.2 Update README and admin usage guidance to describe platform management, notification management, account binding, and execution history querying.
