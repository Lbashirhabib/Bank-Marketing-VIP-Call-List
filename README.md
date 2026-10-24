# Bank marketing response model: Week 2 variation (VIP list of 5%)

A machine-learning pipeline that ranks bank customers by how likely they are to subscribe to a term deposit, built on the UCI Bank Marketing data.

This is my Week 2 portfolio assignment for the Data Science Mentorship (Cohort 2, Module 1). It is adapted from Dr Okunola Orogun's notebook, and I changed one thing: the business assumption.

## The question
Which customers should the bank call when it can only contact a small **VIP list of 5%** of its customers?

## My revised business assumption
My Mentors version lets the bank contact 20% of customers. So i assumed the bank now wants only a **5% VIP list**, because each VIP call is handled by a senior relationship manager and is expensive. The model and the rest of the pipeline stay the same. What changes is how many customers are contacted, and therefore the threshold.

## Data
UCI Machine Learning Repository, Bank Marketing (`bank-additional-full.csv`): 41,188 contact records and 20 input columns. After removing 12 duplicate rows, 41,176 remain, and 11.27% of them subscribed.
Citation: UCI Bank Marketing dataset, https://doi.org/10.24432/C5K306 (CC BY 4.0).

## Setup
You need Python 3.10 or newer.

```bash
python -m venv .venv

# Windows PowerShell
.venv\Scripts\Activate.ps1
# Windows Command Prompt
.venv\Scripts\activate.bat
# macOS / Linux
source .venv/bin/activate

python -m pip install --upgrade pip
pip install -r requirements.txt
```

## How to run
1. Put `bank-additional-full.csv` in `data/uci-bank-marketing/` (or let the notebook download it).
2. Start Jupyter from this folder: `jupyter lab`
3. Open `notebook.ipynb`, then choose **Kernel > Restart and Run All**.
4. Results are saved in `portfolio_outputs/week02_variation_vip_5_percent/`.

## Project structure
```
notebook.ipynb          the full pipeline, with my comments in every cell
problem_brief.md        one-page problem brief
requirements.txt        libraries needed
data/                   the dataset goes here
portfolio_outputs/      This is created when you run the notebook
```

## Method in short
1. Frame the decision i.e the stakeholder, unit, horizon, target, action, constraint.
2. Load the data and check it against a schema.
3. Explore the data (imbalance, unknown values, response rate by group).
4. Leakage audit: remove `duration` it happens after the call.
5. Stratified split into 60% train, 20% validation and 20% test which is 24,705 / 8,235 / 8,236 rows.
6. Preprocessing inside a scikit-learn Pipeline.
7. Compare logistic regression and random forest on validation data, using average precision.
8. Retrain the winner on train + validation and test it once.
9. Set the threshold from the capacity: the score of the top 5% of customers.
10. Check the errors by job and the permutation importance of each column.

## Leakage decision
`duration` (i.e call length) is only known after the call ends, so using it to decide whom to call would be cheating. It makes the model look better than it could ever be in real use, so i removed it. All other columns are known before the call, so I kept them.

## Model and threshold choice
- **Model:** random forest. On the validation set it beat logistic regression on both ROC AUC (0.808 vs 0.802) and average precision (0.463 vs 0.447). I picked on average precision because only about 1 in 9 customers subscribes, and it describes the quality of the ranking without needing a threshold.
- **Threshold: 0.824.** It is not the default 0.5. It is the score of the customer at the 95th percentile of the test scores, so exactly the top 5% (about 412 of 8,236 customers) are contacted. I set the list size first (the capacity) and the threshold follows from it.

## Results (test set, 8,236 customers, 928 subscribers)

| Metric | Value |
|---|---|
| ROC AUC | 0.814 |
| Average precision | 0.490 |
| Threshold for the top 5% | 0.824 |
| Precision on the VIP list | 0.638 |
| Recall on the VIP list | 0.283 |
| Lift on the VIP list | 5.67 |

**Reading this:** about 64 in every 100 customers on the VIP list subscribed, against 11 in 100 across all customers. That is 5.7 times better than calling at random. The list reaches 28% of all subscribers. A list of about 412 people can never reach more than roughly 44% of the 928 subscribers (412 / 928), so low recall here comes from the small list size and not from a weak model.

For reference only, my mentors original 20% run gave a threshold of 0.429, precision 0.376, recall 0.667 and lift 3.33. These numbers come from his saved notebook output. I did not recompute them in my notebook.

## Three insights
1. **A small list is precise but reaches few subscribers.** At 5%, precision is 0.638 and lift is 5.67, which is much higher than the 20% list (precision 0.376, lift 3.33). But recall is only 0.283, so about 7 in 10 subscribers are not on the list. This fits a bank that wants a few high-quality calls and accepts missing the rest.
2. **Students, retired customers and customers with a successful earlier campaign are the strongest prospects.** In the data, 31.4% of students and 25.3% of retired customers subscribed, against 11.3% overall. Customers whose previous campaign was a success subscribed 65.1% of the time, against 8.8% for those never contacted before. The model reflects this: 20.7% of students and 16.1% of retired customers land on the VIP list, but only 1.3% of blue-collar customers. `poutcome` is also among the top five columns by permutation importance.
3. **The model leans heavily on economic conditions and timing.** The three most important columns are `nr.employed`, `euribor3m` and `emp.var.rate` (importance 0.052, 0.049 and 0.040), ahead of `contact` (0.031) and `poutcome` (0.019). In other words, who is likely to subscribe depends a lot on when the campaign runs, not only on who the customer is.

## Two limitations
1. **A 5% list is small and I tested it on only one split.** It holds about 412 customers, so a few lucky or unlucky calls can move precision, recall and lift noticeably. I did not repeat the experiment with other random splits, so I cannot say how much these numbers would change.
2. **The threshold and the split are simplifications.** The 5% capacity is an assumption. The threshold is taken from the test scores themselves, whereas a real deployment would fix it beforehand and apply it to new customers. The random split also ignores that the data is in time order, and because the model leans on economic columns (insight 3), it may not carry over to a different period. The data is also from one Portuguese bank.

## Responsible use
Scores are for prioritising a campaign only, and a person should review the ranked list before any contact. Real use would need a lawful basis, privacy review, monitoring for drift and checks of errors by group. For example, the errors table in the notebook shows that admin customers have the most missed subscribers (195), simply because they are the largest group. This project is for learning.

## Credits
Notebook structure adapted from Dr Okunola Orogun's Cohort 2, Module 1, Week 2 materials. Data from the UCI Machine Learning Repository.
