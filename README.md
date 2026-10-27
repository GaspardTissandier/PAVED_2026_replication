# PAVED with good intentions — replication package

Replication code and data for **"PAVED with good intentions"** by Lecorps and Tissandier.

All results in the paper (tables and figures) are produced by a single R Markdown file:
[`script/PAVED_Lecorps_Tissandier_finale_2026.Rmd`](script/PAVED_Lecorps_Tissandier_finale_2026.Rmd).

The paper estimates the effect of PAVED, rolled out in the gendarmerie areas of 11 French departments in October 2017, on recorded burglaries and vehicle thefts. It uses monthly administrative crime data from French police and gendarmerie services.

---

## Contents

```
.
├── 20261026_PAVED_IRLE.Rproj   RStudio project (sets the working directory)
├── README.md
├── renv.lock                   Exact versions of all R packages (restored with renv)
├── .Rprofile, renv/            renv bootstrap files (activate renv when the project opens)
├── data/                       Raw input data (see "Data sources")
├── script/
│   └── PAVED_Lecorps_Tissandier_finale_2026.Rmd   Full analysis
└── output/                     All figures (.pdf) produced by the Rmd
    └── tables/                 All regression tables (.tex) produced by the Rmd
```

---

## How to replicate

1. Install R 4.5.x and the GLPK system library (see "Requirements").
2. Open `20261026_PAVED_IRLE.Rproj` in RStudio, or start R in the repository root. renv bootstraps itself automatically. Then install the exact package versions used for the paper:
   ```r
   renv::restore()
   ```
3. Open `script/PAVED_Lecorps_Tissandier_finale_2026.Rmd` and **Knit**, or from the `script/` folder run:
   ```r
   rmarkdown::render("PAVED_Lecorps_Tissandier_finale_2026.Rmd")
   ```
4. All figures are written to `output/` and all tables to `output/tables/`. The knitted HTML report is written next to the Rmd in `script/`.

All paths in the Rmd are relative to `script/` (`../data/`, `../output/`), so it must be knitted from that folder, which is the default when you click **Knit**.

**Run from a clean session.** Restart R and clear the environment before knitting. In RStudio, set *Tools → Project Options → General → Restore .RData into workspace at startup* to **No**, so that no objects from an earlier session are used.

### Runtime

Most of the runtime is in the HonestDiD section. The HonestDiD sensitivity analysis (section *Honest DiD approach*) is much slower and can take up to 20mn. It computes relative-magnitudes confidence sets (`createSensitivityResults_relativeMagnitudes`) for 6 event-study models, each with 32 pre-periods, 15 post-periods and 4 values of M̄. Setting `parallel = TRUE` in these calls, after registering cores with `doParallel::registerDoParallel()`, makes it substantially faster and gives the same results.

---

## Requirements

### R and packages

