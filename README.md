
# Evaluating CAMELS-CL as a reference for bias-correcting ERA5-Land in the high Andes of Chile

*Capstone project for the YSE Environmental Data Science Certificate.*
*Author: Valentina Giovanetti · Last updated: November 2026*

## Project description
High-altitude salt flats in Chile are difficult to access and local meteorological stations are scarce. This project is a dataset-evaluation study: it assesses whether using CAMELS-CL as a reference for bias correcting ERA5-Land yields results comparable to using local station observations, with the aim of assessing where the two references produce congruent corrections and where they diverge most, and what this implies for unmonitored salt flats such as Maricunga.

*An earlier version of this project compared gridded datasets directly. Following feedback, the current design uses INIA station observations as an independent benchmark to evaluate both corrections.*

## Research question

**How does CAMELS-CL perform as a reference for QDM correction of ERA5-Land compared to meteorological stations?**

## Data

| File | Description | Source |
|------|-------------|--------|
| `raw_data/era5_daily_putre_caspana_basins.csv` | Daily basin-mean ERA5-Land time series for the Putre and Caspana basins. Contains ~50 variables, including total precipitation (m) and 2-m temperature (K). | Copernicus, ERA5-Land — downloaded [2026/11/15] via [Google Earth Engine](https://code.earthengine.google.com/7d67c1e504f3e2f7c31466281530c7fc) | 
| `raw_data/era5_daily_putre_caspana_stations.csv` | Daily ERA5-Land time series sampled at the pixel containing each station (Putre and Caspana). Contains ~50 variables, including total precipitation (m) and 2-m temperature (K). | Copernicus — downloaded [2026/11/15] via [Google Earth Engine](https://code.earthengine.google.com/004b13d4935d194e7a9ea730ba55f6af)  |
| `raw_data/inia_caspana_20261115.csv` | Raw daily meteorological observations (temperature and precipitation) from the INIA Agroclimatic Network station at Caspana station | INIA Agroclimatic Network - downloaded [2026/11/15] |
| `raw_data/inia_putre_20261115.csv` | Raw daily meteorological observations (temperature and precipitation) from the INIA Agroclimatic Network station at Putre station | INIA Agroclimatic Network - downloaded [2026/11/15] |
| `raw_data/tmean_cr2met_C_day.csv` | Daily mean 2-m temperature (°C) from CAMELS-CL for the basin containing each station. | CR2, CAMELS-CL202201 - downloaded [2026/11/15] |
| `raw_data/precip_cr2met_mm_day.csv` | Daily precipitation (mm/day) from the CAMELS-CL dataset for the basin containing each station. | CR2, CAMELS-CL202201 - downloaded [2026/11/15] |
| `clean_data/1_camels_daily_putre_caspana_basins_clean.csv` | Cleaned daily CAMELS-CL time series aggregated to the Putre and Caspana basins. Contains only precipitation (mm/day) and temperature (°C). One row per basin per day | Made by `scripts/1_data_clean.ipynb` |
| `clean_data/1_era5_daily_putre_caspana_basins_clean.csv` | Cleaned daily basin-mean ERA5-Land time series for the Putre and Caspana basins. Contains only precipitation (mm/day) and temperature (°C). One row per basin per day. | Made by `scripts/1_data_clean.ipynb` |
| `clean_data/1_era5_daily_putre_caspana_stations_clean.csv` | Cleaned daily ERA5-Land time series sampled at the pixel containing each station. Contains only precipitation (mm/day) and temperature (°C). One row per station per day | Made by `scripts/1_data_clean.ipynb` |
| `clean_data/1_inia_daily_putre_caspana_stations_clean.csv` | Cleaned daily INIA station observations for Putre and Caspana, combined into a single file. Contains only precipitation (mm/day) and temperature (°C). One row per station per day, with station metadata attached. | Made by `scripts/1_data_clean.ipynb` |

## Repository structure

```
era5_correction_high_andes/
├── README.md                    ← you are here
├── raw_data/                    ← original data, never edited
    ├── era5_daily_putre_caspana_basins.csv
    ├── era5_daily_putre_caspana_stations.csv
    ├── inia_caspana_20261115.csv
    ├── inia_putre_20261115.csv
    ├── tmean_cr2met_C_day.csv
│   └── precip_cr2met_mm_day.csv
├── clean_data/                  ← output of 1_data_clean.ipynb
    ├── 1_camels_daily_putre_caspana_basins_clean.csv
    ├── 1_era5_daily_putre_caspana_basins_clean.csv
    ├── 1_era5_daily_putre_caspana_stations_clean.csv
│   └── 1_inia_daily_putre_caspana_stations_clean.csv
├── figures/                     ← plots and tables (coming in later scripts)
└── scripts/                     ← notebooks, numbered in the order they run
    ├── 0_project_setup.ipynb
    └── 1_data_clean.ipynb
```

## How to run the code

All scripts are Google Colab notebooks written in R. Run them in number order:

1. `scripts/0_project_setup.ipynb` creates the project folder and subfolders in Google Drive.
2. `scripts/1_data_clean.ipynb` reads 6 raw files, reshapes and joins them, and writes
   -  `clean_data/1_camels_daily_putre_caspana_basins_clean.csv`
   -  `clean_data/1_era5_daily_putre_caspana_basins_clean.csv`
   -  `clean_data/1_era5_daily_putre_caspana_stations_clean.csv`
   -  `clean_data/1_inia_daily_putre_caspana_stations_clean.csv`
3. `scripts/2_analysis.ipynb` *(coming soon)* will correct ERA5-LAnd with the two references, compare the results and save figures to `figures/`.


## Data cleaning steps (`1_data_clean.ipynb`)

1. **Selected only the needed ERA5-Land variables**. From the ~150 columns in the raw ERA5-Land CSV, I kept date, site, temperature_2m, and total_precipitation_sum.
2. **Converted ERA5-Land units**. Temperature from K to °C (- 273.15), precipitation from m to mm (* 1000). Applied identically to both ERA5 extractions (basin-aggregated and station-pixel).
3. **Reshaped CAMELS-CL**. Renamed basin IDs: 1210001 to putre and 2105002 to caspana and transformed data from wide to long with pivot_longer(). Temperature and precipitation were reshaped separately, then merged by date and site.
4. **Standardised the INIA files**. Skipped the 5-line header (skip = 5), renamed the columns to date, temperature_c_inia, and precipitation_mm_inia, added a site column, combined both stations with rbind(), and parsed dates from dd-mm-YYYY to Date.
5. **Removed rows with missing values** in the INIA temperature and precipitation columns: 12 rows with NA were dropped.
6. **Created exploratory histograms**. Visually compare the distributions of temperature and precipitation before correction: INIA observations vs. ERA5-Land pixel values at each station, and CAMELS-CL vs. ERA5-Land basin averages for each basin.
   
**Known data issues:**
- Temporal coverage differs: ERA5-Land 1950–2026, CAMELS-CL 1979–2020, INIA 2016–2026. The common calibration window is limited to ~2016 to 2020. Additional older records are being explored to extend this period.
- ERA5-Land precipitation has small negative values, likely numerical artefacts.

## Status and next steps

- [x] Set up the project folder and GitHub repository
- [x] Clean and combine the raw data
- [x] Exploratory histograms: temperature and precipitation distributions per site (INIA vs. ERA5 pixel; CAMELS vs. ERA5 basin)
- [ ] Apply QDM: (1) ERA5 pixel vs. INIA, (2) ERA5 basin vs. CAMELS-CL
- [ ] Apply both transfer functions to basin-averaged ERA5 and compare
- [ ] Apply both transfer functions to pixel sampled ERA5 and compare

## AI Disclosure 
I used DeepSeek as a writing aid throughout this project. The AI helped me structure and translate ideas from Spanish, and summarize what the script does, to create this README. The research question, the selection of datasets and correction method, the decision to use Putre and Caspana as study basins, and all code in `1_data_clean.ipynb` are my own. The `0_project_setup` script is based on a template provided in the course.
