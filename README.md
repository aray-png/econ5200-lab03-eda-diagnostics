# Pipeline Health Check — EDA, Corruption & Distribution Shift

**Abigail Ray || ECON 5200: Applied Data Analytics in Economics — Lab 3**

## Objective
This project demonstrates a diagnosis-first approach to data pipeline
monitoring, using exploratory data analysis alone to detect and correct
planted data corruption, and Population Stability Index (PSI) analysis
to quantify distribution drift between training and inference data.

## Methodology
- Diagnosed and corrected 5 planted data-quality issues in a merged
  country panel using EDA techniques alone (no automated profiling):
  negative GDP values, life expectancy stored in months rather than
  years, duplicate country-year records, inconsistent trade-percentage
  units, and a GDP unit mismatch caused by a faulty merge
- Identified each corruption through targeted diagnostics — `describe()`
  summary statistics, domain-range constraint checks, and duplicate-key
  validation — rather than reading generating code, mirroring how a
  real analyst would audit an unfamiliar production pipeline
- Quantified distribution shift between training and inference splits
  using the Population Stability Index (PSI), computing a GDP PSI of
  1.7600 (significant shift) for a deliberately introduced 1.3x scaling
  change, while also surfacing that untouched columns can still register
  PSI values in the "moderate shift" range purely from small-sample
  binning noise
- Compared manual EDA against automated profiling (ydata-profiling),
  identifying which corruptions were caught automatically (e.g. negative
  values, raw duplicate rows) versus which required domain knowledge no
  automated tool could supply (e.g. unit mismatches, duplicate keys with
  differing values, percentage-bound violations)
- Engineered `eda_utils.py`, a reusable, documented Python module exposing
  `check_impossible_values()`, `detect_distribution_shift()`, and
  `eda_summary()` for constraint validation, drift detection, and
  automated data profiling
- Built an interactive pipeline-health dashboard (ipywidgets + Plotly)
  enabling live column-level distribution comparison, adjustable PSI
  alert thresholds, editable data-quality constraints, and side-by-side
  before/after corruption views

## Key Findings
- Automated profiling reliably catches statistical anomalies (outliers,
  raw duplicates, negative values) but is fundamentally blind to
  corruptions requiring domain context — such as recognizing a bimodal
  GDP distribution as a units error rather than a natural feature of the
  data
- Standard PSI drift thresholds (0.10 / 0.25), borrowed from
  credit-scoring practice at sample sizes in the tens of thousands, do
  not transfer reliably to smaller production datasets, where binning
  noise alone can push untouched columns above conventional "shift"
  cutoffs
- A defensible drift-monitoring strategy requires establishing a null
  distribution — simulating PSI across random splits of undrifted data
  — rather than applying industry-standard thresholds by convention
- Duplicate-key corruption is invisible to PSI-based monitoring
  entirely, since duplicating existing rows changes row counts but not
  distributional shape, underscoring the need for multiple,
  complementary data-quality checks rather than any single metric

## Repository Contents
| File | Description |
|------|-------------|
| `lab-ch03-diagnostic.ipynb` | Full notebook: corruption diagnosis, PSI drift detection, manual vs. automated EDA comparison, and the interactive dashboard |
| `eda_utils.py` | Reusable module with `check_impossible_values()`, `detect_distribution_shift()`, and `eda_summary()` |
| `pipeline_health_dirty.csv` | Original corrupted dataset (260 rows, all five bugs intact) |
| `pipeline_health_clean.csv` | Cleaned dataset after fixes (230 rows) |
| `training.csv` / `inference.csv` | Reference and drifted splits used for PSI testing |

## Tools & Libraries
`pandas`, `numpy`, `matplotlib`, `ipywidgets`, `plotly`
