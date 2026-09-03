## Authentication

Our authentication process closely resembles the one shown in the "reporting-api" section of 135-site. We utilize bcrypt to compare raw passwords against the stored hash to validate user credentials. Session information, however, is stored in our database under a "users" table via "express-mysql-session" package. We opted for this design versus a server-specific session storage due to the fact that we have two nodes that require session information, auth (session creation, user validation) and reporting (REST endpoint for analytics data). This REST endpoint included POST operations that were dangerous to expose without authentication so it made sense to wrap both the analytics dashboard and analytics API under the same authentication/session procedure. From what we can tell, it functions similarly to "basic" express sessioning except with this session information being stored externally in a shared database. This session information is persistant and can be referenced by any service that requires the information. 

### Flow:
1. Apache routes requests by path: everything except `/api/*` goes to `auth`, and `/api/*` goes to `reporting`. This lets both services share one public domain.
2. A user submits a username/email and password to `auth` through the login form.
3. `auth` looks up the user and checks the password against the stored bcrypt hash.
4. If it matches, `auth` creates a session, saves it in the shared `sessions` table (linked to that user's ID), and sends the browser a signed session cookie.
5. On every later request, the browser sends that cookie back. Both `auth` and `reporting` check the same shared session table to see who's logged in, since they're separate processes reading from the same database.
6. If a request has no valid session, `auth` redirects the browser to the login page, and `reporting`'s API returns an authentication error instead, since it isn't a page a browser navigates to directly.

