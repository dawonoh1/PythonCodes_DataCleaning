# Seoul Park Accessibility: Data Cleaning

Python codes that turn a year of raw park-visitor flow data for Seoul into a modeling dataset for studying **who travels how far to which parks**. They clean and combine 12 monthly visitor files, compute walking and driving travel times on the OpenStreetMap network, combine the two modes into one logsum travel cost, attach park amenities, and estimate how amenities shape travel cost differently by sex and age.

The final model is a cross-classified random-effects model (CCREM). It is finished in R Markdown in the companion repository **ParkAccessibility_GenderDifference**, which has the full results.

## Data

- **2019 Seoul park visitor flows:** 12 monthly files (`Seoul_Park_201901.csv` to `Seoul_Park_201912.csv`). Each gives visitor counts from each origin district (gu) to each park, split into 28 sex × age groups. Only the 25 Seoul districts are kept as origins.
- **Park and origin coordinates**, plus **park attributes**: green-area ratio, user-space ratio, facilities, parking, and bus/subway stops within 500 m.
- **OpenStreetMap** street networks for Seoul, downloaded with OSMnx.

The data are not included in this repository. Notebooks expect the files in a `data/` folder next to them, with the monthly files in `data/seoul_park_monthly_2019/`.

## Notebooks

| Notebook | What it does |
|---|---|
| `01_osmnx_walk_network.ipynb` | Downloads Seoul's **walking** and **driving** networks with OSMnx, imputes missing edge speeds, and adds edge travel times |
| `02_park_accessibility_pipeline.ipynb` | The full pipeline (details below) |
| `03_ccrem_rerun_fixed_age.ipynb` | Re-runs the CCREM with two fixes: the open 70+ age band is coded as 75 instead of 35, and female × age terms are added and tested. It builds the long-format data, checks VIFs, and fits the model in R (`lme4`) through `rpy2`. |

**Steps in `02_park_accessibility_pipeline.ipynb`**

1. **Cleaning and merging.** Filters each monthly file to the 25 Seoul districts, stacks the 12 months, merges origin and park coordinates (Korean-encoded source files), and sums to annual origin–park visitor counts.
2. **Travel cost.** Walking and driving travel times between each origin and park are combined into one **logsum equivalent cost**:
   `c_eq = −(1/λ) · log(exp(−λ·t_walk) + exp(−λ·t_drive))` with λ = 0.1, computed in a numerically stable way.
3. **Accessibility.** Logsum (potential) accessibility for each origin district, overall and weighted by each sex × age group. Includes a sensitivity check over β, and plots by age and sex.
4. **Park attributes.** Merges amenity variables, checks their distributions, and drops pairs without complete park data (10,600 → 6,850 origin–park pairs).
5. **Models and marginal effects.** Regresses travel cost on park amenities, with interactions by sex, age and age², in a model with crossed random intercepts for park and origin district (fit in R `lme4` through `rpy2`). It plots each amenity's marginal effect by age group and sex, and the gender gap in those effects.

## Analytic sample

274 parks × 25 origin districts give 6,850 origin–park pairs. Splitting these into sex × age groups and dropping empty cells leaves **191,750 observations**. The final model has 41 fixed-effect terms.

## How to run

1. Create `data/` next to the notebooks and add the source files described above.
2. Use a Python environment with `pandas`, `numpy`, `scipy`, `matplotlib`, `osmnx` (1.5.1), `networkx` and `rpy2`. Notebook 03 also needs R with `lme4` and `lmerTest`.
3. Run notebooks 01 → 02 → 03 in order. Intermediate files (for example `FINAL_dist_logsum.csv` and `FINAL_dataset_2019only.csv`) are written to `data/`.

## Author

Dawon Oh: data cleaning, network travel-time computation, accessibility measures, and model specification.
