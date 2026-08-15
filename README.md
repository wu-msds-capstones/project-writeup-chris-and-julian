# Automation and Affordability in U.S. Counties

**[Read the study](https://wu-msds-capstones.github.io/project-writeup-chris-and-julian/)**

Chris Bell | Julian Pacheco
DATA 510 Data Science Capstone, Willamette University | Summer 2026

## Overview

We investigated the association between county task groups and resident’s purchasing power after controlling for local economic conditions. Our analysis spanned 15 years, from 2008 to 2023, excluding 2020. We combined data from various sources, including the Census Small Area Income and Poverty Estimates, American Community Survey, Bureau of Labor Statistics Local Area Unemployment Statistics, Bureau of Economic Analysis Regional Price Parities, and Census Core Based Statistical Area delineations, to create a county-year panel.

To understand local work, we classified employment into four task groups based on Autor, Levy, and Murnane (2003) and Autor and Dorn (2013): routine cognitive, routine manual, non-routine cognitive, and non-routine manual. These groups reflected the distinct types of tasks performed across counties.

We compared these task groups with purchasing power, which was measured as the county median household income divided by the local price level from the Bureau of Economic Analysis Regional Price Parities. This adjustment accounted for variations in the purchasing power of the same income level across different locations. The final analytical panel consisted of 11,983 county-year observations from 848 counties. All findings were correlational.

## Key findings

* Task group’s purchasing power remained linked to factors such as poverty, unemployment, the year, and the state after accounting for these variables.
* A one percentage point shift from non-routine cognitive to non-routine manual work resulted in a reduction in purchasing power of approximately $846.
* The relationship was better represented proportionally than as a constant dollar difference, which supported the log specification.
* Random forest models revealed additional structure, but the primary relationships were still evident in the regression results.
* The results described associations but did not support causal claims.

## Study design

|                         |                                                                              |
| ----------------------- | ---------------------------------------------------------------------------- |
| **Unit of analysis**    | County year                                                                  |
| **Study period**        | 2008 to 2023, excluding 2020                                                 |
| **Analytical sample**   | 848 counties and 11,983 county year observations                             |
| **Population coverage** | About 84% of the U.S. population                                             |
| **Outcome**             | Purchasing power in dollars                                                  |
| **Task groups**         | Routine cognitive, routine manual, non-routine cognitive, non-routine manual |
| **Economic controls**   | Poverty rate and unemployment rate                                           |
| **Regression**          | Year indicators with standard errors clustered by county                     |
| **Additional models**   | Random forest and neural network                                             |

The four task groups summed to one, so non-routine cognitive was used as the reference task group in the regression models.

## Data sources

| Source                                                        | Parameter                              |
| ------------------------------------------------------------- | ---------------------------------------- |
| Census Small Area Income and Poverty Estimates                | Median household income and poverty rate |
| Census American Community Survey 1 year estimates             | Occupational employment and population   |
| Bureau of Labor Statistics Local Area Unemployment Statistics | Unemployment rate                        |
| Bureau of Economic Analysis Regional Price Parities           | Local price levels                       |
| Census Core Based Statistical Area delineations               | County to metropolitan area mapping      |

Raw retrievals and intermediate results were stored in PostgreSQL. Data construction, statistical analysis, and modeling notebooks are maintained separately in the companion `project-workbook-chris-and-julian` repository.

## Repository contents

| File                  | Contents                                                   |
| --------------------- | ---------------------------------------------------------- |
| `capstone.qmd`        | Parent document that includes the seven section files      |
| `_01_intro.qmd`       | Introduction                                               |
| `_02_background.qmd`  | Background and related work                                |
| `_03_data.qmd`        | Data sources, measurement, and analytical sample           |
| `_04_analysis.qmd`    | Regression, diagnostics, and machine learning methods      |
| `_05_results.qmd`     | Findings                                                   |
| `_06_conclusions.qmd` | Conclusions, limitations, ethics, and future work          |
| `_07_references.qmd`  | References                                                 |
| `references.bib`      | Bibliography                                               |
| `_quarto.yml`         | Quarto project configuration                               |
| `_freeze/`            | Cached execution output used when rendering the manuscript |

Analysis notebooks and data construction code live in the companion `project-workbook-chris-and-julian` repository.

## Build and publish

The writeup uses Quarto and the `capstone` conda environment. The required Python interpreter can be specified explicitly when rendering:

```bash
QUARTO_PYTHON=/opt/anaconda3/envs/capstone/bin/python quarto render
```

Rendered output is written to `_output/`.

To publish an already rendered version:

```bash
quarto publish gh-pages --no-render
```

The `--no-render` option prevents Quarto from rendering the project again with a different Python interpreter during publication.

The `_freeze/` directory is committed so cached results can be reused without rerunning every model during each build.

## Limitations

* American Community Survey 1 year occupational estimates limited the analytical sample to larger counties, so rural counties were underrepresented.
* The 848 counties in the analytical sample contained about 84% of the U.S. population but represented a minority of U.S. counties.
* Counties outside metropolitan areas used their state Regional Price Parity because the Bureau of Economic Analysis did not publish a separate local price parity for those counties.
* The study identified associations and did not support causal conclusions.