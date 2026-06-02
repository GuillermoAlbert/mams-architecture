# 2 — Multi-Level Permission Model

Authorization is multi-tenant with the **organization** as the tenant boundary, resolved
across two independent membership layers. The **organization layer** grants org-wide roles
(owner/admin); the **team layer** grants per-team staff roles (trainer/physio/analyst).
A user's effective access is the union of both. Athletes are not staff: they reach a team
through roster enrollment and can only ever see their own data.

```mermaid
graph TD
    ORG["Organization<br/><i>(tenant boundary)</i>"]

    subgraph OrgLayer["Layer 1 — Organization membership"]
        OM["organization_members<br/>ROLE_OWNER · ROLE_ADMIN"]
    end

    subgraph TeamLayer["Layer 2 — Team staffing"]
        TS["team_staff<br/>ROLE_TRAINER · ROLE_PHYSIO · ROLE_ANALYST"]
    end

    TEAM["Team"]
    ROSTER["team_players<br/>PENDING → ACTIVE → INACTIVE"]
    USER["User account<br/><i>(global identity)</i>"]
    PLAYER["Player profile<br/><i>(belongs to the athlete, not the club)</i>"]

    ORG --> OM
    ORG --> TEAM
    TEAM --> TS
    TEAM --> ROSTER

    USER --> OM
    USER --> TS
    PLAYER --> ROSTER

    OM -. "org-wide access to all teams" .-> TEAM
    TS -. "scoped access to one team" .-> TEAM
    ROSTER -. "own data only" .-> PLAYER
```

**Effective access**
- **Owner / Admin** — full access to every team in the organization.
- **Trainer** — full management of their team (sessions, imports, all data).
- **Physio** — read-only clinical view (injuries, body-map, wellness).
- **Analyst** — read-only performance view (RPE, sessions).
- **Athlete** — own data only; a single account can be enrolled in multiple teams.
