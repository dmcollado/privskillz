# OAuth Login — Annotated Sequence Diagram Example

## Login Flow

```mermaid
sequenceDiagram
    %% @pg:meta owner "auth-team"
    %% @pg:meta system "oauth-login"

    actor User
    participant Browser
    participant API as API Gateway
    participant Auth as Auth Service
    participant IDP as External IdP
    participant DB as User Database

    User->>Browser: Enter credentials
    Browser->>API: POST /login (email, password)
    API->>Auth: Validate credentials
    Auth->>IDP: OAuth token exchange
    IDP-->>Auth: Access token + ID token
    Auth->>DB: Lookup/create user record
    DB-->>Auth: User profile
    Auth-->>API: Session token
    API-->>Browser: Set-Cookie (session)
    Browser-->>User: Login success

    %% Data Classifications
    %% @pg:data-class User->>Browser CREDENTIALS
    %% @pg:data-class Browser->>API CREDENTIALS, DIRECT_ID
    %% @pg:data-class API->>Auth CREDENTIALS, DIRECT_ID
    %% @pg:data-class Auth->>IDP CREDENTIALS
    %% @pg:data-class IDP-->>Auth CREDENTIALS
    %% @pg:data-class Auth->>DB DIRECT_ID
    %% @pg:data-class DB-->>Auth DIRECT_ID
    %% @pg:data-class Auth-->>API INDIRECT_ID
    %% @pg:data-class API-->>Browser INDIRECT_ID

    %% Trust Boundaries
    %% @pg:boundary User->>Browser user-device
    %% @pg:boundary Browser->>API public-internet
    %% @pg:boundary Auth->>IDP third-party

    %% Controls
    %% @pg:control Browser->>API encrypted-in-transit
    %% @pg:control API->>Auth encrypted-in-transit
    %% @pg:control Auth->>IDP encrypted-in-transit
    %% @pg:control IDP-->>Auth encrypted-in-transit
    %% @pg:control Auth->>DB encrypted-in-transit
    %% @pg:control DB encrypted-at-rest, access-controlled, audit-logged
    %% @pg:control Auth access-controlled, audit-logged
    %% @pg:control IDP dpa-in-place

    %% Compliance
    %% @pg:compliance DB GDPR, SOC2
    %% @pg:compliance Auth SOC2
```

## Expected Rule Evaluation

**Active findings:**

🟡 HIGH — PG-004: Credentials Without Encryption
- Flow: User → Browser
- Credentials entered on user device without documented encryption.
- → This is typically acceptable (local input) — consider a risk-accept:
  `%% @pg:risk-accept User->>Browser PG-004 "Local user input on device, not a network transfer"`

🟡 MEDIUM — PG-009: GDPR Storage Without Retention Policy
- Node: DB
- GDPR-scoped user database lacks retention policy.
- → Add: `%% @pg:control DB retention-policy`

🟡 MEDIUM — PG-003: Missing Retention Policy
- Node: DB
- Stores DIRECT_ID without retention policy.
- → Same fix as above covers this.

**All other rules pass.** Third-party IdP has a DPA, all network flows are encrypted, and audit logging is in place.
