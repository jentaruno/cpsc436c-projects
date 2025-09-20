# Privacy Impact Assessment

### Overview

#### Purpose
Video game players can predict how much time they’ll need to complete a video game, or how long their friends will need if they’re gifting a game. From that data, they can decide if and when it’s worth it to buy/start playing.

#### Scope
- Predictive model (baseline average + collaborative filtering) to generate personalized completion estimates.
- Backend hosted on serverless infrastructure (Lambda + API Gateway) with scalable telemetry.
- Privacy-focused: minimal data collection, strong access controls, transparent retention.

#### Data processing summary

Collection:
- User-submitted playtime data (id, game ID, title, completion time).
- Prediction requests (game ID, timestamp, model version).
- Telemetry (latency, error rates, aggregate usage counts).
- No collection of IPs, user agents, or unnecessary identifiers.

Processing
- Predictions generated via collaborative filtering and baseline models.
- Aggregation of telemetry for SLA monitoring and capacity planning.
- Anonymization and k-anonymity for any community insights.

Sharing
- Predictions returned only to the requesting user.
- Aggregated metrics (e.g., average completion times) may be shared publicly, never tied to individuals.
- No sale or third-party sharing of identifiable data.

### Data Inventory

- **id** (UUID, unique playtime record identifier)  
  - Purpose: Uniquely identify a playtime record for CRUD operations  
  - Lawful basis: Contract (service provision)  
  - Minimization: Random UUID only; not linkable to real identity  
  - Retention: Until user deletes record  
  - Access roles: User (self), backend service  

- **game_id** (canonical identifier, e.g., "elden-ring")  
  - Purpose: Link playtime to a specific game; input for predictions  
  - Lawful basis: Contract  
  - Minimization: Only game_id stored; no platform/account linkage  
  - Retention: Until user deletes record  
  - Access roles: User (self), backend service  

- **title** (human-readable name of the game, e.g., "Elden Ring")  
  - Purpose: User-friendly display in library  
  - Lawful basis: Contract  
  - Minimization: Derived from game_id; not strictly required but improves UX  
  - Retention: Until user deletes record  
  - Access roles: User (self), backend service  

- **completion_time** (numeric, hours played to complete the game)  
  - Purpose: Core input for ML prediction  
  - Lawful basis: Contract  
  - Minimization: Only total hours; no fine-grained play session details  
  - Retention: Until user deletes record  
  - Access roles: User (self), backend service, ML model (aggregates only)  

- **prediction request metadata** (game ID, timestamp, model version)  
  - Purpose: Debug predictions; monitor model evolution  
  - Lawful basis: Legitimate interest (service reliability)  
  - Minimization: No user ID in logs; ephemeral only  
  - Retention: ≤ 7 days  
  - Access roles: Backend service, engineers (debug only)  

- **telemetry: request latency (p95/p99)**  
  - Purpose: SLA monitoring and capacity planning  
  - Lawful basis: Legitimate interest  
  - Minimization: Aggregate metrics only  
  - Retention: Raw ≤ 14 days; aggregates 90–180 days  
  - Access roles: Engineers (aggregates only)  

- **telemetry: error codes (4xx/5xx counts)**  
  - Purpose: Debugging, reliability tracking  
  - Lawful basis: Legitimate interest  
  - Minimization: Aggregate only; no payload/body storage  
  - Retention: Raw ≤ 14 days; aggregates 90–180 days  
  - Access roles: Engineers (aggregates only)  

### Linkability & Identifiability
- Events can be linked across sessions for the same user, they can make an account. Their playtime records and predictions are linked to their account ID.
- Events can't be linked across users.
- Session tokens resolve to account ID. All records are keyed on that ID
- None of the fields collected are quasi-identifiers (no IP/location/device leak).

### Purpose Limitation & Secondary Use
Purposes
- Primary purpose: Deliver game completion time predictions for users, based on their provided play history.
- Secondary purposes (explicit + limited): 
  - Service reliability (capacity planning, scaling, anomaly detection).
  - Debugging (short-term logs, ≤14d retention).
