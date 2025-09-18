# How Fast You’ll Beat
HowLongToBeat.com has global averages of a game’s completion time, but they’re often a little off when you actually play it yourself. This API predicts a user’s completion time for a video game based on their past completion data.

### User & Decision: 
Video game players can predict how much time they’ll need to complete a video game, or how long their friends will need if they’re gifting a game. From that data, they can decide if and when it’s worth it to buy/start playing.

### Target & Horizon: 
Hours to complete a video game 

### Features (No Leakage):
- Predict the completion time for main quests, main + side quests, and completionist playthroughs
- Visualize distribution of other players’ times to complete the game, and where you stand
- Avoid leakage: Ensure golden rule (test data doesn’t influence training data). Ensure stats that are eventually logged in the future, in the database deployed to production, doesn’t influence the predictions.

### Baseline → Model Plan 
- Baseline model: Always predict the global average from HowLongToBeat.com. If the model doesn’t predict it better than the global averages, then it’s not good enough.
- Collaborative filtering; good to use this model because gameplay data is usually sparse, and games are very unique. They’re difficult to categorise into classes or parameters that predict completion time. Collaborative filtering allows us to look at players who played the same games and make a prediction by comparing their play time data to our player’s data, and filling in the blanks.

### Metrics, SLA, and Cost 
- Metrics: MAE (Mean Absolute Error)
Absolute errors between predicted vs actual hours to beat a game. Chosen because this is less sensitive to outlier data (speedrunners and idlers). It’s also very interpretable as an error to evaluate.
- SLA (Service-Level Agreement): p95 < 200ms per prediction
Cost: < $0-0.1 per 10K predictions, remaining on the free tier for normal load and paying when there are surges

### Measurement Plan
- MAE
Run the baseline model vs the collaborative filtering model on a test set. Compute improvements in percentages of MAE
- SLA
Measure API response times under realistic load; p50, p95, and max latency

### Ethics
Users can add, edit, or remove any game completion time entries on their own will anytime
No hidden monitoring

For further information on the metrics used, see [Glossary](https://canvas.ubc.ca/courses/168892/pages/glossary-and-acronyms-course)
