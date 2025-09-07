# Webknot Campus Drive Assignment — README


## 1. Problem Understanding
The whole idea of this platform is to make college events easier to manage. On one side, the college staff can create events like workshops, hackathons, seminars, or even cultural fests. On the other side, students can check what events are coming up, register if they’re interested, and mark their attendance when they actually attend.
The reporting part is needed so that we don’t just create events but also understand how they went. For example, we should be able to see how many students registered, how many showed up, what kind of feedback students gave, which events were more popular, and also how actively each student is taking part in different events.

## 2. My Approach & Decisions
My approach was to first figure out the main things the system should handle: events, students, registrations, attendance, and feedback. These are the core parts that need to be tracked.
I decided to keep one database but separate each college’s data using a college_id. This makes it simple to manage while still supporting many colleges. For event IDs, I thought keeping them unique everywhere is better, so there is no confusion later.
I also considered a few common cases, like students trying to register more than once, events getting cancelled, or feedback not being given. These points helped me decide the rules for the system.
Finally, I kept the design straightforward. The APIs focus only on the main actions (create event, register, mark attendance, give feedback). Reports can then be made easily from this data without making things complicated.

## 3. What I Designed
I designed the system around a few simple parts. First, there is a database structure that stores events, students, registrations, attendance, and feedback. These tables are connected so we can easily track who joined which event, who actually attended, and what feedback was given.
Next, I thought about the APIs. They are just simple actions:
Create an event
Register a student
Mark attendance
Collect feedback
Generate reports
Then, I imagined the workflow. A student sees an event → registers → attends the event → gives feedback. On the admin side, reports can be generated to check registrations, attendance percentage, popularity, and student participation.

## 4. How to Navigate this Repo 
The repository is organized in a straightforward way so everything is easy to find:
README.md → My own explanation of the project, my approach, and my learnings.
DesignDocument.md → The main design details like database schema, APIs, workflows, and queries.
Reports.md → The report queries and outputs listed separately for quick access.
AI-Logs/ → Folder where I kept screenshots of my AI brainstorming conversations.
Submission.md → Quick checklist for how to submit the assignment.

## 5. Assumptions & Edge Cases
Each student can only register once for an event.
If an event is cancelled, all registrations for it are automatically cancelled.
Only students who attend can give feedback, and only one feedback per student per event.
Attendance is allowed only for confirmed registrations (not for waitlisted or cancelled).
If the event is full, new students can only join the waitlist (if waitlist is allowed).
If a registered student cancels, the first person on the waitlist is promoted to confirmed.
Feedback is optional, so some events may not have ratings.
Event IDs are unique across all colleges.
Students are identified uniquely within their college using their roll number or student ID.
## 6. Learnings
While working on this design, I understood how important it is to break a big problem into smaller parts like events, students, registrations, and feedback. It made the system easier to design and explain.
I also learned to think about edge cases early, like duplicate registrations or cancelled events, because they can affect the whole system later if not planned properly.
Another learning was how reports are connected to the way data is stored. If the database is designed neatly, reports like attendance percentage or top active students can be generated easily.
Overall, this assignment gave me a better idea of how real systems are planned before coding and how keeping things simple is often the best approach.

## AI Usage

I used AI to brainstorm ideas for the database, APIs, and report queries.  
I didn’t copy anything directly — I rewrote all explanations, workflows, and queries in my own words.  
I also added my own design decisions, like unique IDs, attendance rules, and multi-college handling. is this good