- Excluded purposes: No use for advertising, profiling outside predictions, or resale of data.

Approving new purposes
1. Proposal: Explain the new data collection, why it's needed, and alternatives to achieve the same outcome with less data.
2. Review: Evaluate lawful basis, privacy risks, and alignment with original user expectations.
3. Decision: If the new purpose is within scope, it can be approved. If it's outside scope, it requires explicit user opt-in and updated disclosures.

### Minimization & Retention

Minimization
- Telemetry stored only as aggregate counts per hour/day. No raw user identifiers kept in telemetry aggregates.
- Completion times truncated to nearest hour. Timestamps truncated to day.
- If IDs of any sort ever appear in telemetry, they are hashed with a salted hash to prevent re-linking to accounts.
- No direct identifiers (IP/email/Steam ID) collected or stored in analytics.

Retention strategy
- User playtime records: Retained as long as user maintains account. Deleted when user chooses to delete the record, or account is deleted.
- Raw logs (infra/debug): TTL 7-14 days, auto-purged.
- Telemetry aggregates: 90-180 days retention for trend analysis and capacity planning.

### Access Control & Governance

User accounts
- Access limited to their own id + associated playtime records.
- Auth via bearer token; no cross-user data visibility.

Engineering / Operations staff
- Read/write access limited by role:
- Developers → access to staging/test data only (no prod).
- Ops/SRE → read-only production metrics/telemetry; no direct user data access.

Admin (break-glass only)
- Time-limited elevated rights, logged and reviewed.

AWS management
- Lambda, API Gateway, DynamoDB, S3, CloudWatch. No other third parties (ad networks/analytics SDKs)
- IAM roles scoped per service (Lambda, DynamoDB, S3). API Gateway limited to required integrations only. No wildcard (*) permissions in policies.
- All privileged access requests logged through CloudTrail / IAM Access Analyzer

### Transparency & Choice
Privacy policy; on the website, let the user know that:
- The service collects aggregated, anonymized telemetry to maintain service quality. 
- Individual predictions are never linked publicly. 
- Predictions are only visible to the user who submitted the data. - Option to delete or edit playtime data at any time.
- Account deletion removes all personal playtime records.

### Security
The API faces potential threats like abuse (excessive requests), scraping of predictions, and doxxing attempts. To mitigate these, we implement:
- Rate limiting via API Gateway, request throttling, and per-user quotas to prevent overuse.
- Web Application Firewall (WAF) filters suspicious traffic.
- Telemetry timestamps or request identifiers are jittered to prevent correlation attacks. 
- All user inputs are validated and sanitized to reduce injection or enumeration risks.
- All sensitive credentials (API keys, database passwords) are stored securely in a managed secrets manager (e.g., AWS Secrets Manager) and rotated regularly. 
- All API communications are enforced over HTTPS/TLS.
- Bearer tokens are validated per request to ensure secure authentication. 
- No sensitive data is transmitted in query parameters or logs.

### Compliance & Policy Alignment
- Collect only necessary data, provide deletion rights, maintain transparency via privacy policy.
- Maintain transparency about data usage, retention, and opt-in mechanisms.
- Follow least-privilege access and internal change management processes.
- Conduct regular internal audits of telemetry, access logs, and data retention.

### Residual Risks & Trade-offs
- Re-identification from rare playtime patterns
  - Business/Ethical Rationale: Even with anonymization, extremely unique combinations of game completions could theoretically link back to a user. 
  - Contingency Measures: Limit public data to aggregated stats (≥10 users), truncate completion times, audit datasets for uniqueness before release.
  - Tradeoff: Needs a minimum number of recorded playtimes to be able to show 
- Abuse or scraping of API
  - Business/Ethical Rationale: Excessive or malicious requests could drive costs up and compromise system integrity.
  - Contingency Measures: API rate limits, WAF, anomaly detection, token validation, and throttling policies.

 > Attach telemetry decision matrix and any diagrams that clarify data flows.