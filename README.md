# Antarctic Ross Sea Ice Thickness and Southern Ocean Heat

This repository analyses how **Antarctic Ross Sea sea-ice thickness** varies on seasonal and interannual timescales, and how that variability relates to **sea-surface temperature (SST)** and **upper-ocean heat content**.

**Research question:** *How does Antarctic Ross Sea sea-ice thickness vary on seasonal and interannual timescales, and to what extent is this variability associated with changes in ocean temperature and heat content?*

Two complementary analyses are included:

- `Antarctica_notebook.ipynb` — ORAS5 reanalysis (January 2015–August 2026)
- `CMIP6_analysis.ipynb` — CMIP6 model sea-ice thickness predictions (2015–2029)

The region of interest is a Ross Sea box: **latitude −76° to −74°, longitude 174° to 176°**.

---

## Repository layout

| Path | Contents |
|------|----------|
| `data/` | Climate data (NetCDF) and the Copernicus download notebook. **ORAS5 files were downloaded manually from the product website** (they are not fetched by the notebook). |
| `data/data_dowload.ipynb` | Notebook to log in to [Copernicus Marine](https://data.marine.copernicus.eu/) and download Antarctic sea-ice thickness (`esa_obs-si_ant_phy-sit_nrt_l4-multi_P1D-m`). |
| `data/ORAS5/` | Monthly ORAS5 high-resolution 2D NetCDF files (local only; not stored on GitHub). |
| `data/CMIP6/` | CMIP6 sea-ice thickness NetCDF files from ESGF (local only; not stored on GitHub). |
| `data/sea_ice_thickness/` | Copernicus / CryoSat–ESA sea-ice thickness NetCDF files (local only). |
| `figures/` | Figures and summary documents (`Doc1_ORAS5.docx`, `Doc2_CMIP6.docx`) produced by the analyses. |
| `tables/` | Derived tables. Full-grid bronze CSVs are kept locally (too large for GitHub). |
| `Antarctica_notebook.ipynb` | ORAS5 EDA, climatology/variability, Isolation Forest outlier test, and discussion. |
| `CMIP6_analysis.ipynb` | CMIP6 SIT prediction analysis: EDA, climatology/variability, February/September statistics, and discussion. |
| `requirements.txt` | Python dependencies. |

Raw NetCDF and bronze CSV files are **gitignored** because they are tens of gigabytes (GitHub file limit is 100 MB). Clone this repo, then place the data under `data/` as described below.

---

## Data

### ORAS5 (used in `Antarctica_notebook.ipynb`)

**Source:** ECMWF Ocean Reanalysis System 5 (ORAS5), monthly high-resolution 2D operational files (`*_control_monthly_highres_2D_*_OPER_v0.1.nc`).

**How they were obtained:** files were **downloaded manually** from the Copernicus Marine / ORAS5 website (not via `data_dowload.ipynb`). Place them in `data/ORAS5/`.

**Period:** 2015 through August 2026.

**Variables (file prefixes):**

| Prefix | Physical quantity |
|--------|-------------------|
| `iicethic` | Sea-ice thickness (SIT) |
| `sosstsst` | Sea-surface temperature (SST) |
| `sohtc300` | Ocean heat content in the upper 300 m (OHC) |

Example filename: `iicethic_control_monthly_highres_2D_201501_OPER_v0.1.nc`.

### CMIP6 (used in `CMIP6_analysis.ipynb`)

**Source:** [ESGF](https://esgf.llnl.gov/) CMIP6 model output.

| Field | Value |
|-------|--------|
| Variable | Sea-ice thickness (`sithick`) |
| Model | HadGEM3-GC31-MM |
| Experiment | ssp585 (`r3i1p1f3`) |
| Period | 2015–2029 |
| Local path | `data/CMIP6/` |

Example filename: `sithick_SImon_HadGEM3-GC31-MM_ssp585_r3i1p1f3_gn_201501-202912.nc`.

### Copernicus Marine sea-ice thickness (download notebook)

`data/data_dowload.ipynb` uses the `copernicusmarine` Python package to:

1. Describe and subset the dataset `esa_obs-si_ant_phy-sit_nrt_l4-multi_P1D-m`.
2. Download `sea_ice_thickness` for longitudes −180°–180°, latitudes −80°–−50°, for 18 April 2025–14 August 2026.

A Copernicus Marine account is required (`copernicusmarine.login()`). Additional CryoSat–ESA southern-hemisphere SIT files (`SIT_sh_c2esaD1_*.nc`) may sit in `data/sea_ice_thickness/` if downloaded separately.

---

## Analysis (`Antarctica_notebook.ipynb`)

1. **EDA and preprocessing**  
   Concatenate monthly ORAS5 NetCDFs, subset the Ross Sea box, and inspect SIT, SST, and OHC.

2. **Sea-ice climatology and variability**  
   - Monthly climatology  
   - Annual variability  
   - Seasonal variability  
   - Anomalies (departures from the monthly climatology)  
   - Trend  

3. **Quick ML test vs. climatology anomalies**  
   An [Isolation Forest](https://scikit-learn.org/stable/modules/generated/sklearn.ensemble.IsolationForest.html) (`scikit-learn`) is fit on SIT, SST, and OHC anomalies to flag multivariate outliers and compare with the climatological anomaly definition.

4. **Discussion / key findings**  

- Antarctic Ross Sea sea-ice thickness exhibits a strong seasonal cycle, with minimum thickness during austral summer and maximum thickness during winter.
- Sea surface temperature shows an opposite seasonal behaviour, with warmer conditions occurring during periods of reduced sea-ice thickness.
- Interannual variability is substantial, but no clear monotonic trend in sea-ice thickness is observed over 2015–2026.
- Recent years show notable SST anomalies, particularly positive anomalies during austral summer 2025–2026. However, the short study period and incomplete 2026 data do not allow a robust long-term warming trend to be established.
- SIT anomalies also show substantial interannual variability, including periods of both positive and negative departures from the monthly climatology.
- SIT and SST anomalies exhibit a moderate negative correlation overall (r=−0.42), indicating that warmer-than-average surface waters tend to coincide with thinner-than-average sea ice.
- This relationship is strongly season-dependent, with stronger negative correlations in spring (r=−0.72) and autumn (r=−0.65), and a weaker relationship in winter (r=−0.28).
- In contrast, the relationship between ocean heat content and SIT is weaker and less consistent across seasons, with correlations ranging from approximately −0.33 to +0.20.
- These results suggest that SST is more strongly associated with SIT variability than OHC, although this does not imply a causal relationship.
- A preliminary Isolation Forest analysis identified March 2021 as a potential multivariate outlier, consistently characterized by below-average SIT (−0.15 m) combined with above-average SST (+0.18 °C) and OHC, supporting the anomaly-based analysis and highlighting this period as a candidate for further investigation.

---

## Analysis (`CMIP6_analysis.ipynb`)

Notebook to analyse CMIP6 sea-ice thickness (`sithick`) predictions for the Ross Sea and produce figures (`figures/Doc2_CMIP6.docx`).

1. **EDA and preprocessing**  
   Load the HadGEM3-GC31-MM NetCDF, subset the region of interest, and prepare a monthly SIT time series.

2. **Sea-ice climatology and variability**  
   - Monthly climatology  
   - Annual variability  

3. **Quick monthly statistics**  
   Distribution of SIT in **February** (end of austral summer) and **September** (end of austral winter).

4. **Discussion / key findings**  

- Strong seasonal variability: SIT varies substantially throughout the year, with a pronounced minimum around March and generally higher values later in the year.
- Large interannual variability: annual mean SIT ranges roughly from 1.04 to 1.20 m, with no clear monotonic trend over 2015–2026.
- Lowest annual SIT: around 2021 and 2026 (~1.04 m).
- Highest annual SIT: 2017 (~1.20 m), followed by 2019 (~1.18 m).
- Recent period: SIT increases from 2021 to 2025, then drops markedly in 2026.
- February: very large spread (~0.7–1.9 m), indicating strong interannual variability.
- September: much narrower distribution (~1.15–1.58 m), with a median around 1.4 m.

**Caveat:** the February–March behaviour looks unusual. It may reflect spatial averaging choices and/or known limitations of CMIP6 `sithick`. Further analysis of drivers is needed to interpret this pattern.

---

## CMIP6 vs. ORAS5

Both CMIP6 and ORAS5 reproduce a strong seasonal cycle in SIT, with substantial year-to-year variability. ORAS5 also suggests a physical link between warmer ocean conditions and thinner sea ice, especially in spring and autumn.

The CMIP6 simulation shows different SIT behaviour (unusual around February–March). This may come from model physics, spatial averaging, or how sea ice and ocean processes are represented. CMIP6 is also known to have issues with the sea ice state, so these results should be treated with caution.

**Next steps:**

- Compare CMIP6 and ORAS5 more consistently (metrics, and drivers such as SST, OHC, air temperature, winds, and ocean circulation).
- Identify biases in the CMIP6 SIT–ocean relationship relative to ORAS5.
- **Redo the same analysis with CMIP7 predictions**, once available..

---

## Setup

```bash
python -m venv .venv
# Windows:
.venv\Scripts\activate
pip install -r requirements.txt
```

Then open `Antarctica_notebook.ipynb` or `CMIP6_analysis.ipynb` in Jupyter or VS Code. Update file paths if your data are not under `data/ORAS5/` or `data/CMIP6/`.

---

## Figures

PNG maps and time-series plots live in `figures/` (monthly/seasonal/annual climatologies, heatmaps, Ross Sea snapshots, anomaly and variable-relationship plots). Summary documents:

- `figures/Doc1_ORAS5.docx` — ORAS5 analysis figures  
- `figures/Doc2_CMIP6.docx` — CMIP6 analysis figures  
