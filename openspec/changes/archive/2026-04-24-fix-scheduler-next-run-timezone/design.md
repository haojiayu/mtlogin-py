## Context

The web admin currently calculates next scheduled executions with `croniter(..., datetime.now())` in both account serialization and the background scheduler loop. These datetimes are naive and inherit the Python process timezone. In Docker this often differs from the administrator's intended local timezone, so a cron expression such as "run at 02:00 local time" can be interpreted as UTC or another container timezone.

The bug affects both display and behavior: `/api/admin/accounts` reports `next_run_at` from the process clock, while `BackgroundScheduler` uses the same process clock to choose the next account, build `schedule_message`, sleep, and decide which scheduled accounts are due.

## Goals / Non-Goals

**Goals:**

- Calculate all web-admin account schedules from one explicit scheduler timezone.
- Keep the existing response fields and timestamp string format stable.
- Make Docker deployments configurable without requiring code changes.
- Cover the timezone behavior with focused backend tests.

**Non-Goals:**

- Add per-account timezone settings.
- Change the cron expression syntax.
- Migrate stored timestamps in execution history.
- Change the standalone `mtlogin.py` script scheduler unless it is needed to share a small helper safely.

## Decisions

### 1. Use one app-level scheduler timezone

Add an app configuration value sourced from `SCHEDULER_TIMEZONE`, then `TZ`, defaulting to the process local timezone when neither is set. The scheduler and account serializers will both receive/use this same timezone.

Alternatives considered:

- Use UTC everywhere. Rejected because existing users write cron expressions as local operational time, and forcing UTC would be a behavioral break.
- Rely only on Docker `TZ`. Rejected as the only fix because it is easy to omit and does not make the scheduler-specific contract explicit.
- Add timezone per account. Rejected for this bug fix because it increases UI and data-model scope without evidence that accounts need separate timezones.

### 2. Centralize next-run calculation

Replace the global `next_run_text(crontab_expr)` helper with timezone-aware helpers that calculate both the next `datetime` and formatted text from the configured scheduler timezone. `serialize_account()` and `BackgroundScheduler._list_scheduled_accounts()` should call the same calculation path.

Alternatives considered:

- Patch only the account page display. Rejected because the user also reported Docker execution not matching the cron expression.
- Patch only scheduler execution. Rejected because the account page would still display misleading operational state.

### 3. Keep API timestamp format unchanged

Continue returning `YYYY-MM-DD HH:MM:SS` strings for `next_run_at` and `schedule_message`. The timezone is a calculation context, not a new field in this focused fix.

Alternatives considered:

- Return ISO 8601 timestamps with offsets. Rejected for now because it would require frontend formatting changes and may break existing consumers.

## Risks / Trade-offs

- [Invalid timezone name] -> Fall back to the process local timezone and log a warning so startup remains compatible.
- [DST transitions] -> Defer to `croniter` with timezone-aware datetimes; add tests around normal timezone offsets first and leave DST-specific cases as follow-up if needed.
- [Confusion over displayed timezone] -> Document `SCHEDULER_TIMEZONE` in Docker/runtime docs and keep the scheduler calculation consistent across display and execution.

## Migration Plan

1. Deploy the code with no environment changes; behavior remains tied to `TZ` if present, otherwise the process local timezone.
2. For Docker deployments, set `SCHEDULER_TIMEZONE` to the intended IANA timezone such as `Asia/Shanghai`.
3. Restart the container so the scheduler recalculates waiting state and next-run values.
4. Rollback is safe because no database schema or API shape changes are introduced.
