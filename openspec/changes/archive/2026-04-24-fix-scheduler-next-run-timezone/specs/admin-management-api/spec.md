## MODIFIED Requirements

### Requirement: Management frontend uses authenticated JSON APIs
The system SHALL provide session-authenticated JSON APIs for the admin frontend to read and mutate management data for platforms, notification channels, login accounts, execution history, system settings, and scheduler runtime state.

#### Scenario: Authenticated administrator loads management data
- **WHEN** an authenticated administrator opens a Vue-based management page
- **THEN** the frontend can request the corresponding management data through a JSON API
- **THEN** the API response contains structured data required to render that page

#### Scenario: Authenticated administrator loads scheduler runtime state
- **WHEN** an authenticated administrator opens the admin frontend and the scheduler has at least one enabled account with a valid cron expression
- **THEN** the bootstrap API exposes scheduler runtime state with `next_run_at` and `schedule_message`
- **THEN** those fields are calculated from the same configured scheduler timezone used for actual scheduled execution
