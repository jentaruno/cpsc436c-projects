# How Fast You’ll Beat
HowLongToBeat.com has global averages of a game’s completion time, but they’re often a little off when you actually play it yourself. This API predicts a user’s completion time for a video game based on their past completion data.

## 1. User & Decision: 
Video game players can predict how much time they’ll need to complete a video game, or how long their friends will need if they’re gifting a game. From that data, they can decide if and when it’s worth it to buy/start playing.

## 2. Target & Horizon: 
Hours to complete a video game 

## 3. Features (No Leakage):
- Predict the completion time for main quests, main + side quests, and completionist playthroughs
- Visualize distribution of other players’ times to complete the game, and where you stand
- Avoid leakage: Ensure golden rule (test data doesn’t influence training data). Ensure stats that are eventually logged in the future, in the database deployed to production, doesn’t influence the predictions.

## 4. Baseline → Model Plan 
- Baseline model: Always predict the global average from HowLongToBeat.com. If the model doesn’t predict it better than the global averages, then it’s not good enough.
- Collaborative filtering; good to use this model because gameplay data is usually sparse, and games are very unique. They’re difficult to categorise into classes or parameters that predict completion time. Collaborative filtering allows us to look at players who played the same games and make a prediction by comparing their play time data to our player’s data, and filling in the blanks.

## 5. Metrics, SLA, and Cost 
- Metrics: MAE (Mean Absolute Error)
Absolute errors between predicted vs actual hours to beat a game. Chosen because this is less sensitive to outlier data (speedrunners and idlers). It’s also very interpretable as an error to evaluate.
- SLA (Service-Level Agreement): p95 < 200ms per prediction
Cost: < $0-0.1 per 10K predictions, remaining on the free tier for normal load and paying when there are surges

## 6. API Sketch

### 6.1. Endpoints
Note: user management endpoints (register, login, logout, get profile details, delete account) are omitted for simplicity.
| Method | Path                          | Purpose                                                                 | Auth Needed |
|--------|-------------------------------|-------------------------------------------------------------------------|-------------|
| GET    | /v1/users/me/playtime/         | Retrieve all of the user’s saved game completion/playtime data                 | Bearer      |
| POST   | /v1/users/me/playtime         | Add a new game completion/playtime record                               | Bearer      |
| PUT    | /v1/users/me/playtime/{id}    | Update an existing playtime record (e.g., hours, completion status)     | Bearer      |
| DELETE | /v1/users/me/playtime/{id}    | Remove a playtime record                                                | Bearer      |
| GET    | /v1/predictions/{gameId}      | Predict completion time for the given game based on user’s play history | Bearer      |

Rate limits?

### 6.2. Request/response examples

#### a. GET /v1/users/me/playtime  
Request  
```
GET /v1/users/me/playtime  
Authorization: Bearer <token>
```
Response (200 OK)  
```
[
    {
        "id": "b8f6e1a2-1234-4f8a-9c77-3b9a2f73a0cd",
        "game_id": "elden-ring",
        "title": "Elden Ring",
        "completion_time": 120,
    },
    {
        "id": "d3f2c4e9-5678-4ab9-81aa-9b24de55f0ab",
        "game_id": "hollow-knight",
        "title": "Hollow Knight",
        "completion_time": 35,
    }
]
```

#### b. POST /v1/users/me/playtime
Request  
```  
POST /v1/users/me/playtime  
Authorization: Bearer <token>  
Content-Type: application/json
{
    "game_id": "hollow-knight-silksong",
    "completion_time": 42,
}
```
Response (201 Created)  
```
{
    "id": "e12ab345-6789-40de-98f3-b12345cd6789",
    "game_id": "hollow-knight-silksong",
    "title": "Hollow Knight: Silksong",
    "completion_time": 42,
}
```

#### c. PUT /v1/users/me/playtime/{id}
Request  
```
PUT /v1/users/me/playtime/d3f2c4e9-5678-4ab9-81aa-9b24de55f0ab
Authorization: Bearer <token>
Content-Type: application/json
{
    "completion_time": 50,
}
```
Response (200 OK)
```
{
    "id": "d3f2c4e9-5678-4ab9-81aa-9b24de55f0ab",
    "game_id": "hollow-knight",
    "title": "Hollow Knight",
    "completion_time": 50,
}
```

