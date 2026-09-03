## Names + Site
Benjamin Michael, Yezhi Wu
https://reporting.benyezhi.site/

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
*Browser capabilities (Bar Chart) (1 week, 4 capabilities w/ presence amongst all users 0-100%)*

`collector.js` runs simple client-side checks (`navigator.cookieEnabled`, a test image load, etc.) to detect whether cookies, JavaScript, images, and CSS actually work in each visitor's browser. The dashboard calculates what percentage of the last week's sessions had each capability enabled and charts it as one bar per capability.

To supplement this data, allow easy testing, and to show recent activity we included tables for recent events and sessions at the bottom of the page. 

## Part 4: Report

#### Guiding Question:
*"Should we implement 'graceful degradation' behavior on our website in case users do not have JavaScript enabled or experience issues with JS related features?"*

#### Conclusion:

Yes (Assuming collected data was valid and from real users). The primary data point we collected was the whether or not the user has JS enabled/working on their client. We did this via a tracking pixel embedded in every page in test.benyezhi.site (via noscript + img + GET request). This gave us a "JS capability" flag associated with each session. This data point along with cookie, image, and CSS capabilities across all users, was aggregated into a barchart. The ratio of users that can even render our JS served data is the most important piece of data we can collect regarding our guiding question which is given by the height of a capabilities corresponding height (0-100%). The actual data, however, is fudged, due to our lack of users as well as testing during development, but if we took this data at face value, only about 30-40% of our users can properly render the JS on our website which would be a strong argument for more robust noscript handling and general "graceful degradation" guided design. Client side activity vs script errors was a secondary data point used to capture if users encountered JS related issues given they could run JS in the first place. The ratio of activity vs errors gives a sense of how many issues are occuring vs the interactions occuring on the site.





