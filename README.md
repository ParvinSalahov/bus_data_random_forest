# Baku Bus Ridership Forecasting

A beginner-friendly machine-learning project that explores daily passenger counts by Baku bus route and compares a Random Forest regressor with simple historical baselines.

> **Current status:** This is an exploratory prototype, not a production forecasting service. In the included data, the Random Forest did not beat the 7-day baseline. The CSV ends on **2026-08-31**, so the notebook's next-day forecast is for **2026-09-01** and is not a current forecast.

## Project contents

| File | Description |
| --- | --- |
| `bakubus_passenger-2026.csv` | Daily passenger counts by date and route |
| `bus.ipynb` | Data checks, feature creation, model evaluation, and next-day forecasting |
| `requirements.txt` | Python packages needed to run the notebook |
| `.gitignore` | Excludes local environments and generated model/output files |

## Dataset

The CSV has 13,801 records and these columns:

| Original column | Meaning |
| --- | --- |
| `Tarix (il-ay-gün)` | Date |
| `Marşrutun adı` | Bus route identifier |
| `Gündəlik gediş sayı` | Daily passenger count |

The observed period is **2026-01-01 through 2026-08-31**, with 65 distinct routes. The notebook renames these columns to `date`, `route`, and `passengers`.

The current checks find no empty cells and no duplicate date/route records. The notebook also reports date/route combinations without a row. This check compares every route seen anywhere in the file with every date in the file; a missing combination is **not necessarily bad data**. A route may not have been operating yet or may not have been scheduled that day. Confirm the data provider's meaning before filling any gaps with zero.

Only publish the CSV to a public GitHub repository if you have permission to redistribute it. The dataset's source and license are not specified in this project.

## Set up on Windows

Open PowerShell in the project folder and run:

```powershell
py -3.11 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

Open `bus.ipynb` in JupyterLab. Choose **Kernel > Restart Kernel and Run All Cells** to reproduce the analysis from a clean state. Keep the CSV in the same folder as the notebook.

If PowerShell blocks virtual-environment activation, use the Python executable directly instead:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\jupyter-lab.exe
```

## What the notebook does

1. Loads the CSV, renames columns, parses dates and passenger counts, and checks nulls and duplicate date/route keys.
2. Checks for route/date combinations with no recorded row. These are flagged for investigation and are not automatically imputed.
3. Creates `lag_7` and `lag_14` features from the same route's count exactly 7 and 14 calendar days earlier, plus weekday and weekend features.
4. Holds out the last 28 days (**2026-08-04 through 2026-08-31**) for a chronological test. This reflects a rolling one-day-ahead setting: earlier actual counts are available as the test period progresses.
5. Trains a scikit-learn pipeline with one-hot route encoding and a Random Forest regressor.
6. Compares the model's mean absolute error (MAE) to a 7-day lag baseline and the average of the 7-day and 14-day lags, then reports per-route errors.
7. Builds a next-day forecast for routes with both lag values available.

## Evaluation on the included data

Re-running the notebook produced these test-set MAE values:

| Method | MAE (passengers) |
| --- | ---: |
| Random Forest | 766.5 |
| 7-day lag baseline | 674.9 |
| Average of 7-day and 14-day lags | 714.2 |

Lower MAE is better. On this holdout, the simple 7-day baseline performed best, so the Random Forest should **not** be presented as an improvement. Results may change if the dataset is updated or the evaluation period changes.

## Important limitations and next improvements

- The dataset covers only eight months and ends on 2026-08-31. Update it before using this as a current forecasting tool.
- The missing route/date check assumes the full set of observed routes might appear on every observed date. Route start dates and operating schedules are not included, so some flagged combinations may be expected.
- The test evaluates rolling daily predictions with observed values becoming available over time. A multi-day forecast made without new observations needs a different evaluation and recursive forecasting setup.
- The model currently uses route, weekday/weekend, and two lag values only. Holidays, service frequency, weather, special events, and route-operation calendars could add useful information if reliable data is available.
- Compare candidate models over more than one chronological holdout period before choosing a model for operational use.
- No model artifact, application, deployment, or automated data update is included.

## Publishing to GitHub

Before publishing, review the CSV's redistribution terms and remove any data you are not allowed to share. This folder is not currently initialized as a Git repository.

1. Create an empty repository on GitHub. Do not initialize it with a README, since this folder already has one.
2. In PowerShell, from this project folder, initialize Git and commit the project files:

   ```powershell
   git init
   git add README.md requirements.txt .gitignore bus.ipynb
   git add bakubus_passenger-2026.csv
   git commit -m "Add Baku bus ridership forecasting project"
   git branch -M main
   ```

   Only add the CSV if you are allowed to redistribute it.
3. Connect the local repository to the GitHub repository and push:

   ```powershell
   git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPOSITORY.git
   git push -u origin main
   ```

Replace the URL with the repository URL shown by GitHub. No license file is included because the project owner has not selected a license.