#### d. DELETE /v1/users/me/playtime/{id}
Request  
```
DELETE /v1/users/me/playtime/e12ab345-6789-40de-98f3-b12345cd6789
Authorization: Bearer <token>
```
Response (204 No Content)    
```
(no body)
```

#### e. GET /v1/predictions/{id}
Request  
```
GET /v1/predictions/silksong  
Authorization: Bearer <token\
```
Response (200 OK)  
```
{
    "game_id": "silksong",
    "title": "Hollow Knight: Silksong",
    "predicted_hours": 42,
    "confidence": 0.87,
}
```

### 6.3. Auth scheme
- Type: HTTP
- Scheme: Bearer
- Header: Authorization
- Format: Authorization: Bearer \<token\>
- Server returns token on login/register. Client stores token and sends it for protected endpoints. Server validates the token on every such request, and either accepts or rejects with 401 Unauthorized.  

Example:
```
GET /v1/users/me/playtime
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```


## 7. Privacy, Ethics, Reciprocity
Full Privacy Impact Assessment (PIA) [here](/pia.md)
- Data inventory:
  - User playtime records (ID, game ID, completion time)
  - Prediction requests (game ID, timestamp, model version)
  - Auth tokens (ephemeral JWTs only, no passwords)
  - Minimal telemetry: request counts, latency, error codes (no IPs/UA stored)
- Purpose limitation:
  - Playtime → personalized predictions
  - Telemetry → reliability (debugging, capacity planning, SLA validation)
  - No secondary use (e.g., advertising/profiling)
- Retention:
  - Playtime records: until user deletes account/data
  - Predictions: ephemeral (≤ 7 days for debugging, then dropped)
  - Telemetry raw logs: TTL ≤ 14 days; aggregates 90–180 days
- Access:
  - Role-based, least-privilege
  - Engineers see aggregates
  - Individual records only accessible to the user

## 8. Architecture Sketch
<img src="architecture.png" alt="architecture" width="300"/>

## 9. Risks & Mitigations

#### a. Service scalability / downtime during spikes
Sudden spikes (e.g., popular game release, Steam Summer Sale) could overwhelm Lambda/API Gateway, causing errors or slow responses. Several mitigations:
- Use serverless architecture (Lambda + API Gateway) with autoscaling.
- Implement rate limiting and caching to smooth bursts.
- Monitor SLA metrics (p95 latency, error rates) and pre-warm Lambda functions if needed.

Testing / Validation:
- Conduct load tests simulating 10x normal traffic.
- Measure p95 latency and error rates; tune concurrency limits, caching, and retry policies.

#### b. Model bias / inaccurate predictions
Collaborative filtering may over- or under-estimate completion times for users who play differently than the majority, e.g., very fast or casual players. Several mitigations:
- Implement baseline model comparison: always check that collaborative filtering improves on global averages.
- Flag predictions with low confidence (e.g., few similar games in user history).

Testing / Validation:
- Use a test set with holdout users and compute RMSE/MAE to evaluate accuracy.
- Simulate edge cases (fast/completionist vs casual players) and review model outputs.

#### c. Re-identification of users from playtime data
No IP, UA, or location collected, but very unique combinations of completion_time across games could theoretically identify a user if leaked or joined with external data. Several mitigations:
- Enforce k-anonymity for any public or aggregate stats (e.g., only publish averages if ≥10 users).
- Truncate completion times to nearest hour.
- Keep telemetry fully aggregated, avoid storing identifiers.

Testing / Validation:
- Perform synthetic re-identification tests: attempt to link a “fake user” record to real records in your dataset.
- Periodically audit published aggregates to ensure no small groups (<10 users) are exposed.

## 10. Measurement Plan
- MAE  
  Run the baseline model vs the collaborative filtering model on a test set. Compute improvements in percentages of MAE
- SLA  
  Measure API response times under realistic load; p50, p95, and max latency

##  11. Evolution & Evidence
- Link a git hash (or range/tag) that shows the design’s evolution (commits, README updates, diagrams).
- Insight memo link (3 insights), assumption audit, and Socratic log references.

---

*For further information on the metrics used, see [Glossary](https://canvas.ubc.ca/courses/168892/pages/glossary-and-acronyms-course)*