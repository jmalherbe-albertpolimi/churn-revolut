# Oral — 7 minutes (≈ 3 min 30 each) + 4 minutes of questions

Support: the notebook `churn_revolut.ipynb`, scrolled section by section. ⏱ = cumulative time.

---

## HÉLOÏSE — Problem, data, test design (≈ 3 min 30)

**[Section 1 — Framing]** ⏱ 0:00
"The retention team at Revolut can send a costly offer to a limited number of customers each month. Our job is to tell them **who should get it**, and whether it is worth it.
The decision is simple: sending one offer earns on average *probability of leaving × probability the offer keeps them × what the customer is worth*, minus the cost of the offer. We send it only when that is positive.
Our offer: **5% cashback on card payments for 30 days, capped at 2 euros** — churners stop using their card, so the offer pushes exactly the behaviour of customers who stay, and its cost is capped. We also test 1 and 5 euros. Its success rate is not in the data, because **no offer was ever sent**: we test 20, 30 and 50%."

**[Section 2 — Audit]** ⏱ 0:50
"Before modelling, we audited the data. Three decisions: we capped amounts, because some transfers reach 85 billion dollars; we dropped `plan`, because it is the plan at export time — a **leak**; and we kept zero-dollar transactions as a failure signal."

**[Section 3 — Churn and split]** ⏱ 1:20
"We defined churn as: active in the 60 days before a cutoff, no transaction in the 60 days after.
*« With 60 days, 26% of labelled churners come back within a month. With 45 days, it's 31%. We keep 60 days, and this false-churn rate is an accepted limit. »*
We split in time: train on November 2018, tune on January 2019, test on March 2019, **opened once**. The cutoffs are 60 days apart, so no label overlaps the next part.
We also exclude customers younger than two months: there is no signup after January 3rd, so the test set has none, and they churn twice as much."

**[Section 6 — What goes with churn]** ⏱ 2:30
"What goes with churn? Disengagement first: no transaction for 19 days or more means 41% churn, against 2% for someone active the day before. Then narrow use: one merchant country, few categories, no contacts. Customers who use Revolut as their main account stay. These are associations, not causes."
→ *hand over to Julien* ⏱ 3:30

---

## JULIEN — Models, result in euros, recommendation (≈ 3 min 30)

**[Section 7 — Models]** ⏱ 3:30
"We compared a one-line baseline — rank by days since the last transaction — with a logistic regression and boosted trees. Metric: precision-recall AUC and churners caught in the 500 riskiest customers, because with 13% churn accuracy means nothing.
The trees win on validation, and on **three different periods**, so it is not luck. They are well calibrated, which matters because our decision rule uses the probability directly."

**[Section 9 — Test]** ⏱ 4:30
"On the test, opened once: trees catch **297 churners in the top 500**, against 246 for the baseline and 65 at random. The model is stable.
But the key result is in euros. Targeting the riskiest customers **loses money in every scenario**: they are dormant customers who earn almost nothing. Targeting the biggest spenders already does better. And our profitability rule — risk times customer value, only when it pays — is **the only strategy positive in all nine scenarios**. For a 2-euro offer that keeps 30% of customers, we estimate **450 to 1,340 euros per month** — validation and test — so we give a range, not a promise."

**[Section 10 — Where it fails]** ⏱ 5:40
"Where does it fail? It catches customers who fade out, but only **4 to 5% of churners who were still active**. The most expensive misses used their account days before leaving: nothing in the transactions announced it. On the other hand, false alarms cost nothing: they are dormant, zero-value customers, so the rule never sends them an offer."

**[Section 11 — Recommendation]** ⏱ 6:20
"Our recommendation: **ship under condition**. Use the profitability rule with the 2-euro cashback offer — in practice, it goes only to high-value customers at risk, mostly in the UK. In the first month: an A/B test with a control group to measure the real success rate — if it is below break-even, we stop — a calibration check, and monthly retraining."
⏱ 7:00

---

## Likely questions (both must be able to answer all of them)

| Question | Answer |
|---|---|
| Why 60 days? | False churn: 26% come back within a month at 60 days, 31% at 45, 39% at 30. 90 days is cleaner but pushes every cutoff back. |
| Why cutoffs 60 days apart? | Otherwise the training label is not yet known on the validation decision date: the model would have seen part of the answers. |
| When did you open the test? | Section 9, once, after freezing every choice (listed at the top of the section). |
| Why not accuracy? | 13% churn: predicting "nobody leaves" already scores 87%. |
| Why boosted trees? | Better than logistic on 3 periods, well calibrated. Cost: less explainable. |
| Why depth 3 / 0.03 / 300 trees? | Best PR-AUC on validation among 18 settings; shallow trees limit memorising. |
| What is `value_eur_year`? | Fees + 0.3% interchange on card spend, last 90 days × 4. A lower bound. Not a model feature: used for the decision. |
| Why is the test gain 3× the validation gain? | More training data (fit + validation) and maybe an easier period. That is why we give a range. |
| Which column did you remove for leakage? | `plan` (export snapshot). `crypto`, `n_contacts`, opt-ins are snapshots too: kept but listed as a limit. |
| Why exclude new customers? | 0 in the test set, churn twice as much, not enough history. |
| What is your offer? | 5% cashback on card payments for 30 days, capped at €2. Same offer for everyone targeted; the model decides *who*, not *which offer*. |
| Why do opt-ins have both 0 and a flag? | Models need a value; the flag lets the model tell "unknown" from "refused". |
| What would you check first in month one? | The real success rate Z with a control group: everything depends on it. |
| What's the biggest weakness? | Sudden departures of active customers: invisible in transactions. |
| Explain this line: `d.n_tx_90d.clip(lower=1)` | Avoids dividing by zero for customers with no transaction in 90 days. |
| Explain `nlargest(BUDGET, 'p')` | Keeps the 500 rows with the highest probability. |
