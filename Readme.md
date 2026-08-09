<div align="center">

# ✈️ Dynamic Flight Operations Scoring System
### A data-driven Flight Difficulty Score for Chicago O'Hare (ORD)

**United Airlines SkyHack 3.0 Case Study**

![Python](https://img.shields.io/badge/Python-3.10-3776AB?logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Wrangling-150458?logo=pandas&logoColor=white)
![Scikit--learn](https://img.shields.io/badge/Scikit--learn-Modeling-F7931E?logo=scikitlearn&logoColor=white)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)
![Cells](https://img.shields.io/badge/Notebook-159%20cells-blue)

*Replacing gut-feel flight prioritization with a transparent, daily, rankable difficulty score.*

</div>

---

## 🎯 The Problem

> *"Frontline teams at United Airlines are responsible for ensuring every flight departs on
> time and is operationally ready. However, not all flights are equally easy to manage...
> Currently, identifying these high-difficulty flights relies heavily on personal experience
> and local team knowledge."*

Some flights are simply harder to turn around than others — a tight connection window, a
bag-heavy load, a cabin full of wheelchair requests. Ops teams currently spot these by
instinct. That doesn't scale, and it isn't consistent from shift to shift.

**This project builds a Flight Difficulty Score** that quantifies exactly how hard each
flight is to manage — using only information known *before* departure — so operations teams
can staff proactively instead of reacting after things go wrong.

---

## 🗂️ The Data

Five raw extracts, 15 days of ORD departures (Aug 1–15, 2025), ~8,000 flights:

| File | Grain | What it adds |
|---|---|---|
| `Flight_Level_Data.csv` | 1 row / flight | Schedule, turnaround times, delay outcome |
| `Bag_Level_Data.csv` | 1 row / bag | Checked, transfer & hot-transfer bag volume |
| `PNR_Flight_Level_Data.csv` | 1 row / booking | Passenger load, family travel, fare class |
| `PNR_Remark_Level_Data.csv` | 1 row / service request | Wheelchair, unaccompanied minor, etc. |
| `Airports_Data.csv` | 1 row / airport | Domestic vs. international lookup |

---

## 🧭 How the Notebook Is Organized

The project deliberately moves slowly and shows its work — no jumping straight to a model.

```
Explore each table on its own
        ↓
Merge tables (one at a time, verified)
        ↓
Post-merge EDA
        ↓
Feature engineering (one feature at a time, validated)
        ↓
🎯 Flight Difficulty Score  ← the core deliverable
        ↓
Machine learning (secondary check only)
        ↓
Business insights + final export
```

| # | Section | What happens |
|---|---|---|
| 1–2 | **Understand the data** | Each of the 5 tables explored individually — shape, types, nulls, duplicates, sample rows, and what it means operationally |
| 3 | **Merge** | Tables joined one at a time onto the flight table, each step verified (row counts, unmatched keys, sanity checks) |
| 4 | **Post-merge EDA** | Load factor, transfer bag ratio, SSR-vs-delay controlling for load, turnaround-vs-delay |
| 5 | **Feature engineering** | 4 features added one at a time — intuition → formula → validation |
| 6 | **🎯 Difficulty Score** | 6 weighted components combined into a 0–100 score, ranked daily, classified into Easy/Medium/Difficult |
| 7 | **ML (secondary check)** | 3 baseline models compared → Random Forest selected → feature importances cross-checked against the score's own weights |
| 8–9 | **Insights & export** | Operational recommendations + final CSV deliverable |

---

## 🧮 The Scoring Engine

The heart of the project. Every input is known **before** the flight departs — the score
never sees delay data, so it can be validated against real outcomes afterward as a genuine,
independent test.

$$\text{Difficulty Score} = 100 \times \sum_{i} w_i \cdot \text{risk}_i$$

| Component | Weight | Why |
|---|---|---|
| 🛬 Turnaround pressure | **0.35** | Strongest single relationship with delay in the EDA |
| 🧑‍🤝‍🧑 Passenger load factor | 0.15 | Fuller flights, more boarding friction |
| 🎒 Bag intensity (bags/pax) | 0.15 | More bags per passenger, slower loading |
| 🔄 Transfer bag ratio | 0.15 | Connecting bags add ramp coordination risk |
| ♿ Special service requests | 0.10 | Wheelchair/UM assistance needs extra gate time |
| ⏱️ Hot transfer bags | 0.10 | Sub-30-minute connections are the sharpest bag risk |

Each component is min-max scaled to [0, 1] before weighting, so no raw unit (minutes vs.
ratios vs. counts) dominates just because of its scale.

**Output, per flight:**
- `difficulty_score` — continuous 0–100
- `daily_rank` — rank within its own scheduled departure day (rank 1 = hardest that day)
- `difficulty_classification` — **Easy / Medium / Difficult**, from the score's own distribution

### ✅ Validation against real delay

| Tier | Delay rate (>15 min) | Avg. delay |
|---|---|---|
| Easy | 22.1% | 17.1 min |
| Medium | 24.8% | 16.8 min |
| **Difficult** | **48.9%** | **45.0 min** |

The Difficult tier delays more than **twice as often**, at nearly **triple** the average
delay of the Easy tier — using nothing but pre-departure signals.

---

## 🤖 Machine Learning — a Secondary Check, Not the Point

The score is the deliverable. Machine learning is used only to ask: *can the same signals be
learned, and does a model agree with what the score already assumes?*

Three baselines, compared honestly before picking one:

| Model | ROC-AUC | Accuracy | F1 |
|---|---|---|---|
| Logistic Regression | 0.687 | 0.715 | 0.466 |
| Decision Tree | 0.671 | 0.720 | 0.421 |
| **Random Forest** ✅ | **0.715** | **0.772** | **0.479** |

Random Forest wins without needing scaling — and with ~8,000 flights and 10 features, that's
already enough model capacity. No XGBoost, no deep learning, no forcing a fancier algorithm
than the data calls for.

**Feature importances confirm the score's own logic:** turnaround-related features dominate
the model's importances too — the hand-built score and the learned model agree on what
actually drives difficulty.

---

## 💡 Key Business Insights

- **Turnaround buffer is the #1 driver of difficulty** — tight-turn flights delay far more
  often than flights with a healthy buffer. The single most actionable lever available.
- **Difficulty compounds** — flights that are *both* tight-turn *and* high transfer-bag-ratio
  cluster at the top of the Difficult tier.
- **Load factor & SSR volume are real but secondary** — good for fine-tuning staffing, not
  strong delay predictors alone.
- **Specific destinations consistently score harder** — typically routes combining long
  scheduled turns with heavy transfer volume.
- **Recommendation:** feed `daily_rank` into the morning staffing huddle as a ready-made
  priority list; revisit score weights quarterly as schedules/fleet mix shift.

---

## 📁 Repo Contents

```
├── Flight_Difficulty_Score.ipynb                 # Main notebook (159 cells, fully executed)
├── Flight_Level_Data_with_difficulty_score.csv   # Final deliverable: flight data + score/rank/class
├── Flight_Level_Data.csv                         # Raw: flight schedule & actuals
├── Bag_Level_Data.csv                            # Raw: bag-level data
├── PNR_Flight_Level_Data.csv                     # Raw: passenger bookings
├── PNR_Remark_Level_Data.csv                     # Raw: special service requests
├── Airports_Data.csv                             # Raw: airport → country lookup
└── README.md
```

## ▶️ Running It

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook Flight_Difficulty_Score.ipynb
```

Run all cells top to bottom — each section depends on the `flights` DataFrame built up
progressively through the notebook.

---

## 🛠️ Tech Stack

`Python` · `Pandas` · `NumPy` · `Matplotlib` / `Seaborn` · `Scikit-learn`

<div align="center">

*Built as a data-driven alternative to spreadsheet-based, experience-only flight triage —
scoring engineering meets operations, one flight at a time.*

</div>
