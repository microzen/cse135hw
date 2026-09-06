## Grader Document

#### Login

Username: grader_admin
Password: grader_admin

Username: grader_analyst
Password: grader_analyst

Username: grader_viewer
Password: grader_viewer

#### Scenario

1. Login as grader_admin
2. Navigate to Users page
3. Add User
   - Username: example
   - Email: example@example.com
   - Password: example123
   - Role: Analyst
4. Logout
5. Login as example
6. Navigate to Create Report page
7. Select Audience & Technology
8. Edit -> “Hello World!” -> Save for all Analyst Notes fields
9. Save Report
10. Navigate to Reports page
11. Download recently created Report
12. Logout
13. Login as grader_admin
14. Navigate to Reports
15. Delete recently created Report
16. Navigate to Users
17. Delete example user

#### Discussion

Overall we’re roughly satisfied with how the reporting page/s turned out. As previously mentioned, time-constraints were a huge issue for us due to family/course related obligations so there wasn’t much of an opportunity for us to really stress test this implementation with E2E, unit, etc. Regardless, we believe we fulfilled the requirements for the assignment although we can’t say it has all the bells and whistles we were hoping for when we started.

Regarding architecture, we realized a bit too late that the structure of our servers was way too complex for what we were serving. We had a node for auth, api, collector, and a leftover node from week 2’s assignment. To trim the fat, we made a pretty significant rework of our codebase to squash everything down into a single node that handled all services. We restructured into a more typical app/ node.js project directory format that also cleanly translated into the MVC-guided design professor wanted from us. From there, it was a lot easier to implement new features although we’re not 100% it was worth the effort considering the existing time constraints. 

Regarding bugs, we managed to squash most of what we could find through manual testing of the live-implementation but this was by no means rigorous. If the grader does find bugs in our implementation we unfortunately were not aware of it at time of submission. 

Regarding security, we managed to find a few security issues that presented XSS risks that were caught while looking over our codebase (Raw db query in script block). We’re not aware of any SQL injection/Session hijacking vulnerabilities at the time of submission, but again, we were too squashed against the clock to be certain no common security vulnerabilities exist. Particularly in terms of sessioning, we would have liked to put more effort in.

Regarding final implementation, we’re satisfied with the create -> read -> delete flow for full reports but we didn’t put as much effort as we would have liked into report internals. We defaulted to bar/hbar charts for most of our data representation which was a safe bet considering most of our dataviz handled quantitative vs qualitative features. We tried to spice it up with some pie charts and plugged in a histogram and line/scatter chart for a few of the report types. Our concern is that some of these charts look extremely sparse which truthfully reflects the sparseness of our actual data. We’re also aware that certain charts like the screen dimension scatter plot poorly represent concentrated data which would probably be better displayed with alternative dataviz. Unfortunately we did not have enough time to tinker with this detail in favor of getting the full feature running end-to-end. We also did not manage to implement much no-script behavior. Report page, for example, should render without JS but without download feature which we warn users of when attempted.

Regarding repo health, we kinda dropped the ball towards the end with huge commits directly to main. We did a lot of implementation testing using docker so a lot of development progress was documented locally. Docker was not used for actual server deployment as we never really had the flexibility to set it up so our github actions workflow was limited to pulling site-related files onto the server and launching all nodes via pm2. We had to manually tinker with apache settings when needed which was another reason we favored local development during the last-assignment time crunch.
