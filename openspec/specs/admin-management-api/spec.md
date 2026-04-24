# admin-management-api Specification

## Purpose
TBD - created by archiving change migrate-admin-frontend-to-vite-vue. Update Purpose after archive.
## Requirements
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

### Requirement: Unauthenticated API requests are rejected without HTML redirects
The system SHALL reject unauthenticated requests to admin frontend JSON APIs with an authentication error response suitable for SPA clients, instead of returning an HTML management page redirect.

#### Scenario: Session expires before an API request
- **WHEN** the frontend sends a request to an admin JSON API without a valid administrator session
- **THEN** the system returns an authentication failure response for the API client
- **THEN** the frontend can use that response to redirect the user back to the login flow

### Requirement: JSON mutations preserve existing management validation rules
The system SHALL apply the same business validation and secret-field preservation rules to JSON-based admin mutations that currently apply to management form submissions.

#### Scenario: Frontend updates an account with blank secret fields
- **WHEN** the Vue admin frontend submits an account update request with blank password, TOTP, or auth token fields
- **THEN** the system preserves the previously saved secret values
- **THEN** the rest of the submitted account changes are validated and persisted using the existing management rules
