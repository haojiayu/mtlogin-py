## ADDED Requirements

### Requirement: System manages login accounts as independent records
The system SHALL provide a login account list where administrators can create, edit, enable, disable, and view multiple login accounts independently.

#### Scenario: Administrator creates a new login account
- **WHEN** an administrator submits a valid login account form
- **THEN** the system creates a distinct account record
- **THEN** the account list shows the account name or username, bound platform, and enabled state

### Requirement: Each account binds exactly one enabled platform
The system SHALL require every login account to bind to exactly one enabled platform entry before the account can be saved.

#### Scenario: Account is saved with a platform selection
- **WHEN** an administrator creates or edits an account and selects one enabled platform
- **THEN** the system saves the account with that platform binding

### Requirement: Accounts can bind multiple notification channels
The system SHALL allow administrators to select zero or more enabled notification channels for each login account.

#### Scenario: Account binds more than one notification channel
- **WHEN** an administrator checks multiple enabled notification channels in the account form
- **THEN** the system saves all selected channel bindings for that account

### Requirement: Account execution fields preserve existing secrets when blank
The system SHALL keep the current saved value for account secret fields when an administrator edits an account and leaves those secret inputs blank.

#### Scenario: Password and token remain unchanged after blank edit
- **WHEN** an administrator edits an existing account and leaves password, TOTP, or auth token inputs blank
- **THEN** the system preserves the previously stored secret values for those fields

### Requirement: Account configuration includes current task execution options
The system SHALL support the existing task execution fields for each account, including username, password, TOTP secret, proxy, cron expression, auth token, DID, timeout, cookie mode, and skip-cache behavior.

#### Scenario: Administrator configures scheduled execution for an account
- **WHEN** an administrator saves an account with login credentials and a valid cron expression
- **THEN** the system persists the account-specific execution options for future scheduled runs

