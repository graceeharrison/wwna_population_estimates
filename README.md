# WWNA Population Estimates

## Overview
This repository contains R code for joining population estimates from multiple data sources to a wastewater facilities list, and calculating total population estimates for facilities identified as inadequate.

## Input Files
| File | Description |
|------|-------------|
| `CIWQS_Assessment_Facilities_Matrix_20260312.xlsx` | Base facilities list (Assessment Matrix sheet) |
| `merged_population_data_py.csv` | Multi-source population data compiled from CWNS, SSO, and COVID-era records |
| `Population_Data_SSO_NPDES_20260305.xlsx` | Population data reported directly by Regional Water Boards, with separate sheets for SSO and NPDES facilities |
| `Inadequate_ID_all_0317_2026.xlsx` | List of facilities identified as inadequate, with separate sheets for NPDES, WDR, and SSSGO facility types |

## Population Data Sources
Four population sources are used:

| Column | Source |
|--------|--------|
| `pop_cwns` | Clean Watersheds Needs Survey (CWNS) |
| `pop_sso_csv` | SSO program records |
| `pop_covid` | COVID-era population reporting |
| `pop_served_via_rwb` | Values reported directly by Regional Water Boards (RWBs), drawn from the SSO sheet for SSO-category facilities and the NPDES sheet for NPDESWW-category facilities. WDR-category facilities will always have a blank value for this field. |

## Population Estimate Decision Logic
Two versions of the final population estimate are produced for each facility.

### Version 1: Original (`pop_final`)
1. If two or more sources report values within 10% of each other, the smallest of those agreeing values is used. The 10% threshold is calculated using the smaller of the two values as the denominator.
2. If no two sources agree within 10% (i.e., all sources are disparate), `pop_cwns` is used as the default.
3. If `pop_cwns` is not available and all sources are disparate, the smallest value across all available sources is used.
4. If only one source is available, that value is used directly.
5. If no source data is available, the estimate is left blank.

### Version 2: RWB-Preferred (`pop_final_rwb_preferred`)
Same logic as Version 1, except that if `pop_served_via_rwb` is available for a facility, it is always used as the final estimate regardless of what other sources report. The fallback logic (steps 1–5 above) only applies when RWB data is absent.

### Source Tracking
The columns `pop_final_source` and `pop_final_rwb_preferred_source` record which data source the final population estimate was drawn from for each facility.

## Output Files
All output files are saved to:
`C:/Users/grace/Box/Luskin Center for Innovation/All Files/Programs/Water/June2015-present/WWNA/Phase 2C/Inadequacy/Population Estimates`

| File | Description |
|------|-------------|
| `facilities_with_population.csv` | Full facilities list with all population columns and final estimates joined |
| `inadequate_npdes_with_population.csv` | Inadequate NPDES facilities with population estimates and totals |
| `inadequate_wdr_with_population.csv` | Inadequate WDR facilities with population estimates and totals |
| `inadequate_sssgo_with_population.csv` | Inadequate SSSGO facilities with population estimates and totals |

## Requirements
The following R packages are required:
```r
library(readxl)
library(dplyr)
library(readr)
library(stringr)
```

## Notes
- All joins between files are performed on `FACILITY ID` (numeric), which is consistent across the facilities list and all inadequate facility sheets.
- The join between the facilities list and the population CSV is performed on `FACILITY NAME` (normalized to uppercase, whitespace trimmed), as no shared numeric ID exists between those two files.
- Population totals for inadequate facilities are calculated separately for NPDES, WDR, and SSSGO program types using both the original and RWB-preferred methods to allow for comparison.
