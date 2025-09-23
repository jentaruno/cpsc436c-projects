## Top 3 insights

1. **Lambda's reliability in handling normal load and viral spikes**
   - Evidence: See [assumption audit](assumption_audit.md)
   - Why it matters: Using Fargate only would've cost $0.03/mo - twice the price. Using Lambda only is free for up to 1 million requests per month, and it autoscales for ease of maintenance. Choosing the right cloud solution matters a lot for money and convenience.
   - Limits: Did not consider Azure or Google Cloud alternatives here.
   - Next question: How would Azure or Google Cloud pricing structure compare?

2. **Even with sparse data, we can still choose a good ML model**
   - Evidence: See [overview](overview.md)
   - Why it matters: The nature of the dataset will be sparse, because there are so many games out there, and most people play only a small fraction of them. Collaborative filtering allows us to make good predictions with that limitation.
   - Limits: Assumed that the model would make its calculations in <100ms (SLA).
   - Next question: If we run the model locally, what would its performance look like? What about on the cloud?

3. **Use k-anonymity on aggregations to protect users from re-identification** 
   - Evidence: See [Socratic Log](socratic_log.md) and [PIA](pia.md)
   - Why it matters: This service is principally a personalized prediction API, so users likely don't expect to see their completion times on a public leaderboard. Moreover, even if I think it’s low risk, regulators often require showing that I assessed and mitigated re-identification risk
   - Limits: Assumption; I'm taking ChatGPT's word for the legal compliance requirement here, and just erring on the side of caution.
   - Next question: Should we differentiate k by who is accessing it? (e.g. engineers vs public)

---

No attachments because this project is no-code