## MODIFIED Requirements

### Requirement: Axios instance with token injection
A central Axios instance SHALL prepend the backend base URL to every request and inject the current token into the `Authorization: Bearer <token>` header when a token is stored. On the server side, the token SHALL be read from the request cookies via `cookies()`. On the client side, requests requiring authentication SHALL be proxied through a Next.js Route Handler that reads the cookie and injects the token.

#### Scenario: Token header is attached (server-side)
- **WHEN** a Server Component sends a request while a token is stored in the cookie
- **THEN** the server-side axios instance reads the token from `cookies()` and the outgoing request carries `Authorization: Bearer <token>`

#### Scenario: No token omits header
- **WHEN** a request is sent while no token is stored in the cookie
- **THEN** the outgoing request omits the `Authorization` header

### Requirement: Unauthorized redirect
When the backend signals an unauthenticated state (token invalid/expired), the interceptor SHALL clear the authentication cookies and redirect to the login page. On the server side, this SHALL be handled via middleware or Server Component redirect; on the client side, via the Route Handler response.

#### Scenario: Invalid token redirects to login
- **WHEN** the backend returns a not-authenticated result
- **THEN** the `imooc_token` and `imooc_user_id` cookies are cleared and the user is redirected to the login page

### Requirement: Route permission guard
A Next.js `middleware.ts` SHALL intercept navigation to protected routes and redirect unauthenticated users to the login page. After a successful login, dynamic routes SHALL be loaded from the server menu and registered. The middleware SHALL check for the presence of the `imooc_token` cookie to determine authentication status.

#### Scenario: Unauthenticated access redirects
- **WHEN** an unauthenticated user (no `imooc_token` cookie) opens a protected route
- **THEN** the middleware redirects to the login page with a `redirect` query parameter

#### Scenario: Authenticated access proceeds
- **WHEN** an authenticated user (valid `imooc_token` cookie) opens a protected route
- **THEN** the middleware allows the request to proceed to the page

#### Scenario: Post-login dynamic routes
- **WHEN** a user logs in successfully
- **THEN** the guard fetches the server menu and registers the corresponding dynamic routes before navigation
