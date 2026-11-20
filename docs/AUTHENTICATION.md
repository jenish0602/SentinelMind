# Authentication and Authorization Architecture

This document describes the implementation in the supplied SentinelMind Java ZIP, not a generic JWT design.

## Components
- `RegistrationServlet` (`/register`): validates user input; hashes the submitted registration access code using SHA-256; uses a database transaction and `SELECT ... FOR UPDATE` to check whether the code is valid, active, unexpired, and below its usage limit; assigns its associated role; stores a BCrypt password hash (work factor 12); increments code usage and commits.
- `LoginServlet` (`/login`): queries `users` joined to `roles` with a JDBC prepared statement; checks account status; verifies the supplied password using `BCrypt.checkpw`; replaces the previous HTTP session; adds user and role attributes; then redirects to `/dashboard`.
- `AuthenticationFilter`: protects mapped routes by requiring a session containing `userId`; otherwise redirects to the login page. It also adds some cache and browser safety headers.
- `AuthorizationFilter`: checks session role against ADMIN, ANALYST, USER, and restricts selected routes and actions. Admin routes require ADMIN; analyst routes accept ADMIN or ANALYST; selected writes are admin-only.
- `LogoutServlet`: closes the monitoring session if possible, invalidates the HTTP session, redirects to login.
- `ActivityTrackingFilter` and `MonitoringDAO`: track page activity and monitored sessions. Login hashes the raw HTTP session ID with SHA-256 before passing it to the monitoring database layer.

## Request flow
1. Browser opens `/login.jsp` or `GET /login`.
2. User submits email and password to `POST /login`.
3. `LoginServlet` reads a matching active user via JDBC; checks the BCrypt password hash.
4. On success, it invalidates any previous session, creates/renews an HTTP session, and sets `userId`, `fullName`, `email`, `roleId`, and `roleName`.
5. Browser accesses protected application routes with Tomcat's standard session cookie.
6. Authentication and authorization filters check session state and permissions.
7. Logout invalidates the session and clears the user-side authenticated state.

## Credentials and tokens
- **Passwords:** not stored in plaintext by the registration servlet; its BCrypt hash is saved as `users.password_hash`. Login verifies against that hash.
- **Registration codes:** their SHA-256 digests are matched against `access_codes.code_hash`. Unlike password hashing, plain SHA-256 on low-entropy access codes is susceptible to offline guessing if the database leaks; use long random codes.
- **Session identifiers:** the browser uses the server-managed servlet session cookie; the application does not issue a JWT or OAuth access token for login. The monitoring subsystem receives a SHA-256 digest of the session ID rather than the raw session ID.
- **Database credentials:** JDBC loads `SM_DB_URL`, `SM_DB_USER`, and `SM_DB_PASSWORD` from environment variables when available. Review Hibernate settings separately.
- **Avoid committing:** real passwords, real database exports, session cookies, and live registration codes.

## Security limitations and improvements
- No JWT refresh-token or OAuth2 flow appears in this implementation.
- Confirm all sensitive endpoints are covered by authentication/authorization filters, including newly added endpoints.
- Add CSRF protection for state-changing actions.
- Use HTTPS and appropriate Secure, HttpOnly and SameSite cookie settings in deployment.
- Add throttling/lockout for repeated failed login attempts. Although a `failed_login_attempts` column exists, the examined login servlet does not increment it on failure.
- Rotate demo access codes before deploying publicly.
- Keep application server secrets out of Git history and production logs.

## Primary code references
`src/main/java/com/sentinelmind/web/{LoginServlet,RegistrationServlet,LogoutServlet,AuthorizationFilter}.java`, `src/main/java/com/sentinelmind/filter/{AuthenticationFilter,ActivityTrackingFilter}.java`, and `src/main/java/com/sentinelmind/config/DBConnection.java`.
