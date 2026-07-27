CRED — The UPI Growth Story

Capstone project — IIT Mandi AI-Powered Coding and Analytics Programme

The question

Is UPI's growth broadening across apps and banks, or concentrating into a duopoly? NPCI publishes the monthly ecosystem numbers every month — this project assembles them into a single dataset and answers the question with evidence, from the perspective of a challenger app like CRED trying to understand the market it's competing in.

Answer, in one line: the market is highly concentrated, not broadening. PhonePe and Google Pay together account for the overwhelming majority of transaction volume; every other app, including CRED, is competing for a thin remainder.

Repo structure
├── data/
│   ├── raw/monthly/          # NPCI's raw monthly Excel files (Ecosystem-Statistics-UPI-...)
│   └── processed/
│       ├── data.csv          # cleaned, combined monthly app-wise data
│       └── cred_upi.db       # SQLite database, table: app_monthly
├── dashboard/
│   └── total_volume_trend.png
├── notebooks/
│   └── project.ipynb         # full pipeline: assembly → cleaning → SQL → EDA → model
├── memo.pdf                  # one-page decision memo
└── README.md
Data
Source: NPCI monthly UPI Ecosystem Statistics (app-wise), publicly published.
Coverage: 24 months, January 2024 – December 2025.
Fields used: app name, total transaction volume (Mn), total transaction value (Cr), month.
Scale: ~90 distinct apps per month after cleaning, ~580+ app-month rows in the raw pull, aggregated to monthly totals and per-app trends for analysis.
Setup & how to run
bash
# 1. Clone the repo
git clone <your-repo-url>
cd cred-upi-growth-story

# 2. Install dependencies
pip install pandas matplotlib seaborn scikit-learn openpyxl

# 3. Place NPCI's monthly Excel files in data/raw/monthly/
#    (filenames must contain the pattern YYYY-Mon, e.g. 2025-Jan)

# 4. Run the notebook top to bottom
jupyter notebook notebooks/project.ipynb

Running the notebook regenerates data/processed/data.csv, loads it into cred_upi.db, reproduces the SQL queries, the trend chart, and the growth model.

Methodology

1. Assembly. Each monthly NPCI file arrives with a two-row multi-level header and inconsistent app-name spellings. The pipeline parses the month from the filename, flattens the header, extracts app name / total volume / total value, and concatenates all months into one table.

2. Cleaning. App names needed real reconciliation before any analysis was trustworthy:

A manual name map merged obvious spelling/formatting variants (e.g. "Phone Pe" → "PhonePe", "Paytm (OCL)" → "Paytm", "Fampay"/"Fam App by Trio"/"FAM" → one canonical brand).
A case-normalization pass caught duplicates that differed only in capitalization, mapping each to its most common casing.
Volume and value fields were coerced to numeric after stripping thousands-separator commas, with a check confirming zero rows failed conversion.
This took 91 raw app-name variants down to 90 clean, canonical apps.

3. SQL. Data was loaded into SQLite (cred_upi.db, table app_monthly). Key queries:

A window function (SUM(volume_mn) OVER (PARTITION BY month)) computing each app's market share within its month, used to track CRED, PhonePe, Google Pay, Paytm, Navi, and super.money over time.
A GROUP BY month aggregation producing total ecosystem-wide volume and value per month, the base series for the trend and the model.

4. Exploration. The monthly total-volume series (plotted in dashboard/total_volume_trend.png) shows steady, roughly linear growth from ~12,300 Mn transactions in Jan 2024 to ~21,100 Mn in Dec 2025, with no sharp discontinuities — a real, sustained expansion of UPI usage. Ranking apps by cumulative volume makes the concentration visible immediately: the top 2 apps dwarf everyone else, including CRED.

5. Model. A linear regression on a monthly time index predicts total ecosystem volume, split 80/20 by time (19 months train, 5 months test — never shuffled, since this is a trend, not i.i.d. data).

Metric	Value
Monthly growth	~368.6 million transactions/month
Test MAE	~346.3 million transactions
Test R²	0.656

The model captures the overall growth trend reasonably well but leaves real error — roughly one month's worth of average growth — because it only sees a time index and nothing about seasonality, pricing changes, or entrant/exit events.

Findings
Concentration, not broadening. PhonePe (~58,900 Mn) and Google Pay (~45,300 Mn) cumulative volume together vastly exceed the rest of the ecosystem combined. Paytm is a distant third (~8,600 Mn).
CRED's position. CRED's own UPI volume is small in absolute terms (~968 Mn cumulative) and its monthly market share hovered close to ~1% through 2024, drifting slightly downward rather than growing share.
Ecosystem growth is real but doesn't change the shape. Total volume grew steadily across the 24 months, but that growth has not translated into a more even spread across apps — the leaders are growing with the market, not losing ground to challengers.
Dashboard

[Link to published Tableau dashboard — add here]

Decision memo

See memo.pdf — the one-page recommendation for a challenger app's strategy team, given a concentrating rather than broadening market.

Limitations
24 months only. The window doesn't include earlier UPI history, so long-run concentration trends before 2024 aren't captured.
Trend-only model. The linear regression uses time as its only feature — it can't explain why volume moves, and it can't anticipate one-off shocks: pricing changes, new entrants, or regulatory action (e.g. NPCI's market-share caps) could break the projection without warning.
App-name reconciliation was manual. The name map was built by inspection; a handful of smaller or newly-rebranded apps could still be undercounted or double-counted.
"Others" bucket. Some very small apps are grouped and not individually tracked, which slightly understates the long tail's true diversity.
AI Workflow Appendix

[Fill in: the prompts that helped you, what the AI contributed, one moment it was confidently wrong and how you caught it, and where you trusted it vs. didn't.]

Author

[Your name] — IIT Mandi AI-Powered Coding and Analytics Programme
