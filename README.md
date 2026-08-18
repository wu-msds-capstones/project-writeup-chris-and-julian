# Automation and Affordability in U.S. Counties

**[Read the project](https://wu-msds-capstones.github.io/automation-and-affordability-us-counties/)**

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

## Project design

|                         |                                                                              |
| ----------------------- | ---------------------------------------------------------------------------- |
| **Unit of analysis**    | County year                                                                  |
| **Project period**      | 2008 to 2023, excluding 2020                                                 |
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

### What you need

| Requirement | Detail |
| --- | --- |
| Quarto | v1.8.25 or later, on macOS (Apple silicon) |
| Python | 3.12, supplied by a conda environment named `capstone` |
| R | Installed inside the same conda environment, reached from Python through rpy2 |
| Database access | Two environment variables, `AUTORACK_URL` and `THOMAS_URL` |

Every figure in the project is a ggplot drawn through rpy2, and every number is queried live from PostgreSQL, so R and both database URLs are required rather than optional.

### Environment setup

Create the conda environment and install the Python and R packages the section files import:

```bash
conda create -n capstone python=3.12
conda install -n capstone -c conda-forge \
  r-base rpy2 r-ggplot2 r-scales r-patchwork
conda run -n capstone pip install \
  tensorflow scikit-learn statsmodels pandas numpy \
  sqlalchemy psycopg2-binary geopandas graphviz great_tables \
  jupyter ipykernel
```

Register the environment as the `python3` Jupyter kernel, because Quarto resolves the kernel by name and will otherwise pick whichever Python kernel it finds first:

```bash
/opt/anaconda3/envs/capstone/bin/python -m ipykernel install --user --name python3
```

### Database credentials

Both databases are hosted on Railway and are read at render time. Export the connection strings before rendering, and never write them into a file in this repository:

```bash
export AUTORACK_URL='postgresql://...'   # analytical warehouse; most cells query this
export THOMAS_URL='postgresql://...'     # data lake; the task framework figure queries this
```

`AUTORACK_URL` alone is not enough. One cell in `_02_background.qmd` reads individual occupational categories from the lake, since the warehouse stores only the four aggregated task groups.

### Rendering

The section files carry Python cells, so Quarto runs the project through the jupyter engine and holds one interpreter for the whole render. Point it at the conda environment explicitly; the shell default Python is incompatible with TensorFlow and the render fails partway through rather than at the start.

```bash
# Full render of the manuscript
QUARTO_PYTHON=/opt/anaconda3/envs/capstone/bin/python quarto render

# Single section, while iterating
QUARTO_PYTHON=/opt/anaconda3/envs/capstone/bin/python quarto render _03_data.qmd
```

Rendered output is written to `_output/`, and figures are written to `output/` at 300 dpi.

Single file renders work only for sections without R cells. The `%%R` magic is registered by `capstone.qmd`, so `_02_background.qmd`, `_04_analysis.qmd`, and `_05_results.qmd` can be checked only through a full render.

### The freeze cache

The `_freeze/` directory is committed so a rebuild can reuse cached results instead of rerunning every model, notably the neural network. Its hash covers `capstone.qmd` alone, so edits inside an included section file do not invalidate it and a full render will silently reuse the stale execution. After editing a section file, clear the cache first:

```bash
rm -rf _freeze/capstone .quarto/_freeze/capstone
QUARTO_PYTHON=/opt/anaconda3/envs/capstone/bin/python quarto render
```

Then confirm the edited prose actually appears in `_output/index.html` before publishing. Clear the whole of `_freeze/` only when the environment itself changes, such as a package upgrade or a new Quarto version.

### Publishing

Render first, confirm the output, then publish what was already built:

```bash
quarto publish gh-pages --no-render
```

The `--no-render` option matters. Without it, Quarto renders again outside the `QUARTO_PYTHON` prefix and picks the wrong interpreter. Quarto manages the `gh-pages` branch itself, so do not check it out or edit it by hand.

### If a render fails

| Symptom | Cause |
| --- | --- |
| Dies on a TensorFlow import | The `QUARTO_PYTHON` prefix was omitted |
| `FileNotFoundError` on an unrelated Python path | A stale user level Jupyter kernelspec; remove it with `jupyter kernelspec uninstall <name>` |
| A ggplot cell fails, or R objects behave oddly | rpy2 found a system R rather than the conda R; check the `R_HOME` pin in `capstone.qmd` still runs before the rpy2 extension loads |
| Connection error partway through | Railway dropped an idle connection; engines use `pool_pre_ping=True` and `pool_recycle=300`, and `engine.dispose()` clears a stale one |
| Edited prose missing from the output | The freeze cache was not cleared after a section file edit |

## Limitations

* American Community Survey 1 year occupational estimates limited the analytical sample to larger counties, so rural counties were underrepresented.
* The 848 counties in the analytical sample contained about 84% of the U.S. population but represented a minority of U.S. counties.
* Counties outside metropolitan areas used their state Regional Price Parity because the Bureau of Economic Analysis did not publish a separate local price parity for those counties.
* The project identified associations and did not support causal conclusions.
