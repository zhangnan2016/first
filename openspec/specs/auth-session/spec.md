# Auth Session

## Purpose

Defines Sa-Token based authentication: login and token issuance, token interception with an allow-list, and permission annotation checks.

## Requirements

### Requirement: Login and token issuance
The system SHALL provide a login endpoint that validates credentials and, on success, issues a Sa-Token session and returns a token. The token SHALL be required as `Authorization: Bearer <token>` on protected endpoints.

#### Scenario: Successful login issues a token
- **WHEN** a client posts valid credentials to the login endpoint
- **THEN** the response contains a token string and the token is bound to a Sa-Token session

#### Scenario: Failed login is rejected without username enumeration
- **WHEN** a client posts invalid credentials (wrong password OR non-existent username)
- **THEN** no token is issued and an authentication error `ResultCode` is returned. The error message SHALL be identical in both cases (e.g. "用户名或密码错误") to prevent username enumeration.

#### Scenario: Brute-force protection
- **WHEN** a username fails login more than N times (e.g. 5) within a sliding window (e.g. 5 minutes)
- **THEN** subsequent login attempts for that username SHALL be temporarily blocked or require CAPTCHA verification

### Requirement: Token interception
A Sa-Token interceptor SHALL protect all endpoints except an exhaustive allow-list. The allow-list SHALL include exactly: `/auth/login`, `/doc.html`, `/webjars/**`, `/swagger-resources/**`, `/v3/api-docs/**`. All other paths SHALL require a valid token. The allow-list SHALL be defined in a single configuration location and reviewed when new public endpoints are added.

#### Scenario: Unauthenticated request is rejected
- **WHEN** a protected endpoint is called without a token
- **THEN** the response is `R<Void>` with a not-authenticated `ResultCode`

#### Scenario: Login route is always reachable
- **WHEN** the login endpoint is called without a token
- **THEN** it is not blocked by the interceptor

### Requirement: Permission annotations
The system SHALL enforce Sa-Token permission checks via annotations (`@SaCheckPermission`). All data-mutating endpoints (create/update/delete) and administrative query endpoints SHALL be annotated with the appropriate permission code. At minimum, `/sys/user/**` endpoints SHALL require `sys:user:list` (query) or `sys:user:manage` (mutate) permissions. Endpoints without an explicit permission annotation SHALL be treated as a spec violation.

#### Scenario: Missing permission is denied
- **WHEN** an authenticated caller without the required permission hits an annotated endpoint
- **THEN** the response is `R<Void>` with an authorization `ResultCode`
