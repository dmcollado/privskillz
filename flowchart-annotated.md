# Health Tracker — Annotated Flowchart Example

This shows a fully annotated PrivGraph diagram and the expected rule evaluation results.

## Architecture

```mermaid
flowchart TB
    %% @pg:meta owner "platform-team"
    %% @pg:meta system "health-tracker"
    %% @pg:meta reviewed "2024-01-15"

    subgraph UserDevices["User Devices (Untrusted)"]
        Mobile[Mobile App]
        Web[Web App]
    end

    subgraph Backend["Backend Services (AWS us-east-1)"]
        API[API Gateway]
        Auth[Auth Service]
        UserSvc[User Service]
        HealthSvc[Health Service]
    end

    subgraph DataStores["Data Stores"]
        UserDB[(User Database)]
        HealthDB[(Health Records)]
        Cache[(Redis Cache)]
    end

    subgraph ThirdParty["Third Parties"]
        Analytics[Analytics Platform]
        EmailSvc[Email Service]
    end

    Mobile --> API
    Web --> API
    API --> Auth
    API --> UserSvc
    API --> HealthSvc
    UserSvc --> UserDB
    HealthSvc --> HealthDB
    UserSvc --> Cache
    HealthSvc --> Analytics
    UserSvc --> EmailSvc

    %% Data Classifications
    %% @pg:data-class Mobile-->API DIRECT_ID, PHI, LOCATION
    %% @pg:data-class Web-->API DIRECT_ID, PHI
    %% @pg:data-class API-->Auth CREDENTIALS
    %% @pg:data-class API-->UserSvc DIRECT_ID
    %% @pg:data-class API-->HealthSvc PHI
    %% @pg:data-class UserSvc-->UserDB DIRECT_ID
    %% @pg:data-class HealthSvc-->HealthDB PHI
    %% @pg:data-class UserSvc-->Cache INDIRECT_ID
    %% @pg:data-class HealthSvc-->Analytics PHI
    %% @pg:data-class UserSvc-->EmailSvc DIRECT_ID

    %% Trust Boundaries
    %% @pg:boundary Mobile-->API user-device
    %% @pg:boundary Web-->API user-device
    %% @pg:boundary HealthSvc-->Analytics third-party
    %% @pg:boundary UserSvc-->EmailSvc third-party

    %% Controls — Nodes
    %% @pg:control UserDB encrypted-at-rest, access-controlled, audit-logged
    %% @pg:control HealthDB encrypted-at-rest, access-controlled, audit-logged, retention-policy
    %% @pg:control Cache encrypted-at-rest, access-controlled
    %% @pg:control Cache !retention-policy
    %% @pg:control Analytics dpa-in-place
    %% @pg:control EmailSvc dpa-in-place

    %% Controls — Flows
    %% @pg:control Mobile-->API encrypted-in-transit
    %% @pg:control Web-->API encrypted-in-transit
    %% @pg:control API-->Auth encrypted-in-transit
    %% @pg:control API-->UserSvc encrypted-in-transit
    %% @pg:control API-->HealthSvc encrypted-in-transit
    %% @pg:control HealthSvc-->Analytics encrypted-in-transit
    %% @pg:control UserSvc-->EmailSvc encrypted-in-transit

    %% Compliance
    %% @pg:compliance HealthDB HIPAA, GDPR
    %% @pg:compliance UserDB GDPR, CCPA
    %% @pg:compliance HealthSvc-->Analytics HIPAA

    %% Accepted Risks
    %% @pg:risk-accept Cache PG-003 "Session data only, 24hr TTL enforced at Redis level. Approved by security@ 2024-01-10"
```

## Expected Rule Evaluation

**Active findings:**

🟡 MEDIUM — PG-009: GDPR Storage Without Retention Policy
- Node: UserDB
- UserDB has GDPR compliance scope but no retention-policy control.
- → Add: `%% @pg:control UserDB retention-policy`

**Suppressed findings (1):**

- PG-003 on Cache — suppressed by risk-accept (session data, 24hr TTL)

**All other rules pass.** The diagram has full encryption in transit for all sensitive flows, encryption at rest for all stores, DPAs for third parties, audit logging for HIPAA-scoped components, and retention policy for the health database.
