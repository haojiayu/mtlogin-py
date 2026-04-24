## 1. Timezone Configuration

- [x] 1.1 Add a scheduler timezone setting sourced from `SCHEDULER_TIMEZONE`, defaulting to the process local timezone when unset.
- [x] 1.2 Add a small helper for timezone-aware current time and cron next-run calculation.
- [x] 1.3 Ensure invalid timezone names fall back safely and emit an operational log message.

## 2. Scheduler And API Behavior

- [x] 2.1 Update account serialization so `next_run_at` uses the configured scheduler timezone.
- [x] 2.2 Update `BackgroundScheduler` to use the same timezone-aware calculation for waiting state, sleep duration, and due-account selection.
- [x] 2.3 Keep existing API field names and `YYYY-MM-DD HH:MM:SS` timestamp formatting unchanged.

## 3. Tests And Documentation

- [x] 3.1 Add regression tests proving account API `next_run_at` is calculated from `SCHEDULER_TIMEZONE`.
- [x] 3.2 Add scheduler tests proving `schedule_message` / `next_run_at` and due execution use the configured timezone rather than the container process timezone.
- [x] 3.3 Document `SCHEDULER_TIMEZONE` for Docker/runtime deployments.
- [x] 3.4 Run the backend test suite and OpenSpec validation for this change.
