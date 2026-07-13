# LIVE Creator Growth & Segmentation

An end-to-end analytics project that segments livestreaming creators, predicts which ones are likely to break out before it shows up in revenue, and turns both into a set of decisions a growth/ops team could actually act on.

**[View the live dashboard](#https://sara-iqbal.github.io/LIVE-Creator-Growth-Segmentation/)** · Analysis notebook: `tiktok_live_creator_growth_segmentation.ipynb` · Dashboard: `dashboard.html`

![dashboard preview](docs/preview.png)

---

## The problem

Platforms that run livestreaming programs face the same recurring question: *out of thousands of creators, who deserves attention right now?*

Some creators are already thriving and just need to be retained. Some are quietly building momentum and would benefit from early support but by the time that shows up in their revenue or follower count, the window to help them accelerate has often already passed. Others are drifting away and about to churn. Treating all creators the same wastes time on people who don't need it and misses the people who do.

I wanted to answer three concrete questions with data instead of gut feel:

1. **What natural groups do creators fall into**, based on how they actually behave not by title or vertical, but by their engagement, growth, and monetization patterns?
2. **Can we flag "high-potential" creators *before* they're obviously successful**, using only the signals available early, so a team could intervene while it still matters?
3. **Where should recruitment effort go** which content categories have the best combination of growth, monetization, and headroom?

## The data

Since I didn't have access to a real livestreaming platform's internal data, I generated a **synthetic dataset of 2,500 creators** in Python, designed to behave like a real one:

- Behavioral metrics: live sessions, average session length, concurrent viewers, watch time, comments per session
- Growth metrics: new followers, follower base, follower growth rate
- Monetization metrics: unique gifters, gifting revenue, repeat-gifter rate
- Context: content vertical, region, account tenure, community guideline strikes

The generator doesn't just draw random numbers it builds in realistic relationships (e.g. more tenured creators tend to have larger, steadier audiences; certain content categories monetize better than others; new creators have higher variance) so that the patterns a model finds are the kind of patterns that would plausibly show up in a real dataset. All data is fabricated; no real platform, creator, or user data is used anywhere in this project.

## How I approached it

**1. RFM-style scoring.** I adapted the classic Recency/Frequency/Monetary framework from e-commerce to livestreaming: recency became days since last stream, frequency became sessions in the last 28 days, and monetary became estimated gifting revenue. Each dimension is scored into quintiles and combined into a single RFM score per creator.

**2. Segmentation with K-Means.** I clustered creators on their RFM scores plus growth rate, reach, and gifting loyalty, then inspected each cluster's actual behavior profile before naming it rather than assuming what the clusters would look like ahead of time. This produced five segments: Star Creators, Rising Talent, High-Reach Veterans, Steady Core, and At Risk/Dormant.

**3. A classifier for early potential.** This was the most important design decision in the project: the model is trained only on *leading* indicators (growth rate, engagement, gifting breadth and loyalty) and deliberately excludes *lagging* ones like current revenue or RFM score. A model that just learns "creators who already make money are high-potential" isn't useful it's just restating the present. Predicting who's about to succeed, using only what's visible before they succeed, is the actual problem worth solving. I used a Random Forest classifier, evaluated with a held-out test set, and inspected feature importances to sanity-check that the model was picking up genuine behavioral signal rather than noise.

**4. A leading vs. lagging indicator framework.** For each illustrative growth objective (e.g. "grow the creator base," "improve monetization health"), I paired a leading indicator a team could monitor weekly with a lagging indicator that confirms the result later. This is a small piece of analytics strategy, not just modeling it's the difference between a dashboard people check after the fact and one that changes what they do this week.

**5. Content vertical opportunity scoring.** I ranked content categories on growth rate, monetization, and share of high-potential creators, discounted by how saturated (crowded) each category already is so the recommendation isn't just "whatever's biggest," but where the room to grow actually is.

**6. A growth funnel**, from registered creator down to Star Creator, to make dropout points visible at a glance.

Everything was built in **Python** (pandas, scikit-learn) in a Colab notebook, then exported to a JSON file that feeds a static **HTML/JavaScript dashboard** (Chart.js) so the results are viewable without needing to run any code.

## What I found

- The five segments separate cleanly and tell different stories: Rising Talent creators have the highest follower growth rate of any segment despite the shortest tenure exactly the group an early-warning system needs to catch.
- The leading-indicator-only classifier reached a strong ROC-AUC on held-out data, with follower growth rate, tenure, and comment engagement as the strongest predictors meaning early behavioral signals really do carry information about future trajectory, not just correlate with already-being-successful.
- Content verticals differ meaningfully in "opportunity": some categories combine high growth and monetization with relatively low saturation, making them better recruitment targets than their raw creator count would suggest.

## What I'd do differently with real data

- Validate the "high-potential" label against actual future outcomes (e.g. did the creator reach a revenue threshold 90 days later) instead of a constructed proxy target.
- Test whether segments and the model hold up across regions and languages, since behavior likely varies more than a single synthetic distribution can capture.
- Add time-series features (trend over multiple windows, not just a single 28-day snapshot) a lot of "leading indicator" value comes from the *direction* a metric is moving, not just its current level.

## What I learned

- **The most important modeling decision isn't the algorithm it's the target.** Excluding lagging indicators from the classifier's features was a small change that turned the model from a restatement of current success into an actual early-warning tool. It's easy to build a model with great accuracy that's secretly useless because it's allowed to see the answer.
- **Segmentation is only useful if the labels are grounded in the data, not assumed.** I named clusters after inspecting their real averages rather than deciding upfront what five segments "should" look like a couple of my initial guesses about which cluster would be which were wrong.
- **A good metric needs a counterpart.** A leading indicator without a lagging one to check it against is just a guess; a lagging indicator without a leading one gives you no time to react. Pairing them explicitly made the OKR framework far more actionable than a flat list of metrics would have been.
- **Building the dashboard forced clearer thinking than the notebook did.** Deciding what deserved a headline number versus a chart versus a table meant re-asking, for every result, "what decision does this actually inform?" which caught a couple of metrics that were interesting but not actually useful.

## Tech stack

`Python` · `pandas` · `NumPy` · `scikit-learn` (KMeans, Random Forest) · `Google Colab` · `HTML/CSS/JavaScript` · `Chart.js`

## Repo structure

```
├── tiktok_live_creator_growth_segmentation.ipynb   # full analysis, runs top-to-bottom in Colab
├── dashboard.html                                   # standalone dashboard (data embedded, no server needed)
├── dashboard_data.json                               # data export consumed by the dashboard
├── creators_synthetic_dataset.csv                    # full synthetic dataset with all engineered features
└── README.md
```

To view the dashboard, either open `dashboard.html` directly in a browser or enable GitHub Pages on this repo and visit the published link. To reproduce or extend the analysis, open the notebook in Google Colab and run all cells it regenerates the dataset, retrains the model, and re-exports `dashboard_data.json`.
