# 🗳️ Tamil Nadu 2026 Assembly Election — Data-Driven Prediction Pipeline

> A constituency-level seat projection model for the Tamil Nadu 2026 state elections, built on **real ECI 2021 data**, **2026 official elector rolls**, Logistic Regression, and a 10,000-run Monte Carlo simulation.

---

## 📌 Project Summary

This project answers one question with data:  
**Who is likely to win Tamil Nadu in 2026 — and by how much?**

Rather than relying on opinion or punditry, this pipeline uses:
- Real 2021 ECI election results (4,232 candidate rows across 234 constituencies)
- Official 2026 Tamil Nadu elector rolls (voter registration data)
- A trained Logistic Regression model (97.9% cross-validated accuracy)
- 10,000-run Monte Carlo simulation with per-AC uncertainty
- TVK (Tamilaga Vettri Kazhagam) vote-split modeling across 7 scenarios

---

## 🔑 Key Results (Base Scenario: TVK = 15%)

| Bloc | Mean Seats | 80% Confidence Interval | Majority Prob |
|---|---|---|---|
| **DMK Alliance** | **174** | 155 – 192 | ~99% |
| AIADMK Alliance | 58 | 42 – 74 | <1% |
| TVK | ~0–15 | varies | <1% |
| Others | ~2 | — | — |

> **Hung Assembly probability: <2%**  
> Majority threshold: 117 seats

---

## 🧠 Methodology

### 1. Data Ingestion & Party Mapping
- Loaded 2021 ECI CSV (4,232 rows) and 2026 elector rolls (234 ACs)
- Mapped full party names → short codes (DMK, AIADMK, INC, NTK, PMK, BJP, VCK, etc.)
- Assigned each party to alliances: **DMK Alliance** vs **AIADMK Alliance** vs Others
- Built constituency-level pivot table of votes per party

### 2. 2026 Elector Roll Integration
- Merged 2026 registration data (Male / Female / Total electors per AC)
- Estimated 2021 elector base using statewide average turnout (72.8%)
- Computed elector growth rate and new voter count per constituency

