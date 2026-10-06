# Road Safety Investment & Risk Analysis — Italian Municipalities (2018–2024)

Capstone project for the **Master in Data Analytics by Boolean**.

The purpose of the project is to analyze road accident data for every Italian municipality between 2018 and 2024, on behalf of a fictional road traffic management company that needs to decide where to invest in road safety, and why.

It covers the full analytics workflow: automated data fetching, data cleaning in Python, exploratory and statistical analysis, k-means clustering, a linear regression model, and an interactive Power BI dashboard.

---

## Business question

> Which Italian municipalities have the highest road accident risk, and what kind of risk is it?

Absolute accident counts mostly reflect city size: the largest cities always rank first. To find where risk is really concentrated, the analysis shifts the focus from absolute numbers to **relative risk**, measured per resident and per square kilometre.

---

## Data sources

The analysis combines three official ISTAT sources:

- **ISTAT road accident data** (dataset code `41_983`): number of road accidents, injured and killed people per municipality per year, from 2001 to 2024. The data was fetched automatically through the official ISTAT API (SDMX standard).
- **ISTAT SITUAS**: resident population and surface area (km²) of each municipality, from 2018 to 2024. The data was downloaded as yearly CSV files.
- **ISTAT municipality list**: province codes and full province names. This Excel file was used to build a lookup table that adds the province name to each municipality.

The meaning of the `RESULT` column was decoded using the official ISTAT codelist [CL_ESITO](https://esploradati.istat.it/SDMXWS/rest/codelist/IT1/CL_ESITO).

---

## Repository structure

**Notebooks and deliverables**
- `fetching_cleaning_data.ipynb`: data fetching, cleaning, joining and creation of the derived metrics.
- `eda_and_modeling.ipynb.ipynb`: exploratory analysis, outlier handling, clustering and regression.
- `report.pbix`: Power BI dashboard.
- `Road_Safety_Investment_Dashboard_Presentation.pptx`: final presentation.

**Raw data**
- `istat_dataset.csv`: raw ISTAT data, saved locally after the first API call.
- `situas_2018.csv` to `situas_2024.csv`: raw SITUAS yearly files.
- `situas_dataset.csv`: SITUAS yearly files combined into a single dataset.
- `municipality_list.xlsx`: ISTAT list of municipalities and provinces.
- `istat_province_lookup.csv`: lookup table linking each province code to its full name.

**Output datasets**
- `final_dataset.csv`: cleaned and joined dataset, produced by the first notebook.
- `final_dataset_with_analysis.csv`: dataset enriched with the analysis results and clusters, produced by the second notebook and used in Power BI.

---

## Workflow

### 1. Data fetching and cleaning — `fetching_cleaning_data.ipynb`

- **API fetching**: ISTAT data is downloaded through the API with retry logic and saved locally, so the API is only called once.
- **Combining sources**: the SITUAS yearly files are merged and converted from the Italian number format. Municipality codes are kept as strings to preserve leading zeros.
- **Decoding the `RESULT` column**: `F` and `M` were initially read as female and male, but the distribution did not make sense. The ISTAT codelist confirmed that `F` = injured, `M` = killed and `9` = total.
- **Reshaping and joining**: one row per municipality per year, joined with population and surface data for 2018–2024.
- **Values misread as missing**: `NA` (Napoli's province code) and `None` (a municipality in Piedmont) were read as missing values by pandas and then restored.
- **Derived metrics**: accidents per 10,000 residents (`ACCIDENTS_PER_CAPITA`) and per km² (`ACCIDENTS_PER_KMQ`).

**Result**: a clean dataset of 53,817 rows with no missing values.

### 2. Exploratory analysis and modeling — `eda_and_modeling.ipynb.ipynb`

- **Outliers** capped with the IQR method, keeping the original values.
- **Small municipalities** (under 1,000 residents) flagged, since their per-capita rates are unstable.
- **Geographic patterns** compared across provinces.
- **K-means clustering** (k = 3, elbow method) on the two normalized risk metrics.
- **OLS linear regression** of total accidents on population, evaluated with MAE on a train/test split.

### 3. Power BI dashboard — `report.pbix`

The **Road Safety Investment Dashboard** allows users to explore accidents, injuries and fatalities over time and across provinces, and to compare municipalities according to their risk profile. The dashboard is designed for a non-technical audience: it focuses on clear indicators and leaves out the statistical model details.

---

## Key findings

**Three risk profiles** emerged from the clustering:

- **Low Risk (52.4%)**: municipalities with low values on both metrics, mostly in the South and Islands (e.g. Enna, Agrigento, Benevento).
- **High Per-Capita Risk (28.0%)**: many accidents per resident, spread over a wide area. This profile is typical of Center-North provinces (e.g. Ferrara, Reggio nell'Emilia, Firenze), with Brindisi as an exception.
- **High Density Risk (19.6%)**: risk concentrated per km², typical of major cities and metropolitan areas (e.g. Monza e della Brianza, Milano, Napoli).

**Other insights**

- The top 5 municipalities by total accidents (Roma, Milano, Genova, Torino, Napoli) mostly reflect city size, not relative risk.
- **Napoli stands out**: fewer accidents than Roma or Milano, but the highest fatality rate among the major cities (about 1.3 deaths per 100 accidents).
- **Regression**: population alone explains about **80% of the variability** in total accidents (R² = 0.805). The model predicts about 1.8 additional accidents per year for every 1,000 additional residents. Test MAE (2.57) is close to training MAE (2.58) and much lower than the naive baseline (7.22), so the model generalizes well. This result quantifies an expected relationship rather than revealing a surprising one.

---

## Tools

- **Python**: pandas, NumPy, requests, scikit-learn, statsmodels, seaborn, matplotlib
- **Power BI**: Power Query, DAX
- **Git / GitHub**

---

## Author

**Greta Barrale** — Junior Data Analyst