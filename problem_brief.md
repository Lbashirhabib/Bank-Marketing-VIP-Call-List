# Problem brief: Who belongs on the bank's VIP call list?

**Name:** Habib Bashir Lawal
**Course:** Data Science Mentorship, Cohort 2, Module 1, Week 2

## 1. The decision
Which customers should the bank phone first for a term-deposit campaign when only a small **VIP list** of 5% can be contacted?

## 2. The decision frame

| Box | My answer |
|---|---|
| **Stakeholder** | The campaign manager (and the senior relationship managers who make the VIP calls) |
| **Unit** (one row) | One customer contact record |
| **Horizon** | The outcome recorded for this campaign |
| **Target** | Subscribed to a term deposit (yes = 1, no = 0) |
| **Action** | Rank customers by score and give the call team the top of the list |
| **Constraint** | Contact no more than **5%** of customer |

## 3. My revised business assumption
The bank wants a small **VIP list of 5%** of customers instead of 20%. So each VIP call is handled by a senior relationship manager, so calls are expensive and the list must be as accurate as possible.

**What this changes:** only the cut-off is changed here, the model stays the same. The threshold becomes the score at the 95th percentile, so exactly the top 5% are contacted.

**What I expected before running the model:**
- I expect precision to be higher than at 20%, because only the most confident customers are picked.
- While recall must be lower, With a list this small even a perfect list could reach only a fraction of all subscribers, so low recall is a limit of the list size, not a failure of the model.
- Lift should be higher than at 20% as well.

**What I found (test set, 8,236 customers):** the threshold is 0.824 and the list holds about 412 customers. then Precision is 0.638, recall is 0.283 and lift is 5.67 This matches what I expected, precision and lift are higher than the 20% run which are 0.376 and 3.33, and recall is lower. The ceiling on recall for a 5% list is about 44% (412 of 928 subscribers).

## 4. How success is judged, deciding before looking at my results
- **Ranking quality:** ROC AUC and average precision on held-out data, these do not depend on list size.
- **Operating policy:** contact the top 5%, Reporting the threshold, precision, recall and lift. then i Judge the list mainly by precision and lift, because recall is capped by its small size.
- **Model choice:** i am to pick between logistic regression and random forest on validation data only, using average precision. and then touch the test set once at the end.

## 5. Data and the leakage decision
- Data: UCI Bank Marketing which contained 41,188 contact records, 41,176 after removing 12 duplicates, 11.27% subscribed.
- `duration` (i.e call length) is removed because it is only known **after** the call, so i cannot use it to decide whom to call.

## 6. What would make me stop using the model
- If the precision of the VIP list falls close to the overall subscription rate (the list stops adding value).
- If the data changes like new categories, or a different customer mix, and checks start failing.
- If the VIP list turns out to be made up almost entirely of one narrow group of customers. like i saw for now It already leans towards students and retired customers, so I would check this regularly.
- The economic conditions change a lot. The model relies heavily on `nr.employed`, `euribor3m` and `emp.var.rate`, so it should be re-tested for a new period before the bank relies on it.
