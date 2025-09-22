#### SLA
- Test: Synthetic probes (e.g., k6/Artillery) + CloudWatch p95 latency metric. 
- Accept: p95 < 200 ms under normal load (50 requests per second); < 1s under spike (500 RPS)

#### Baseline beats trivial:
– Test method: on a 1k-row sample, compare:  
    - Trivial baseline: always predict HowLongToBeat.com global average.  
    - Rule baseline: collaborative filtering model (matrix factorization or KNN).
    - Compute MAE and RMSE.
– Accept: model baseline improves MAE by ≥ 15% over global average.

#### No leakage:
– Test method: Feature timeline check. Ensure all training features (user’s past playtimes, game metadata) are available before prediction time. Exclude fields like “completion_time” for the target game.
– Accept: 0 features sourced from post-prediction data.

#### Privacy guardrails: k-anonymity
- Test method: Make requests for games with varying numbers of completion times recorded, check response
- Accept: Requests for games with k<10 completion time aggregates return a bucketed range, and games with k≥10 return an integer

#### Cost envelope
- Test method: AWS calculator for API Gateway + Lambda at target RPS; add 30% buffer
- Accept: ≤ $10 per 10K predictions at expected traffic
- Result: 293.16/131400000 