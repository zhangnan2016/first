## ADDED Requirements

### Requirement: Login and token issuance
The system SHALL provide a login endpoint that validates credentials and, on success, issues a Sa-Token session and returns a token. The token SHALL be required as `Authorization: Bearer <token>` on protected endpoints.

#### Scenario: Successful login issues a token
- **WHEN** a client posts valid credentials to the login endpoint
- **THEN** the response contains a token string and the token is bound to a Sa-Token session

#### Scenario: Failed login is rejected
- **WHEN** a client posts invalid credentials
- **THEN** no token is issued and an authentication error `ResultCode` is returned

### Requirement: Token interception
A Sa-Token interceptor SHALL protect endpoints except an explicit allow-list (e.g. login, API docs). Requests without a valid token SHALL be rejected with a not-authenticated `ResultCode`.

#### Scenario: Unauthenticated request is rejected
- **WHEN** a protected endpoint is called without a token
- **THEN** the response is `R<Void>` with a not-authenticated `ResultCode`

#### Scenario: Login route is always reachable
- **WHEN** the login endpoint is called without a token
- **THEN** it is not blocked by the interceptor

### Requirement: Permission annotations
The system SHALL support Sa-Token permission checks via annotations (e.g. `@SaCheckPermission`). Endpoints annotated with a permission SHALL deny callers lacking that permission.

#### Scenario: Missing permission is denied
- **WHEN** an authenticated caller without the required permission hits an annotated endpoint
- **THEN** the response is `R<Void>` with an authorization `ResultCode`
