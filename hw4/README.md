## Part 1: Authentication

Our authentication process closely resembles the one shown in the "reporting-api" section of 135-site. We utilize bcrypt to compare raw passwords against the stored hash to validate user credentials. Session information, however, is stored in our database under a "users" table via "express-mysql-session" package. We opted for this design versus a server-specific session storage due to the fact that we have two nodes that require session information, auth (session creation, user validation) and reporting (REST endpoint for analytics data). This REST endpoint included POST operations that were dangerous to expose without authentication so it made sense to wrap both the analytics dashboard and analytics API under the same authentication/session procedure. From what we can tell, it functions similarly to "basic" express sessioning except with this session information being stored externally in a shared database. This session information is persistant and can be referenced by any service that requires the information. 

### Flow:
1. Apache routes requests by path: everything except `/api/*` goes to `auth`, and `/api/*` goes to `reporting`. This lets both services share one public domain.
2. A user submits a username/email and password to `auth` through the login form.
3. `auth` looks up the user and checks the password against the stored bcrypt hash.
4. If it matches, `auth` creates a session, saves it in the shared `sessions` table (linked to that user's ID), and sends the browser a signed session cookie.
5. On every later request, the browser sends that cookie back. Both `auth` and `reporting` check the same shared session table to see who's logged in, since they're separate processes reading from the same database.
6. If a request has no valid session, `auth` redirects the browser to the login page, and `reporting`'s API returns an authentication error instead, since it isn't a page a browser navigates to directly.

## Part 2: User Management

### Admin Login:

**Username:** grader_admin
**Password:** grader_admin

### User Login:

**Username:** grader_user
**Password:** grader_user

## Part 3: Dashboard

It was a bit tough to figure out what data should be focused on. From what we learned in class, we should start with a queston about our users, determine what data is necessary to answer that question, collect such data, and finally visualize the collected relevant data. Unfortunately we don't have any real users so we have to answer the most "generally useful" questions we can think of. Under this vague heuristic, we decided on 3 main focus points: *"Where are our users coming from?"*, *"When are they visiting?"*, and *"How do they experience our site?"*. For each of these questions we created an associated main visual tool using Chart.js:

#### "Where are our users coming from?"
*Visitors Over Time (Graph) (1 week, 2hr bins)*

We fetch `/api/events` for the last 7 days and bucket it into 2-hour windows, counting unique `sessionId`s per bucket rather than raw event counts so repeat activity from one visitor doesn't inflate the numbers. Chart.js renders this as a line chart, with only the first point of each day labeled so the axis doesn't get crowded across all 84 points.

#### "When are they visiting?"
*Visitor Locations (Pie Chart) (1 week, 10 country max)*

`collector` looks up each visitor's country from their IP using `geoip-lite`, a free offline IP-to-country database that needs no API key or account, and stores it alongside their session data. The dashboard groups the last week of sessions by country and charts the top 10 individually, collapsing anything past that into a single "Other" slice so it stays readable.

#### "How do they experience our site?"
*Browser capabilities (Bar Chart) (1 weel, 4 capabilities w/ presence amongst all users 0-100%)*

`collector.js` runs simple client-side checks (`navigator.cookieEnabled`, a test image load, etc.) to detect whether cookies, JavaScript, images, and CSS actually work in each visitor's browser. The dashboard calculates what percentage of the last week's sessions had each capability enabled and charts it as one bar per capability.

To supplement this data, allow easy testing, and to show recent activity we included tables for recent events and sessions at the bottom of the page. 



