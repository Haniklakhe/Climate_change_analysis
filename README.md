# Climate Change Analysis: Karnali Basin

Station-wise bias correction of CMIP6 daily precipitation and temperature for the Karnali River basin, with a gridded ERA5-based workflow in progress.

> **Status: work in progress.** Precipitation bias correction has been run for one model (ACCESS-CM2, historical scenario only). Scenario outputs (SSP2-4.5, SSP5-8.5), temperature outputs and the gridded workflow are not yet complete. See [Known limitations](#known-limitations).

## Objective

Correct systematic biases in CMIP6 downscaled climate projections at the locations of observed stations in the Karnali basin, so that the corrected series can be used for climate-change impact analysis.

*Inferred from folder names, notebook headers and the data files. The project owner should confirm this statement.*

## Repository layout

```
Climate_change_analysis/
├── Basin_shp/                  Karnali basin boundary (shapefile, WGS84 used in the code)
├── Stationwise/
│   ├── Karnali_1986_2015.xlsx  Observed daily data and station metadata (sheets below)
│   ├── Precipitation/          Extraction and bias-correction notebooks + outputs
│   └── Temperature/            Extraction and bias-correction notebooks (Tmax; Tmin sheet present, not yet processed)
└── Gridded/
    └── Untitled0.ipynb         ERA5 gridded workflow (draft; incomplete)
```

### Workflow notebooks

| Step | Notebook | Purpose |
|---|---|---|
| 1 | `Stationwise/Precipitation/NC_to_Excel.ipynb` | Extract daily `pr` from one CMIP6 model at station locations (single model) |
| 1 | `Stationwise/Precipitation/NC_to_Excel_all_models_Rainfall.ipynb` | Same extraction for all models and scenarios |
| 1 | `Stationwise/Temperature/NC_to_Excel_all_models_Temperature/` | Same extraction for temperature (notebook saved without `.ipynb` extension) |
| 2a | `Stationwise/Precipitation/Quanitle_Mapping_Bias_Correction_Rainfall.ipynb` | Empirical quantile mapping (EQM), precipitation |
| 2b | `Stationwise/Precipitation/Linear_Scaling_Bias_Correction_Rainfall.ipynb` | Monthly linear scaling, precipitation |
| 2c | `Stationwise/Temperature/Quanitle_Mapping_Bias_Correction_Temperatuure/` | EQM, Tmax (notebook saved without extension) |
| 2d | `Stationwise/Temperature/Linear_Scaling_Bias_Correction_Temperature/` | Monthly linear scaling, temperature (notebook saved without extension) |
| – | `Gridded/Untitled0.ipynb` | ERA5 reference data and linear scaling on the grid (draft) |

Run the steps in order: 1 → 2. The Precipitation folder also contains `Copy of Copy of Quanitle_Mapping_Bias_Correction_Rainfall`, a duplicate of 2a without an extension. It is kept for now and should be removed after checking that it is identical to 2a.

## Data

| Dataset | Location | Description | Source |
|---|---|---|---|
| Observed station data | `Stationwise/Karnali_1986_2015.xlsx` | Sheets: `Precipitation`, `TMax`, `Tmin` (daily observations); `Sheet1` (station `Index No.`, `Lat`, `Lon`) | [Provenance to be added by the owner] |
| Basin boundary | `Basin_shp/Karnali_basin.shp` (+ `.dbf`, `.shx`, `.prj`) | Karnali basin polygon; reprojected to EPSG:4326 in the code | [Provenance to be added by the owner] |
| CMIP6 daily precipitation and temperature | Not in this repository | NASA NEX-GDDP-CMIP6, read from Google Drive at `/content/drive/MyDrive/NEX_GDDP_CMIP6/` | NEX-GDDP-CMIP6 (see the dataset's own documentation for licence and citation) |
| ERA5 reanalysis (hourly, `t2m`, `tp`) | Read remotely in `Gridded/Untitled0.ipynb` from a public Icechunk store (`earthmover-icechunk-era5`) | Reference for the gridded workflow | ERA5 (Copernicus Climate Change Service) |

CMIP6 variables: `pr` (precipitation, kg m⁻² s⁻¹; multiplied by 86400 to get mm day⁻¹) and `tasmax` (temperature).

## Method

**Bias correction.** Two methods are implemented:

- **Empirical quantile mapping (EQM)**, following the equations cited in the notebook headers (Themeßl et al., 2012; Gudmundsson, 2014). Monthly empirical CDFs are built with 99 quantile bins (`N_QUANTILES = 99`) for the observation and model series.
- **Monthly linear scaling (LS).** Precipitation uses multiplicative monthly factors (observed / simulated monthly mean, clipped to 0.1–10 in the gridded notebook). Temperature uses additive monthly differences.

**Calibration period.** The notebooks set `CAL_START = 1986-01-01` and `CAL_END = 2014-12-31`. The same correction factors or quantile tables are applied to the SSP scenarios.

**Spatial matching.** Each station is matched to the nearest CMIP6 grid cell.

## Outputs

Current outputs (committed, example run):

| File | Content |
|---|---|
| `Stationwise/Precipitation/Raw_GCM_Precipitation/ACCESS-CM2/historical/ACCESS-CM2_historical_karnali_pr.csv` | Raw ACCESS-CM2 daily precipitation at stations |
| `Stationwise/Precipitation/BiasCorrected_LS_Precipitation/ACCESS-CM2/historical/ACCESS-CM2_historical_bias_corrected.csv` | Linear-scaling corrected series |
| `Stationwise/Precipitation/BiasCorrected_QM_Precipitation/ACCESS-CM2/historical/ACCESS-CM2_historical_pr_qm_bias_corrected.csv` | EQM corrected series |
| `Stationwise/Precipitation/BiasCorrected_QM_Precipitation/qm_precipitation_summary.csv` | Run log: 1 model, 1 scenario, 10 stations, 10 957 days, status `Success` |

Column names in the station CSVs follow the pattern `<station index> (<lat>, <lon>)`.

## How to run

The notebooks were written for Google Colab with Google Drive mounted:

1. Upload the repository to Google Drive at `MyDrive/Hydrology_Applications/Climate_change_analysis/`.
2. Place the NEX-GDDP-CMIP6 files at `MyDrive/NEX_GDDP_CMIP6/<model>/<scenario>/<variable>/`.
3. Run the notebooks in the order in the table above.

Required packages: `xarray`, `rioxarray`, `geopandas`, `pandas`, `numpy`, `matplotlib`, `tqdm`, plus `icechunk` for the gridded notebook.

Local (non-Colab) runs are not yet supported. The paths are hard-coded (see [Known limitations](#known-limitations)).

## Known limitations

- **Hard-coded Colab paths.** Paths such as `/content/drive/MyDrive/...` are set in each notebook. The code cannot run on another machine without editing them.
- **Incomplete coverage.** Only ACCESS-CM2 historical precipitation has been corrected. The SSP2-4.5 and SSP5-8.5 scenarios, the second model (ACCESS-ESM1-5) and temperature are not yet run. `Stationwise/Temperature/Tasmax/BiasCorrected_QM_Tasmax/qm_tasmax_summary.csv` is empty.
- **Calibration period mismatch to check.** The observed file is named `1986_2015`, the notebooks calibrate on 1986–2014, but the corrected ACCESS-CM2 output starts on 1985-01-01 (10 957 days = 1985–2014). Confirm how the 1985 values were produced.
- **Gridded notebook is a draft.** `Gridded/Untitled0.ipynb` has an empty cell and text encoding errors (for example `â†’` in place of `→`). It is not used for the station results.
- **Notebooks saved without extensions.** Four notebooks in `Stationwise/Temperature/` and one in `Stationwise/Precipitation/` have no `.ipynb` extension. Rename them before opening them in some tools.
- **Duplicate notebook.** `Copy of Copy of Quanitle_Mapping_Bias_Correction_Rainfall` duplicates 2a.
- **Limited validation.** No bias-correction skill metrics (for example bias, RMSE, or quantile error on the held-out period) are saved in the repository. The corrected series have not yet been validated against observations.
- **Temperature and precipitation units.** Check the units of the observed `TMax` and `Tmin` sheets before comparing them to CMIP6 Kelvin values. This has not been verified.

## Citation and licence

No licence has been chosen yet. Until one is added, the code and data are all rights reserved.

Methods are attributed in the notebook headers to Themeßl et al. (2012) and Gudmundsson (2014). These full references have not been verified and must be checked before citing this work.

Dataset citations: NEX-GDDP-CMIP6 and ERA5 (see the dataset providers' documentation).

## Contact

Hanik Lakhe
