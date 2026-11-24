# 🎓 World University Rankings 2018–19 — Exploratory Data Analysis

An end-to-end EDA of a 1,000-university ranking dataset (the layout and indicators match the *Center for World University Rankings*, CWUR, 2018–19 edition). The focus is on **data quality**: hidden missing values, outliers, suspicious records, and a documented, defensible cleaning strategy — followed by 15 visualisations, including a world map.

![World map](images/world_map.png)

## What's in this project

| Step | What is done |
|------|--------------|
| 1. Loading | Robust loading (the file is **not** UTF-8), column-name repair (non-breaking spaces) |
| 2. Column review | A markdown description + profile for **every** column |
| 3. Non-numeric characters | Value-, token- and character-level proof of which characters live in the "numeric" columns |
| 4. Missing values | Detection of *hidden* missing values, a statistical test of their meaning, flags + imputation |
| 5. Outliers | IQR and z-score detection, a decision per column, winsorising / log versions of `Score` |
| 6. Suspicious values | 13 rule-based consistency checks, tied-rank plateaus, Isolation-Forest anomaly detection |
| 7. Visual exploration | 15 plots (distributions, correlation, pair plot, radar, heat-maps, **world map**…) each with an explanation |
| 8. Summary & export | Key findings, decisions log, cleaned CSV |

## Key findings

* **`isna()` reports zero missing values — but 7 of 10 numeric columns are stored as text.** Only two placeholder tokens are responsible: `-` (*not ranked*) and `> 1000` (*beyond the list*). No other stray characters exist.
* **The two tokens mean different things.** `-` is *informative* missingness (universities without it score significantly lower; Mann-Whitney p ≈ 0) and affects 60 % / 73 % of `Quality of Education` / `Quality of Faculty`. `> 1000` is right-censoring (5–51 % of five other columns).
* **Outliers are real.** 33 universities lie above the IQR fence of `Score` (≈ 86); they are the world's elite institutions and are kept. The seven rank columns show no IQR outliers.
* **Small defects found and repaired:** non-breaking spaces in two column names and 8 institution names where accented letters were irreversibly replaced by `?`.
* **`Citations` is low-resolution:** a single rank value (898) is shared by 129 universities (tied ranks).
* **The research block drives the ranking:** `Quality Publications` has |Spearman ρ| ≈ 0.96 with `Score`; `Alumni Employment` carries the most independent information.
* **Geography:** the USA holds 213 of the 1,000 slots and 51 of the top 100; North America supplies 55 % of the top 100 while Asia supplies 40 % of the 501–1000 tier.

<p align="center">
  <img src="images/missing_map.png" width="48%" alt="Hidden missing values map">
  <img src="images/correlation_heatmap.png" width="48%" alt="Spearman correlation heat-map">
</p>

## Cleaning decisions

| Topic | Decision |
|-------|----------|
| Encoding | Read as `cp1252` |
| Column names | Non-breaking space → normal space |
| `-` | Flag `<col>_unranked`; fill = (largest observed rank of that column) + 1 |
| `> 1000` | Flag `<col>_gt1000`; fill = 1001 |
| Outliers | Kept; added `Score_winsor` and `Score_log` |
| Corrupted names | Manual repair; flag `name_repaired` |
| Multivariate anomalies | Kept; flags `anomaly_flag`, `anomaly_score` |
| Derived features | `Region`, `Rank band` |

Why not the mean, median or nearest-neighbour fill? Because `-` means *"behind every ranked university"*. Any statistic computed from ranked universities places the unranked ones **ahead** of many genuinely ranked ones.

### Imputation experiment (`scripts/imputation_experiment.py`)

Following a discussion with classmates, four fill methods were compared:

![Imputation experiment](images/imputation_experiment.png)

* If values were missing **at random**, neighbour methods recover them best (masked test, right).
* But `-` is **not** random: only `max+1` respects its meaning — the others let an unranked university "beat" 35–42 % of the ranked ones (logic check, left).
* Using `World Rank` to fill anything is **target leakage** if the goal is later to predict it.
* Group-wise or KNN imputation is better done **after a train/test split** in a modelling stage; flags are kept so any method can be swapped in.

## Repository structure

```
.
├── README.md
├── requirements.txt
├── LICENSE
├── data/
│   └── README.md          # where to put the dataset (the CSV is not committed)
├── notebooks/
│   └── university_rankings_eda.ipynb   # the full analysis, outputs included
├── scripts/
│   └── imputation_experiment.py
└── images/                # figures used in this README
```

## How to run

```bash
git clone <your-repo-url>
cd <your-repo>
python -m venv .venv && source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# put eighteen_nineteen_university_datasets.csv into data/  (see data/README.md)
jupyter notebook notebooks/university_rankings_eda.ipynb
```

Running the notebook writes `data/university_rankings_clean.csv`. The world map downloads country outlines from [Natural Earth](https://www.naturalearthdata.com/) the first time (internet needed once; if it fails only that section is skipped). Afterwards you can run `python scripts/imputation_experiment.py`.

Tested with Python 3.12, pandas 3.0, numpy 2.5, matplotlib 3.11, seaborn 0.13, SciPy 1.18, scikit-learn 1.9.

## Data

`eighteen_nineteen_university_datasets.csv` — 1,000 rows × 12 columns: `World Rank`, `Institution`, `Location`, `National Rank`, seven indicator ranks (`Quality of Education`, `Alumni Employment`, `Quality of Faculty`, `Research Output`, `Quality Publications`, `Influence`, `Citations`) and `Score`. Column descriptions follow the public CWUR methodology; see `data/README.md` for details and the licence note.

## Acknowledgements

Course project completed with peer discussion — classmates' questions about imputation (neighbour-based fills, group-wise fills, data leakage) directly shaped the decisions and the experiment above. 

## License

Code and notebook: MIT (see `LICENSE`). The dataset has its own terms — see `data/README.md`.