The results were produced with **R 4.5.2** on Ubuntu Linux. The project uses [renv](https://rstudio.github.io/renv/): `renv.lock` records the exact version of all 144 packages, including dependencies, and `renv::restore()` installs them into a project-specific library without affecting your other R projects. The main packages are:

| Package | Version | Used for |
|---|---|---|
| `fixest` | 0.14.2 | All regressions (`feols`, `fepois`, `etable`) |
| `ggfixest` | 0.4.0 | Event-study plots (`ggiplot`) |
| `HonestDiD` | 0.2.8 | Sensitivity analysis (Rambachan & Roth) |
| `Rglpk` | 0.6-5.1 | Linear programs inside `HonestDiD` |
| `tidyverse` | 2.0.0 | Data manipulation and plots (`dplyr` 1.2.1, `tidyr` 1.3.2, `purrr` 1.2.2, `ggplot2` 4.0.3) |
| `readxl` | 1.5.0.1 | Reading the Excel crime data |
| `lubridate` | 1.9.5 | Dates |
| `broom` | 1.0.13 | Extracting model coefficients |
| `xtable` | 1.8.8 | LaTeX tables |
| `doParallel` | 1.0.17 | Optional parallel backend for `HonestDiD` |
| `rmarkdown` / `knitr` | 2.32 / 1.52 | Knitting the report |

**Recommended:** run `renv::restore()` from the project root (see "How to replicate"). renv does not install R itself. With an R version other than 4.5.x, some packages may need to be compiled from source.

**Without renv:** install the packages manually. Results may then differ slightly, particularly with other versions of `fixest` and `HonestDiD`.

```r
install.packages(c("fixest", "ggfixest", "HonestDiD", "tidyverse", "readxl",
                   "lubridate", "broom", "xtable", "doParallel", "rmarkdown", "knitr"))
```

If you start R in the project without wanting renv, run `renv::deactivate()` first.

### System dependency: GLPK

`HonestDiD` depends on `Rglpk`, which needs the GLPK library installed on the system:

- **Debian / Ubuntu:** `sudo apt install libglpk-dev`
- **Fedora:** `sudo dnf install glpk-devel`
- **macOS (Homebrew):** `brew install glpk`
- **Windows:** the CRAN binary of `Rglpk` includes GLPK, so nothing extra is needed.

Without it, the HonestDiD section fails with `unable to load shared object ... Rglpk.so: libglpk.so.40: cannot open shared object file`.

---

## Data sources

All data are public French administrative data released under the [Licence Ouverte / Open Licence](https://www.etalab.gouv.fr/licence-ouverte-open-licence/), which allows redistribution with attribution. The Rmd reads four files from `data/`:

### 1. Monthly recorded crimes: `tableaux-4001-gn.xlsx`, `tableaux-4001-pn.xlsx`

- **Dataset:** *Chiffres départementaux mensuels relatifs aux crimes et délits enregistrés par les services de police et de gendarmerie depuis janvier 1996* ("état 4001").
- **Publisher:** Ministère de l'Intérieur.
- **URL:** https://www.data.gouv.fr/fr/datasets/chiffres-departementaux-mensuels-relatifs-aux-crimes-et-delits-enregistres-par-les-services-de-police-et-de-gendarmerie-depuis-janvier-1996/
- **Content:** one file for the gendarmerie (`gn`) and one for the police (`pn`). Each has one sheet per department, with monthly counts for each of the 107 offence indices of état 4001. The dataset is no longer updated; its last data are from August 2022.
- **Use:** the source of all crime outcomes (see "Variables").

### 2. Police / gendarmerie jurisdiction of each municipality: `export-competence-territoriale-pn-gn.csv`

- **Dataset:** *Compétence territoriale gendarmerie et police nationales*.
- **Publisher:** Ministère de l'Intérieur.
- **URL:** https://www.data.gouv.fr/fr/datasets/competence-territoriale-gendarmerie-et-police-nationales/
- **Content:** for each municipality (`code_commune`), the force responsible (`institution` = `GN` or `PN`) and the specific brigade or police station.
- **Use:** assigns each municipality's population to the gendarmerie or the police area of its department.

### 3. Municipal population: `population_2006_2021.csv`

- **Dataset:** *Historique des populations communales — Recensements de la population 1876-2021*.
- **Publisher:** INSEE.
- **URL:** https://www.insee.fr/fr/statistiques/3698339
- **Content:** municipal population (`PMUN<year>`) for each year from 2006 to 2021, by municipality (`CODGEO`), with region (`REG`), department (`DEP`) and name (`LIBGEO`).
- **Construction:** this CSV is an extract of INSEE's `base-pop-historiques-1876-2021.xlsx` (sheet `pop_1876_2021`, first 5 rows are a header block), keeping the columns `CODGEO, REG, DEP, LIBGEO, PMUN2021, …, PMUN2006`. The original workbook is included in `data/` for reference. To rebuild the CSV:
  ```r
  library(readxl)
  pop <- read_excel("data/base-pop-historiques-1876-2021.xlsx",
                    sheet = "pop_1876_2021", skip = 5)
  pop <- pop[, c("CODGEO", "REG", "DEP", "LIBGEO", paste0("PMUN", 2021:2006))]
  write.csv(pop, "data/population_2006_2021.csv", row.names = FALSE)
  ```
- **Use:** denominators for the crime rates. Municipal populations are summed by department × jurisdiction × year.

---

## Sample and variables

### Sample

- **Units:** jurisdiction (gendarmerie or police) × department, observed monthly.
- **Period:** the data start in 2006, when the population series starts. Regressions use **January 2015 – December 2018**. The descriptive long-run plots use 2010–2018.
- **Exclusions:** overseas departments (971–974), and Paris and the inner suburbs (75, 92, 93, 94), which have no gendarmerie area.
- **Treatment:** PAVED starts in **October 2017** (`post = 1` from 2017-10-01) in the gendarmerie areas of 11 departments: 04, 31, 34, 38, 44, 59, 60, 63, 67, 77, 83.
- **Neighbouring departments** (`dep_n`): departments that border treated departments. They are controlled for to allow for spillovers into catchment areas.

### Outcomes

Rates are per 10,000 inhabitants of the jurisdiction × department cell. The numbers are état 4001 offence indices.

| Variable | Definition |
|---|---|
| `burglary_r` | Residential burglary: indices 27 + 28 |
| `vehicle_theft_r` | Vehicle theft: indices 34 + 35 + 36 |
| `theft_r` | Theft (placebo outcome): indices 32 + 33 |
| `tfa_r` | Theft from vehicles (placebo outcome): indices 37 + 38 |
| `dealing_r` | Drug dealing (placebo outcome): index 55 |

### Identification strategies

The analysis uses three comparisons:

1. **Gendarmerie vs. gendarmerie** (prefix `cg_`): gendarmerie areas of treated departments vs. gendarmerie areas of untreated departments.
2. **Gendarmerie vs. police within treated departments** (prefix `gp_`, files `*_police_*`): within each treated department, gendarmerie areas (treated) vs. police areas (not treated).
3. **Triple difference** (prefix `cd3_` / `d3_`, files `*_didid_*` / `*_3d_*`): the gendarmerie–police gap before and after October 2017, in treated vs. untreated departments.

Each comparison is estimated with four sets of fixed effects:

1. none (canonical DiD)
2. jurisdiction × department + date (TWFE)
3. department × calendar month (seasonality)
4. jurisdiction × department + date + department × calendar month ("augmented TWFE")

Standard errors are clustered at the level of the intervention, jurisdiction × department.

---

## Outputs

Each output file is produced by one section of the Rmd. Section names below are the headings in the Rmd.

### Tables (`output/tables/`)

| File | Rmd section | Content |
|---|---|---|
| `conterfact_gendarmerie_catchment_burglary.tex` | Comparing gendarmerie areas → Burglary | DiD, 4 FE specifications |
| `conterfact_gendarmerie_catchment_vehicle.tex` | Comparing gendarmerie areas → Vehicle theft | DiD, 4 FE specifications |
| `conterfact_police_no_catchment_burglary.tex` | Gendarmerie vs. police areas → Burglary | DiD, 4 FE specifications |
| `conterfact_police_no_catchment_vehicle.tex` | Gendarmerie vs. police areas → Vehicle theft | DiD, 4 FE specifications |
| `conterfact_d3_catchment_burglary.tex` | Triple diff-in-diff → Burglary | DDD, 4 FE specifications |
| `conterfact_d3_catchment_vehicle.tex` | Triple diff-in-diff → Vehicle theft | DDD, 4 FE specifications |
| `poisson_reg_model4.tex` | Robustness check: count regression | Poisson models, specification 4 |
| `simple_theft_reg.tex` | Effect on other street crime | Theft |
| `theft_from_auto_reg.tex` | Effect on other street crime | Theft from vehicles |
| `drug_dealing.tex` | Effect on other street crime | Drug dealing |
| `alternative_crime_model_2_4.tex` | Effect on other street crime | Theft and drug dealing, specifications 2 and 4 |
| `short_long_term_effects.tex` | Short and long term effect | Effects by sub-period after treatment |

### Figures (`output/`)

| File pattern | Rmd section | Content |
|---|---|---|
| `counterfactual_{gendarmerie,police,didid}_{burglary,vehicle_theft}_r.pdf` | Plot (each strategy) | Raw trends of treated vs. control, 2015–2018 |
| `event_study_conterfact_{gendarmerie,police,3d}_{burglary,vehicle}_r.pdf` | Regression (each strategy) | Event-study coefficients (TWFE), reference month 2017-09 |
| `recap_model4.pdf` | Plot of all final regressions | Main estimates with 90% and 95% CIs |
| `placebo_counterfactual_{gendarmerie,police,3d}_{burglary,vehicle_theft}_r.pdf` | Placebo and randomization | Distribution of placebo estimates vs. true estimate |
| `long_term_counterfactual_{gendarmerie,police,didid}_{burglary,vehicle_theft}_r.pdf` | Long term plots | Raw trends, 2010–2018 |
| `honest_did_{cg,cp,3d}_{burglary,vehicle_theft}.pdf` | Honest DiD approach | Rambachan & Roth relative-magnitudes sensitivity |

---

## Citation

If you use this code or data, please cite the paper:

> Lecorps, & Tissandier, G. *PAVED with good intentions*. Working paper.

Please also cite the data producers (Ministère de l'Intérieur; INSEE) and the methods packages, in particular:

- Bergé, L. (2018). *Efficient estimation of maximum likelihood models with multiple fixed-effects: the R package FENmlm.* CREA Discussion Papers. (`fixest`)
- Rambachan, A., & Roth, J. (2023). *A More Credible Approach to Parallel Trends.* Review of Economic Studies, 90(5), 2555–2591. (`HonestDiD`)

## Contact

For questions about the code or data, please open an issue on this repository.
