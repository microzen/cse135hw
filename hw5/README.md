## Technical Document

#### Links

Repo: https://github.com/BenKMichael/benyezhi-site
Monitored Site: https://test.benyezhi.site
Report Site: https://reporting.benyezhi.site
Landing Site: https://benyezhi.site

#### AI Use:

AI-use scaled directly proportionally to perceived work/time density. In other words, the closer we were to the deadline and the more work we had to do, the more we ended up booting up our LLMs. For week 5 this was particularly apparent since we were preoccupied with week 4 homework, the final, and having to travel to see family after the end of the quarter. AI guidance was used to assist in the refactor (Mentioned in GRADER.MD) and a lot for chart implementation and front-end HTML/CSS in .ejs views. Unfortunately we were in a position where we likely couldn’t have completed week 5 assignment satisfactorily without LLMs given time-constraints. We did notice verbose and questionable design decisions made by our LLMs that required significant supervision at times. We acknowledge that LLM generated code is definitely “McDonalds” but between starving to death (imminent deadline) and a Big Mac, we chose the carefully monitored Big Mac.

#### Architecture:

We opted for a server-side MVC due to relatively small data size and mostly static views. We retained the front-facing apache server with a singular node.js express node running all controllers, views, and models. A MySQL database is hosted at a separate port that is only accessed via the models listed in src/models. Database and node utilize a reverse-proxy so all incoming requests and outgoing responses pass through the apache server. 

#### Security:

Login checks a username or email and password against the users table, with passwords hashed using bcrypt so nothing is ever stored or compared in plain text. A small role system controls what each logged in user can see and do, and it is checked on every request instead of just at login. Sessions live in a signed cookie holding only a user id, marked so scripts cannot read it and so it will not be sent along with requests from other sites. Every database query is parameterized, so user input cannot change the structure of a query. Most input is also validated for format and length before use, though coverage is not perfectly even across every field. Anything rendered back to a page goes through the template engine's automatic escaping, and report data sent to the browser is escaped again before being embedded, so stored HTML or scripts should not execute. Production secrets are kept out of the repo and set through our deploy process rather than the example config used for local development. Overall, session cookies guard against hijacking, input sanitation guards against SQL-injections, and text escaping guards against XSS attacks.

#### Timeline:

We didn’t have much time to map out what we wanted to implement as we were more worried with getting requirements done, but we did discuss and back down on the idea of having a more dynamic report-creation page. We wanted to allow the user to choose from a list of chart types and select data to use to construct it to allow better customization of reports. This was, unfortunately, a bit too complicated for us to scope out so we opted for static charts with space for analyst comments as a compromise. Were this an actual page with real users, this would have been a bare minimum implementation. We would have also liked to add more “types” of analysts with access to different report options. This would have been a reasonable thing to implement given we had more time and would probably be the first thing we would have worked on after tightening up our working implementation.
