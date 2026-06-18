## ADDED Requirements

### Requirement: Axios instance with token injection
A central Axios instance SHALL prepend the backend base URL to every request and inject the current token into the `Authorization: Bearer <token>` header when a token is stored.

#### Scenario: Token header is attached
- **WHEN** a request is sent while a token is stored
- **THEN** the outgoing request carries `Authorization: Bearer <token>`

#### Scenario: No token omits header
- **WHEN** a request is sent while no token is stored
- **THEN** the outgoing request omits the `Authorization` header

### Requirement: Response auto-unwrapping
A response interceptor SHALL unwrap the `R<T>` envelope: it SHALL resolve with `data` when `code == 0`, and reject with the envelope's `msg` when `code != 0`, surfacing the error via a toast.

#### Scenario: Successful response unwraps data
- **WHEN** the backend returns `{ code: 0, msg, data }`
- **THEN** the calling code receives `data` directly

#### Scenario: Business error surfaces a toast
- **WHEN** the backend returns `{ code: <non-zero>, msg }`
- **THEN** a toast displays `msg` and the promise rejects

### Requirement: Unauthorized redirect
When the backend signals an unauthenticated state (token invalid/expired), the interceptor SHALL clear the stored token and redirect to the login page.

#### Scenario: Invalid token redirects to login
- **WHEN** the backend returns a not-authenticated result
- **THEN** the stored token is cleared and the router navigates to the login page

### Requirement: Route permission guard
A global route guard SHALL redirect unauthenticated users attempting to reach protected routes to the login page, and SHALL load dynamic routes from the server menu after a successful login.

#### Scenario: Unauthenticated access redirects
- **WHEN** an unauthenticated user opens a protected route
- **THEN** the guard redirects to the login page

#### Scenario: Post-login dynamic routes
- **WHEN** a user logs in successfully
- **THEN** the guard fetches the server menu and registers the corresponding dynamic routes before navigation
