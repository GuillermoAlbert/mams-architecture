# 3 — Domain Model (ERD)

The core domain in simplified form — key entities and relationships only, not every column.
A **global user identity** owns a portable **player profile**; tenancy and staffing flow
through join tables (`organization_members`, `team_staff`, `team_players`). All daily
metrics (`daily_wellness`, `session_rpe`, `body_pain_log`) are keyed by athlete and
`record_date`, which is the axis that joins them. A training session is optional: RPE can
be submitted with no session and linked later.

```mermaid
erDiagram
    USER ||--o| PLAYER_PROFILE : has
    USER ||--o{ ORGANIZATION_MEMBER : "is member"
    ORGANIZATION ||--o{ ORGANIZATION_MEMBER : has
    ORGANIZATION ||--o{ TEAM : owns
    USER ||--o{ TEAM_STAFF : staffs
    TEAM ||--o{ TEAM_STAFF : has
    TEAM ||--o{ TEAM_PLAYER : rosters
    PLAYER_PROFILE ||--o{ TEAM_PLAYER : "enrolled via"
    TEAM ||--o{ TRAINING_SESSION : schedules
    USER ||--o{ DAILY_WELLNESS : submits
    USER ||--o{ SESSION_RPE : submits
    TRAINING_SESSION ||--o{ SESSION_RPE : "optionally links"
    USER ||--o{ BODY_PAIN_LOG : submits
    PLAYER_PROFILE ||--o{ INJURY : records

    USER {
        uuid id PK
        string email
    }
    PLAYER_PROFILE {
        uuid id PK
        uuid user_id FK
    }
    ORGANIZATION {
        uuid id PK
        string name
    }
    ORGANIZATION_MEMBER {
        uuid org_id FK
        uuid user_id FK
        string role
    }
    TEAM {
        uuid id PK
        uuid org_id FK
    }
    TEAM_STAFF {
        uuid team_id FK
        uuid user_id FK
        string role
    }
    TEAM_PLAYER {
        uuid team_id FK
        uuid player_id FK
        string status
    }
    TRAINING_SESSION {
        uuid id PK
        uuid team_id FK
        date scheduled_date
        int duration_minutes
        string session_type
    }
    DAILY_WELLNESS {
        uuid id PK
        uuid user_id FK
        date record_date
    }
    SESSION_RPE {
        uuid id PK
        uuid user_id FK
        uuid session_id FK "nullable"
        date record_date
        int rpe
    }
    BODY_PAIN_LOG {
        uuid id PK
        uuid user_id FK
        date record_date
        string body_part
    }
    INJURY {
        uuid id PK
        uuid player_id FK
        date onset_date
    }
```

**Key constraints**
- `daily_wellness` — one row per athlete per day.
- `body_pain_log` — one row per athlete per day per body part.
- `session_rpe` — up to two per athlete per day; `session_id` is nullable.
- Team training load (`duration_minutes × avg RPE`) is computable only when a session exists; individual metrics are always available.
