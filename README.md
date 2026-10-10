# Attributable fractions of drug-label side effects from placebo-controlled trials

Analysis code for the manuscript *"Trial-validated hierarchical modelling shows most labelled side effects are largely not caused by the drug"* (submitted to *Nature Communications*).

The code estimates, for every drug–side-effect pair with placebo-controlled evidence on ClinicalTrials.gov, the
attributable fraction (the share of reports on the drug that would not have occurred on placebo) with a hierarchical
estimator, validates the estimator by predicting trials that started after it was fitted, and applies it to the side
effects listed on current US FDA labels.

## Contents

| Path | What it is |
|---|---|
| `afstd/` | Stand-alone reference implementation of the hierarchical estimator and the prediction benchmark, with tests and conformance data. Start here to inspect or reuse the method. |
| `src/oajoint/` | Shared modules: `af3.py` (hierarchical estimator), `bench.py` (prediction benchmark and scoring), `af2.py` (empirical-Bayes comparator), `af_common.py`, `routes.py`, `provenance.py` (the only write path into the study database), `manifest.py` (audit), `paper.py`, `figures.py`, `schematics.py` (figures). |
| `scripts/` | The analysis in run order (file-name numbers). |
| `db/schema.sql` | Schema of the study database (DuckDB). |

### Pipeline

| Scripts | Step |
|---|---|
| 32–34, 37, 64, 91 | Download ClinicalTrials.gov results (API v2), map drugs to ChEMBL, parse arms, adverse-event groups, baseline and flow modules, build placebo-controlled contrasts |
| 41–68, 73–75 | Conventional (Mantel–Haenszel) estimates, linkage to FDA (openFDA) and 2016 (SIDER) labels, and the analyses of Supplementary Fig. 6 (reproducibility, route, indication, published meta-analyses, dose) |
| 80–83 | Empirical-Bayes shrinkage (a comparator) |
| 92–102, 106 | Hierarchical estimator, known-truth simulations, prediction benchmark (development windows and confirmation set), held-out drugs and serious events |
| 108–110c | Label analyses of Figs. 3–5 (expected shares, organ systems, drug classes, absolute risks, listing rules, sensitivity analyses) |
| 111–112 | Main and supplementary figures and Source Data |

Scripts whose names end in `prereg` record analysis rules (with thresholds and pass criteria) in the decision table
`prov_decision` before the script that evaluates them is run; `python -m oajoint.manifest --verify` checks that every
evaluated rule was registered first. Internal identifiers in the code map to the paper as follows: `AF 3.3` is the
hierarchical estimator reported in the paper, `AF 3.2` the same without its no-effect component, `AF 1.0` the
conventional estimator, `B0`–`B7` the comparators of Fig. 2b, and `bench22_confirm33` the confirmation-set evaluation.

## Requirements

Python 3.14 with the packages in `requirements.txt` (tested with the pinned versions). The analysis runs on a laptop
(11 cores, 18 GB RAM); the hierarchical fits take minutes to hours.

## Reproducing the analysis

```bash
pip install -r requirements.txt
export PYTHONPATH=src
python -m oajoint.provenance --init          # creates db/oajoint.duckdb from db/schema.sql
python scripts/32_ctgov_download.py          # then the remaining scripts in numerical order
```

All inputs are public and downloaded by the scripts: ClinicalTrials.gov (API v2), the openFDA drug-label export,
SIDER 4.1 and ChEMBL. The paper used the downloads of 24 September 2026; later downloads will include newer trials and
labels. Intermediate files cached by scripts 98, 100 and 106 go to `$OAJOINT_SCRATCH` (default `/tmp`).

Figure scripts (`111`, `112`, `src/oajoint/figures.py`, `schematics.py`) use `figkit`, a private plotting helper that
is not part of this repository; all plotted values are provided in the Source Data of the paper.

### Reference implementation

```bash
pip install ./afstd
pytest afstd/tests
```

## Data and licences

Code: MIT licence (`LICENSE`). Input data remain under their providers' terms: ClinicalTrials.gov (terms of use of
the US National Library of Medicine), openFDA (CC0 1.0), SIDER 4.1 (CC BY-SA 4.0) and ChEMBL (CC BY-SA 3.0). Derived
estimates are provided with the paper as Supplementary Data and Source Data.

## Citation

If you use this code, please cite the paper (citation to be added on publication) and this repository
(`CITATION.cff`).

Correspondence: Xingrong Guo and Xiaohu Zhu, Taihe Hospital, Hubei University of Medicine, Shiyan, China.
