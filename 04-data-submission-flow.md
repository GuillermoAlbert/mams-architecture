# 4 — Data Submission Flow

How a daily metric travels from athlete to database, and how late-created sessions reconcile
with already-submitted RPE. Submissions are **idempotent**: each metric is uniquely keyed by
athlete and `record_date` (and body part, for pain logs), so re-submitting updates the
existing row instead of duplicating it — an upsert. Because `session_id` is nullable, an
athlete can log RPE before any session exists; when staff create the session later, the
backend links orphan RPE rows by matching date.

```mermaid
sequenceDiagram
    actor Athlete
    participant SPA as React SPA
    participant API as REST API
    participant SVC as Service Layer
    participant DB as PostgreSQL

    Note over Athlete,DB: 1 — Athlete submits a daily metric (no session yet)
    Athlete->>SPA: Fill wellness / RPE / body-pain form
    SPA->>API: POST metric (+ JWT)
    API->>API: Authorize (role) & validate payload
    API->>SVC: submit(metric, record_date)
    SVC->>DB: SELECT existing row by (user, record_date[, body_part])
    alt row exists
        SVC->>DB: UPDATE existing row (idempotent)
    else no row
        SVC->>DB: INSERT new row (session_id = NULL)
    end
    DB-->>SVC: persisted row
    SVC-->>SPA: 200 OK

    Note over Athlete,DB: 2 — Staff create the session afterwards
    actor Staff
    Staff->>SPA: Create training session (date, duration)
    SPA->>API: POST session (+ JWT)
    API->>SVC: createSession(team, date)
    SVC->>DB: INSERT training_session
    SVC->>DB: UPDATE orphan RPE rows SET session_id WHERE record_date matches
    DB-->>SVC: linked rows
    SVC-->>SPA: 201 Created (team load now computable)
```

**Notes**
- Idempotency comes from the database's uniqueness keys, not application-side dedup logic.
- Linking is by `record_date`, so the order of athlete vs. staff submission does not matter.
- Bulk staff imports follow the same service path, so they obey the same upsert and linking rules.