### 3. TVK Split Modelling
- TVK (Vijay's party) is a new 2026 entrant with no historical data
- Modelled TVK vote share as drawn **50% from AIADMK, 25% from DMK, 25% from Others**
- Urban ACs (larger electorates) receive proportionally higher TVK share
- Added per-AC random variance (σ = 4%) for realism

### 4. New Voter Scaling
- New voters routed to parties using survey-derived preference priors:
  - DMK Alliance 40% | TVK 25% | AIADMK Alliance 20% | Others 15%
- Final adjusted vote shares normalised to sum to 100% per AC

### 5. Logistic Regression (Trained on 2021 Data)
- Features: DMK Alliance share, AIADMK Alliance share, NTK share, Others share, Elector count
- Target: Did DMK Alliance win this AC? (binary)
- 5-fold stratified cross-validation
- **Accuracy: 97.9% ± 2.3% | F1: 98.5% ± 1.7%**
- Used to generate per-AC DMK win probability for 2026

### 6. Monte Carlo Simulation (n = 10,000)
- Added Normal(0, 7%) noise per AC per simulation
- Applied FPTP (First Past the Post) rule to determine winner per AC
- Aggregated seat counts across 234 ACs for each simulation
- Outputs: mean seats, 80% CI, majority probability, hung assembly probability

### 7. TVK Sensitivity Sweep
- Tested TVK vote share scenarios: 8%, 10%, 12%, 15%, 18%, 20%, 22%
- Finding: Even at 22% TVK share, DMK Alliance remains dominant (163 seats)
- AIADMK is the primary loser as TVK share grows

### 8. Backtest (2016 → 2021)
- Validated the model by simulating 2021 from 2016 baseline + anti-incumbency swing
- Predicted DMK Alliance wins fell within the 80% CI of the actual 2021 result (159 seats)

### 9. Swing Seat Identification
- Classified each AC: DMK Safe | AIADMK Safe | Marginal (40%–60% win probability)
- Identified top 20 most competitive constituencies

---

## 📁 Repository Structure

```
TN_2026_Fixed.ipynb          ← Main notebook (run cells top to bottom)
Tamil_Nadu_State_Elections_2021_Details.csv   ← 2021 ECI data (4,232 rows)
2026_VOTE_UPDATES.csv        ← 2026 elector rolls (234 ACs)
TN_2026_AC_Projections.csv   ← Output: full 234-AC projection table

outputs/
  ├── elector_growth.png       ← Elector growth histogram + top 10 ACs
  ├── vote_to_seat_2021.png    ← 2021 alliance scatter + NTK spoiler chart
  ├── confusion_matrix.png     ← LR model confusion matrix
  ├── lr_coefficients.png      ← Feature coefficients chart
  ├── monte_carlo_2026.png     ← Seat distribution histograms (4 blocs)
  ├── tvk_sensitivity.png      ← TVK sweep: seats + probability lines
  ├── marginal_seats.png       ← Top 20 swing seats
  ├── final_projection_2026.png← Pie + bar chart final summary
  └── poll_tracker.png         ← Live poll tracker (updatable)
```

---

## ⚙️ Setup & Usage

### Prerequisites
```bash
pip install pandas numpy scikit-learn matplotlib seaborn scipy
```

### How to Run
1. Clone this repo
2. Update `FILE_2021` and `FILE_2026` paths in **Cell 1** to point to your local CSV files
3. Run cells **top to bottom in order** (do not skip any cell)
4. All charts save automatically to your working directory

### Cell Run Order

| Cell | What it does |
|------|-------------|
| 1 | Imports + file paths |
| 2 | Load & verify CSVs |
| 3 | Party mapping + AC-level pivot |
| 4 | Merge 2026 elector rolls |
| 5 | Elector growth visualisation |
| 6 | 2021 baseline scatter chart |
| 7 | Define helper functions (TVK split, MC, voter scaling) |
| 8 | Build 2026 base scenario (TVK = 15%) |
| 9 | Train Logistic Regression |
| 10 | Run Monte Carlo (10,000 simulations) |
| 11 | MC distribution plots |
| 12 | Backtest 2016 → 2021 |
| 13 | TVK sensitivity sweep (8%–22%) |
| 14 | Marginal / swing seat analysis |
| 15 | Final projection chart |
| 16 | Export 234-AC CSV |
| 17 | Poll tracker chart |

---

## 📊 Output: 234-AC Projection Table

The exported `TN_2026_AC_Projections.csv` contains per-constituency:

| Column | Description |
|---|---|
| `Constituency` | AC name |
| `Winner_Party` | 2021 actual winner party |
| `Electors_2026` | Registered voters in 2026 |
| `New_Voters` | Additional voters since 2021 |
| `DMK_Alliance_Share` | 2021 actual vote share |
| `DMK_share_adj` | 2026 projected vote share |
| `Winner_adj` | Projected 2026 winner |
| `LR_DMK_WinProb` | Model win probability (0–1) |
| `Seat_Risk` | Safe / Marginal classification |

---

## ⚠️ Limitations & Assumptions

- **No actual 2026 polls used** — TVK priors are assumptions, not survey data. Update `apply_tvk_split()` parameters when real polls release.
- Backtest uses **simulated** 2016 shares (real 2016 AC-level data was not included in this pipeline).
- Elector growth calculation assumes a **72.8% turnout** in 2021 to back-estimate the 2021 elector base.
- Logistic Regression is trained and tested on the **same 2021 dataset** — the CV accuracy measures pattern detection, not true out-of-sample prediction.
- This is a **probabilistic model**, not a forecast. Results change with real poll data inputs.

---

## 🔄 How to Update with Real Poll Data

When real survey data releases, update `poll_tracker` in **Cell 17** and update the new voter preference priors in `scale_for_new_voters()` in **Cell 7**:

```python
new_voter_prefs = {
    'DMK_Alliance':    0.40,   # ← update from real surveys
    'AIADMK_Alliance': 0.20,
    'TVK':             0.25,
    'Others':          0.15,
}
```

---

## 🛠️ Tech Stack

- **Python 3.14**
- **pandas / numpy** — data wrangling
- **scikit-learn** — Logistic Regression, cross-validation, pipeline
- **matplotlib / seaborn** — all visualisations
- **scipy** — statistical utilities

---

## 📂 Data Sources

- 2021 Tamil Nadu Assembly Election results — **Election Commission of India (ECI)**
- 2026 Tamil Nadu elector roll summary — **ECI / CEO Tamil Nadu**

---

## 👤 Author

**Prasanth Sundar**  
Data Analyst | CYGNUSA Technologies  
[LinkedIn](https://www.linkedin.com/in/prasanth-sundar-b65475359/) • [GitHub](https://github.com/Prashantsundar)

---

## 📄 License

This project is for educational and analytical purposes only. All electoral data is sourced from publicly available ECI records.

---

*Last updated: March 2026 — Model will be updated as real pre-poll surveys release closer to the 2026 election.*
