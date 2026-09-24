# Microcystin_Tox

This repository summarizes and analyzes aquatic toxicity data for microcystins to support
water quality criteria development for California freshwater species. Two analyses are
implemented following the US EPA (1985) *Guidelines for Deriving Numerical National
Water Quality Criteria for the Protection of Aquatic Organisms and Their Uses*
(Stephan et al. 1985).

## Repository Contents

| File / Folder | Description |
|---|---|
| `CCC.qmd` | Criterion Continuous Concentration (CCC) derivation — step-by-step EPA 1985 method in base R |
| `SSD.qmd` | Species Sensitivity Distribution (SSD) construction — log-normal fits across four species scopes using `ssdtools` |
| `Data/ToxSummary.csv` | Compiled microcystin toxicity dataset (see below) |

## Data: `ToxSummary.csv`

Each row is a single toxicity test result. Key columns:

| Column | Description |
|---|---|
| `source` | Source database or review (e.g., Mehinto) |
| `Species` | Common name with scientific binomial in parentheses |
| `reference` | Full literature citation |
| `life_stage` | Life stage tested (e.g., Embryonic, Juvenile, Adult) |
| `endpoint` | Biological endpoint (Growth, Development, Reproduction, Mortality, Growth + Development, Behavior, Histology, Subcellular) |
| `congener` | Microcystin congener tested (e.g., Microcystin-LR) |
| `exp_days` | Exposure duration in days |
| `chronic_acute` | Whether the exposure is chronic or acute |
| `noec_ug_l` | No Observed Effect Concentration (µg/L) |
| `loec_ug_l` | Lowest Observed Effect Concentration (µg/L) |
| `other_dose_ug_l` | Other reported dose values (µg/L) |
| `doses` | Full dose series tested |
| `group` | Broad taxonomic group (e.g., Chordata, Crustacea, Insecta) |
| `ca_resident` | `Yes` if the species has an established California population |
| `ca_native` | `Yes` if the species is native to California |
| `ca_native_genus` | `Yes` if the genus is native to California |
| `ca_relevant` | `Yes` if the species is considered relevant to California conditions (residents + widely used non-native test species) |
| `note` | Flags such as `Invasive` |

## Analyses

### CCC Derivation (`CCC.qmd`)

Implements the EPA FAV/FACR fallback method to produce a Criterion Continuous
Concentration:

1. Load data and parse genus from species names
2. Filter to apical endpoints (Growth, Development, Reproduction, Mortality, Growth + Development)
3. Apply `ca_relevant == "Yes"` scope filter
4. **Chronic side** — compute Chronic Values (√NOEC×LOEC), Species Mean Chronic Values (SMCV), and Genus Mean Chronic Values (GMCV)
5. **Acute side** — restricted further to Mortality endpoint; compute Acute Values, SMAV, GMAV
6. Fit EPA's log-triangular extrapolation to the four lowest GMAVs to produce the Final Acute Value (FAV)
7. Compute the Final Acute-Chronic Ratio (FACR)
8. CCC = FAV / FACR

> **Note:** The dataset does not meet EPA's 8-family minimum data requirement
> on either the chronic (2 genera / 2 categories) or acute (6 genera / 3 categories)
> side. The resulting CCC (~0.03 µg/L) is a worked demonstration of the
> calculation mechanics, not a defensible regulatory criterion.

### SSD Construction (`SSD.qmd`)

Fits log-normal SSDs across four species-scope variants using `ssdtools`:

| Variant | Filter | N species |
|---|---|---|
| All Species | none | maximum available |
| California Relevant | `ca_relevant == "Yes"` | subset comparable to CCC scope |
| California Residents | `ca_resident == "Yes"` | established CA populations only |
| California Native Genus | `ca_native_genus == "Yes"` | CA-native genera |

Data preparation steps applied before fitting: apical endpoint filter, LOEC/2
substitution for missing NOECs, acute values divided by 10 (fixed ACR), and
per-species median collapse.

A fifth variant (`ca_native == "Yes"`) is attempted but fails due to
insufficient species for reliable log-normal fitting.

## Reproducing the Analyses

The analyses are written as Quarto (`.qmd`) documents. To render them:

1. Install [R](https://cran.r-project.org/) and [Quarto](https://quarto.org/docs/get-started/).
2. Install required R packages:
   ```r
   install.packages(c("tidyverse", "ssdtools", "ggplot2"))
   ```
   `CCC.qmd` uses only base R — no additional packages needed.
3. Render from the terminal:
   ```bash
   quarto render CCC.qmd
   quarto render SSD.qmd
   ```
   Or open in RStudio and click **Render**.

Output is written as self-contained HTML and Word (`.docx`) files.

## Acknowledgments

Code and document text in this repository were developed with assistance from
Claude (Anthropic), an AI assistant. All code, analytical methods, and written
content were reviewed and verified for accuracy by Leah Thornton Hampton prior
to publication.

## Contact

**Leah Thornton Hampton**
Southern California Coastal Water Research Project (SCCWRP)
[leahth@sccwrp.org](mailto:leahth@sccwrp.org)

For questions about the dataset, analysis methods, or California-specific
scoping decisions, please reach out via email.
