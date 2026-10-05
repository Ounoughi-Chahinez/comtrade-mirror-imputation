# comtrade-mirror-imputation

Mirror-statistics + Random Forest imputation of missing UN Comtrade bilateral trade
records (exports and imports), 2017-2024, with a provisional 2025 extension built
from GoodTrade-sourced partner declarations.

Generated: 2026-10-05

**This repository holds code, documentation, and all small result files.**
The trained models (~4.1GB) and full imputed datasets (~390MB+ including
per-year files) are hosted separately on Zenodo:

> **Data + models DOI: https://doi.org/10.5281/zenodo.23154375**

## Method summary

Missing country-year-commodity trade records are imputed using a Random Forest
regression model trained on partner-reported ("mirror") trade data, after excluding
non-country/aggregate reporter and partner codes (EU aggregate, World, UN Comtrade
"nes" regional buckets, Bunkers, Free Zones, Special Categories) from training --
see `data/qc/non_country_codes.csv` for the excluded code list and
`data/qc/qc_summary_{exports,imports}.json` for the quantified impact of that
exclusion on each pipeline run.

Evaluation uses partner-based 5-fold cross-validation (folds split by partner
country, not by row) over test years 2022-2024, reported by fold and HS level
(HS2/HS4/HS6) in `data/{exports,imports}/cv_folds_*.csv` and `cv_hs_*.csv`.

### Exports

- Rows removed by non-country/aggregate-code QC filter: 17.07%
- Value removed by the same filter: 59.79%
- Best cross-validation fold: Fold 3 (R² = 0.875, OOB = 0.635)
- Missing-country counts by year: 2017: 64, 2018: 44, 2019: 37, 2020: 33, 2021: 28, 2022: 28, 2023: 31, 2024: 52
### Imports

- Rows removed by non-country/aggregate-code QC filter: 17.07%
- Value removed by the same filter: 60.52%
- Best cross-validation fold: Fold 5 (R² = 0.901, OOB = 0.889)
- Missing-country counts by year: 2017: 64, 2018: 44, 2019: 37, 2020: 33, 2021: 28, 2022: 28, 2023: 31, 2024: 52

## Repository structure

```
code/          the pipeline notebooks (QC, exports, imports, this build script)
data/
  qc/          non-country/aggregate code list + QC impact summaries
  exports/     exports-side results: missing countries, CV, coverage, Russia
               case study, 2025 validation (small files only -- see Zenodo for
               the full imputed dataset)
  imports/     same, for imports
docs/          data dictionary, license
```

## Reproducing the full dataset / models

Either download them from the Zenodo DOI above, or regenerate them by running
`code/Pipeline_Exports_Full.ipynb` and `code/Pipeline_Imports_Full.ipynb`
end to end against the raw UN Comtrade source data.

## Data dictionary

See `docs/DATA_DICTIONARY.csv` for a column-by-column description of the imputed
dataset files. `isReported=False` marks rows produced by this pipeline rather
than originally declared.

## Known limitations

- `isAggregate` in the raw UN Comtrade data is a commodity-classification
  (HS-hierarchy) flag, not a country-aggregation flag -- it is not usable for
  detecting non-country reporters/partners (confirmed empirically; see QC notebook).
- 2025 figures are a provisional extension based on partner-reported GoodTrade
  declarations, not yet validated against a finalized external benchmark.
- Reported external benchmarks (BACI, IMF, World Bank) differ in scope
  (goods-only vs. goods-and-services) and valuation convention (FOB vs. CIF) from
  this HS-coded, goods-only reconstruction -- see the accompanying paper for a
  full discussion of comparability.

## Citation / license

Released under CC BY 4.0 -- see `docs/LICENSE.txt`.
