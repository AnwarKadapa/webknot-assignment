## Campus Event Management — Reporting Module (Design Document)

Author: Kadapa Anwar
College: REVA University
Date: 2025-09-07

## 1 Scope & Goals

This system is meant to help colleges manage events and easily generate simple reports.

Admins can create events like workshops, fests, or hackathons.

Students can browse events, register, and check in.

Reports will cover popularity, attendance %, student participation, and average feedback.

This is just a design document, so the focus is on structure — data, APIs, workflows, and report queries — not on coding.

## 2 Data to Track

We need to track:

Events: name, type, timing, capacity, status.

Students: details like name, roll no, email, branch, year.

Registrations: status (Confirmed, Waitlisted, Cancelled) and timestamps.

Attendance: who checked in/out, method (QR/manual).

Feedback: rating (1–5) and optional comments (only from attendees).

## 3 Multi-Tenant Design & Scale Assumptions

The platform is for multiple colleges (~50), each with ~500 students and ~20 events per semester.

Single database: separate data using college_id in all tables.

Event IDs: globally unique (UUID) to avoid confusion.

Optional: human-friendly event_code unique within each college.

This keeps queries simple, indexing fast, and the design ready to scale if more colleges are added later.

## 4 ER Diagram (Mermaid)
erDiagram
    COLLEGE ||--o{{ STUDENT : has
    COLLEGE ||--o{{ EVENT : organizes
    STUDENT ||--o{{ REGISTRATION : registers
    EVENT ||--o{{ REGISTRATION : has
    STUDENT ||--o{{ ATTENDANCE : checks_in
    EVENT ||--o{{ ATTENDANCE : records
    STUDENT ||--o{{ FEEDBACK : submits
    EVENT ||--o{{ FEEDBACK : receives

    COLLEGE { 
      string college_id PK
      string name
    }

    STUDENT { 
      string student_id PK
      string college_id FK
      string roll_no
      string name
      string email
      string phone
      string branch
      int year
      timestamp created_at
    }

    EVENT {
      string event_id PK
      string college_id FK
      string name
      string type
      timestamp start_time
      timestamp end_time
      string location
      int capacity
      bool allow_waitlist
      string status
      string created_by
      timestamp created_at
    }

    REGISTRATION {
      string reg_id PK
      string event_id FK
      string student_id FK
      timestamp registered_at
      string status
      string source
    }

    ATTENDANCE {
      string att_id PK
      string event_id FK
      string student_id FK
      timestamp check_in_at
      timestamp check_out_at
      string method
      string marked_by
    }

    FEEDBACK {
      string feedback_id PK
      string event_id FK
      string student_id FK
      int rating
      string comment
      timestamp submitted_at
    }

## 5 Table Sketch & Key Constraints

STUDENT: no duplicate roll numbers per college (UNIQUE(college_id, roll_no)).

REGISTRATION: prevent duplicate registrations (UNIQUE(event_id, student_id)).

FEEDBACK: only one per student per event (UNIQUE(event_id, student_id)).

ATTENDANCE: one per event per student (or last check-in counts).

Capacity rule: Confirmed registrations ≤ capacity. Extra → Waitlist if allowed.

## 6 API Design (Specs Only)

Event Management

POST /events → create event

GET /events?college_id=…&type=…&status=… → list events

PATCH /events/{event_id} → update status

Registration

POST /events/{event_id}/register → register student (auto waitlist if full)

DELETE /events/{event_id}/register/{student_id} → cancel registration

Attendance

POST /events/{event_id}/attendance/check-in → mark check-in

POST /events/{event_id}/attendance/check-out → optional

Feedback

POST /events/{event_id}/feedback → submit rating/comment (only if attended)

Reports

GET /reports/popularity?college_id=…

GET /reports/participation?student_id=…

GET /reports/event-summary?event_id=…

Optional: top-active-students, filter by event type

## 7 Core Workflows (Mermaid Sequence)

7.1 Registration

sequenceDiagram
    autonumber
    participant U as Student
    participant API as Backend
    participant DB as Database

    U->>API: Register(event_id, student_id)
    API->>DB: Check duplicates
    DB-->>API: Not found
    API->>DB: Count confirmed vs capacity
    DB-->>API: count
    alt Capacity available
        API->>DB: Insert REGISTRATION(status=Confirmed)
        API-->>U: Confirmed
    else Waitlist allowed
        API->>DB: Insert REGISTRATION(status=Waitlisted)
        API-->>U: Added to Waitlist
    else Full
        API-->>U: Registration rejected
    end


7.2 Attendance (QR)

sequenceDiagram
    participant U as Student
    participant API as Backend
    participant DB as Database
    U->>API: Scan QR
    API->>DB: Verify registration
    API->>DB: Upsert ATTENDANCE(check_in_at=now)
    API-->>U: Check-in successful


7.3 Reporting

sequenceDiagram
    participant A as Admin
    participant API as Backend
    participant DB as Database
    A->>API: Request Event Summary(event_id)
    API->>DB: Run queries
    DB-->>API: Aggregated results
    API-->>A: Summary JSON/CSV

## 8 Edge Cases & Rules

Reject duplicate registrations.

Cancelled events → registrations auto-cancel; attendance/feedback locked.

Feedback only from attendees; one per student/event.

Attendance only for confirmed registrants.

Waitlist auto-promotion when seat frees.

Check-in only within event hours.

Unique email/phone per student per college.

Restrict cross-college data access using college_id.

## 9 Report Queries (SQL Sketch)

Event Popularity → count confirmed registrations

Attendance % → % of registered students who attended

Average Feedback → average rating per event

Student Participation → events attended per student

Top N Active Students → most active students

Filter by Event Type → reports filtered by type


## 10 Non-Functional Notes

Indices: (college_id, event_id), (college_id, roll_no), (event_id, student_id)

Privacy: filter by college_id in all queries

Portability: SQL works for SQLite/Postgres

Extensibility: can add ORGANIZER table later

## 11 Appendix — Status Codes

Event.status: Draft | Open | Closed | Cancelled

Registration.status: Confirmed | Waitlisted | Cancelled

Attendance.method: QR | Manual
