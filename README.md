# Automation and Affordability in U.S. Counties

**Is the task composition of local work associated with what residents can afford?**

Chris Bell | Julian Pacheco
DATA 510 Data Science Capstone, Willamette University | Summer 2026

**Live writeup:** https://wu-msds-capstones.github.io/project-writeup-chris-and-julian/

## About

This project asks whether the kind of work a county does relates to what its residents can afford, once local price levels are accounted for. We build a county by year panel covering 2008 to 2023, excluding 2020, from five federal data sources. Each county year is summarized by four task groups drawn from the task framework of Autor, Levy and Murnane (2003) and the occupation taxonomy of Autor and Dorn (2013): routine cognitive, routine manual, non routine cognitive, and non routine manual.

The outcome is purchasing power, measured in dollars as county median household income divided by the local price level from the BEA Regional Price Parities. Prior work in this area measures outcomes in unadjusted wages at the commuting zone level; adjusting for local prices at the county level is what this project adds.

The analytical panel covers 848 counties and 11,983 county year observations. All findings are associational.

## Repository contents

| File | Contents |
|---|---|
| `capstone.qmd` | Parent document; the seven section files are included into it |
| `_01_intro.qmd` | Introduction |
| `_02_background.qmd` | Background and related work |
| `_03_data.qmd` | Data sources, measurement, and the analytical sample |
| `_04_analysis.qmd` | Panel regression, specification diagnostics, machine learning methods |
| `_05_results.qmd` | Findings |
| `_06_conclusions.qmd` | Conclusions, limitations, ethics, future work |
| `_07_references.qmd` | References |
| `references.bib` | Bibliography |
| `_quarto.yml` | Project configuration |
| `_freeze/` | Cached render output, committed so the manuscript builds without rerunning every model |

Analysis notebooks live in the companion repository, `project-workbook-chris-and-julian`.

## Data sources

| Source | Supplies |
|---|---|
| Census SAIPE | Median household income, poverty rate |
| Census ACS 1 year estimates | Occupational employment, population |
| BLS LAUS | Unemployment rate |
| BEA Regional Price Parities | County price level relative to the national average |
| Census CBSA delineation | County to metropolitan area crosswalk |

Data are held in two PostgreSQL databases on Railway: a data lake of raw retrievals and intermediate results, and a normalized analytical warehouse of eleven tables.

## Building


```
quarto render capstone.qmd
```

Output lands in `_output/`. To publish:

```
quarto publish gh-pages
```

Credentials are never committed. `_freeze/` is committed so that sections requiring packages not present on every machine, notably the neural network, do not need to rerun on each build.

## Methods

Panel regression with year indicators and standard errors clustered by county, checked with residual plots, quantile quantile plots, and variance inflation factors. The four task groups sum to one, so one enters as the reference category; non routine cognitive is used throughout.

Diagnostics show the relationship is proportional rather than additive in dollars, which motivates a log respecification and, beyond that, testing whether more flexible models capture structure the linear specification cannot. Random forest and neural network models are fit on a split grouped by county, so every held out prediction comes from a county the model has never seen.

## Limitations

The ACS publishes occupational estimates only for areas above 65,000 residents, so the panel covers 848 counties holding about 84 percent of the United States population but a minority of its counties. Rural counties are underrepresented. Counties outside metropolitan areas take their state price parity, since BEA publishes no local parity for them. All findings are associational, not causal.
