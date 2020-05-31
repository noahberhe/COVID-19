# COVID-19
NLP applied to recent Coronavirus / COVID19 tweets: How has sentiment progressed in recent weeks?

# Aim:
- Scrape London-based Tweets related to #Coronavirus
- Apply Vader to run Sentiment Analysis
- Chart Sentiment over the period vs. Virus: How have Londoners' attitudes changed as the virus progressed?
- Who were the winners / losers of the lockdown period?

# Approach:
- Retrieve tweets, apply post-processing using spacy and vader to parse entities, and score for sentiment.
- Import case statistics and compare with sentiment to try to model the trend
- Conduct Hypothesis tests to look for winners / losers this way. Visualise the data through wordclouds, also use wordclouds to drive new hypotheses.
- Improve on this non-rigorous, piecemeal approach by modelling winners / losers holistically using Logistic Regression
- Use the Logistic Regression as the basis to then analyse each winner / loser: by moving to a Bayesian beta-binomial model and animating the progression over time of the posterior beta distribution

# Results:
# Lockdown: The Winners and Losers
- __Amazon__: as we mentioned, it started off in neutral territory: 0.5. But jumped into positive territory almost immediately after the lockdown, and stayed.
- __Cummings__: Very interesting to see, e.g. you can run this for Cummings and see how his stock has fallen over the lockdown period. This had already started by a week into the lockdown, which perhaps shows the prior belief (informed by tweets prior to lockdown) was not in line with what we would come to later understand of public feeling towards him. In the last few days, with the story that he broke the lockdown, the posterior makes a dramatic shift to the left, crossing over into <0.5 territory and rapidly becoming peaked at ~0.4
- __SocialDistancing__: Very quickly became positive, and stayed positive, showing the focused mood of a nation.
- __Government__: no prior belief about views on the Government, however after immediately seeing positive sentiment in the lockdown period, we started to see that normalise back towards 0.5 and settling just above that level. Mixed views here, started strong but faded.
- __Keir Starmer__: has been a big Loser from the Coronavirus period, with many Labour supporters unhappy at his support of the government's approach, and of course those on Conservative side bound to be against him.

# Conclusion
- __In conclusion__, we showed that it's possible to set up a feed in from Twitter, apply some text processing, conduct sentiment scoring, and analyse the per-entity sentiment, in a way that clarifies the winners / losers.
- __Hypothesis Testing__: used this extensively to understand bring some statistical clarity to my findings, which taught me a few areas to be cautious about:
    - __p-values__: can be easily made to be statistically significant if the sample size if large enough to ensure a low enough standard error, ask yourself if your test is specific enough to ask a pointed question? Ask yourself if your filtering is set up correctly, ensuring your sample isn't inflated.
    - __normality assumption__: this generally held in most cases, thanks to the Central Limit Theorem, but we can always use non-parametric versions of our hypothesis tests like Mann-Whitney or Wilcoxon Signed-Rank tests.
    - __mean reversion / neutralisation__: for any slice of the data that's large enough / not specific enough, sentiment would generally cluster around 0 and not be meaningful. View the distribution of your subsample, and if it has a high peak at 0 sentiment then likely you need to be more specific with filtering for a meaningful hypothesis test.
- __Holistic Winners/Losers__: To get an overall view, I used Logistic Regression to model positive vs negative labelled tweets, using the vectorized entity column as the predictor matrix. This gave some interesting results:
    - The overall $R^2$ was low, suggesting I needed more features than the few I included to be able to model accurately whether a tweet would be positive or negative.
    - However, this wasn't what I wanted... I was interested in how those features I did include contribute to positivity or negativity, which I could get from their coefficients. The nice things about using the Statsmodel package vs sklearn's was being able to see the statistical confidence of each coefficient.
    - So because we actually want to solve for $P(p \;|\; data)$ not $P(data \;|\; p)$, and also because we have a constantly evolving view of $P(p \;|\; data)$ we want to follow, we turned to Bayesian approach:
- __Bayesian Winners/Losers__: Here I set up an animated slider to show the prior distribution, and then how the posterior distribution varied after that as new data came in, which could be configured to the entity of choice.
    - This showed clearly how/when Cummings and Starmer sentiment started to deteriorate, giving some good explanation.
    - It also clearly showed certainty of our posterior beliefs, by attaching a probability distribution to the sentiment at each stage.
    - The Beta-Binomial model was used which simplified the parameters greatly, a good way to view this conjugacy is that the binomial distribution is just the discrete version of the beta ditribution, and therefore simply reflects the addition of individual new data-points to the continuous prior. This is solved mathematically as the update of the beta parameter by addition of the binomial parameter.

# Future enhancements
- This has hope of being modularized such that it's generalizable as a Twitter monitor, which has the below capabilities:
    - New tweets come in, they are post-processed using spacy and vader.
    - We are able to conduct some hypothesis tests on sections of the data.
    - We can provide a holistic view through Logistic Regression
    - We can view Bayesian Updating in an animation for a chosen entity.
- I also want to learn a bit more about spacy, and see if it's possible to train the Entity Recognizer better such that any capitalized word isn't mistaken for an entity
