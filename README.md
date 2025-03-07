# Age Verification Law Analysis Replication Instructions

This repository contains code to analyze the effects of age verification laws on internet search behavior. The analysis uses synthetic control methods to estimate treatment effects across multiple states and specifications.

The Manuscript can be found at the Open Science Foundation at this URL: https://osf.io/26jq4. 

## Prerequisites

### Required R Packages

```R
# Core packages
library(tidyverse)
library(tidylog)
library(here)
library(glue)

# Data manipulation and processing
library(janitor)
library(vroom)
library(furrr)
library(unglue)
library(lubridate)

# Synthetic control packages
library(gsynth)
library(augsynth)

# Visualization
library(patchwork)
library(ggrepel)
library(geofacet)

# Census data
library(tidycensus)
library(gtrendsR)
```

### API Keys
- You'll need a Census API key from https://api.census.gov/data/key_signup.html
- Create a file called `api_keys` (no extension) in the root directory with your census API key

## Project Structure

```
.
├── api_keys                              # Census API key file (user created)
│
├── data/                                 # Processed data directory
│   ├── census_data.csv                  # Generated by 01b_pull_census_data.R
│   ├── fsc_implementation_dates.csv     # Generated by 01c_scrape_implementation_dates.R
│   ├── porn.csv                        # Generated by 02_assemble_data.R
│   ├── pornhub.csv                     # Generated by 02_assemble_data.R
│   ├── vpn.csv                         # Generated by 02_assemble_data.R
│   ├── xvideos.csv                     # Generated by 02_assemble_data.R
│   └── analysis_data_[DATE].rds        # Generated by 02_assemble_data.R
│
├── data-raw/                            # Raw Google Trends data
│   ├── [STATE]_porn_[TIMESPAN].csv     # Generated by 01a_pull_gtrend_data.R
│   ├── [STATE]_pornhub_[TIMESPAN].csv
│   ├── [STATE]_vpn_[TIMESPAN].csv
│   └── [STATE]_xvideos_[TIMESPAN].csv
│
├── figures/                             # Generated visualizations
│   ├── [KEYWORD]_att_plot.png          # Generated by 03_preregistered_hypotheses.R
│   ├── [KEYWORD]_overall_plot.png
│   ├── [KEYWORD]_state_plots.png
│   ├── pre_registered_specification_plot.png
│   ├── pre-registered_cumulative_effects.png
│   ├── single_state_example.png        # Generated by 03_single_state_example.R
│   ├── multiverse_[STATE].png          # Generated by 04c_multiverse_plots.R
│   ├── multiverse_gsynth_[STATE].png   # Generated by 06c_multiverse_gsynth_plots.R
│   └── combined_multiverse.png         # Generated by 07_combined_multiverse_plots.R
│
├── tables/                              # Generated tables
│   └── single_synth_att.tex           # Generated by 03_single_state_example.R
│
├── model_weights/                       # Model weight outputs
│   └── single_synth_example.tex        # Generated by 03_single_state_example.R
│
├── multiverse_mod/                      # Multiverse analysis outputs
│   └── [PARAMS].rds                    # Generated by 04_multiverse.R
│
├── multiverse_tables/                   # Processed multiverse results
│   └── [INDEX].csv                     # Generated by 04b_multiverse-table_extraction.R
│
├── gsynth_output/                       # GSynth model outputs
│   └── [PARAMS].rds                    # Generated by 06_multiverse_gsynth.R
│
├── gsynth_tables/                       # Processed GSynth results
│   └── [PARAMS].csv                    # Generated by 06b_multiverse_gsynth.R
│
├── combined_multiverse/                 # Combined analysis results
│   ├── augsynth_multiverse.csv         # Generated by 04c_multiverse_plots.R
│   └── gsynth_results.csv              # Generated by 06c_multiverse_gsynth_plots.R
│
├── mods/                               # Additional model outputs
│   └── [KEYWORD]_model.rds            # Generated by 03_preregistered_hypotheses.R
│
└── scripts/                            # R scripts
    ├── 01a_pull_gtrend_data.R         # Google Trends data collection
    ├── 01b_pull_census_data.R         # Census data collection
    ├── 01c_scrape_implementation_dates.R # Implementation dates
    ├── 02_assemble_data.R             # Data assembly
    ├── 03_preregistered_hypotheses.R   # Initial analysis
    ├── 03_single_state_example.R       # Single state analysis
    ├── 04_multiverse.R                # Multiverse analysis
    ├── 04b_multiverse-table_extraction.R # Results extraction
    ├── 04c_multiverse_plots.R         # Multiverse visualization
    ├── 06_multiverse_gsynth.R         # GSynth analysis
    ├── 06b_multiverse_gsynth.R        # GSynth results processing
    ├── 06c_multiverse_gsynth_plots.R  # GSynth visualization
    └── 07_combined_multiverse_plots.R  # Combined results visualization
```

### File Naming Patterns

#### Data Files
- Raw Google Trends data: `[STATE]_[KEYWORD]_[TIMESPAN].csv`
  - Example: `CA_porn_2022-01-01 2024-10-31.csv`

#### Multiverse Files
- Multiverse models: `[KEYWORD]_[TIMERANGE]_[COVARIATES]_scm[SCM]_fixedeff[FIXEDEFF]_leads[LEADS]_lags[LAGS]_treatment_[TREATMENT]_verification_method_[METHOD].rds`
  - Example: `pornhub_2022-01-01_2024-10-31_with_covariates_scmTRUE_fixedeffFALSE_leads52_lags13_treatment_post_treat_verification_method_government_id.rds`

#### GSynth Files
- GSynth models: `[KEYWORD]_[TIMERANGE]_treatment_[TREATMENT]_verification_[METHOD]_force_[FORCE]_estimator_[ESTIMATOR].rds`
  - Example: `pornhub_2022-01-01_2024-10-31_treatment_post_treat_verification_government_id_force_none_estimator_ife.rds`

## Execution Order

1. **Data Collection and Processing**
   - `01a_pull_gtrend_data.R`: Pulls Google Trends data for specified search terms
   - `01b_pull_census_data.R`: Collects census demographic data
   - `01c_scrape_implementation_dates.R`: Compiles law implementation dates
   - `02_assemble_data.R`: Combines all data sources

2. **Initial Analysis**
   - `03_preregistered_hypotheses.R`: Runs primary analysis
   - `03_single_state_example.R`: Analyzes single state example

3. **Multiverse Analysis**
   - `04_multiverse.R`: Runs multiple synthetic control specifications
   - `04b_multiverse-table_extraction.R`: Extracts results from multiverse analysis
   - `04c_multiverse_plots.R`: Creates visualization of multiverse results

4. **GSynth Analysis**
   - `06_multiverse_gsynth.R`: Runs GSynth specifications
   - `06b_multiverse_gsynth.R`: Processes GSynth results
   - `06c_multiverse_gsynth_plots.R`: Creates GSynth visualizations

5. **Combined Analysis**
   - `07_combined_multiverse_plots.R`: Creates combined visualizations of all results

## Running the Analysis

1. Set up your R environment and install required packages
2. Create and populate your `api_keys` file
3. Run scripts in the order listed above
4. Check output directories for results

## Notes

- Some scripts use parallel processing. Adjust the number of cores used based on your system capabilities
- Large data files may be generated in the process. Ensure adequate disk space
- Scripts contain progress indicators and will skip already-completed computations
- The multiverse analysis may take several hours to complete depending on your system
- `[DATE]` in filenames refers to the date the file was generated
- `[PARAMS]` represents various parameter combinations used in the analysis
- `[INDEX]` represents numerical indices for generated tables
- Files marked with `[STATE]` are generated for each U.S. state in the analysis
- `[KEYWORD]` represents one of the search terms: porn, pornhub, vpn, xvideos

## Troubleshooting

- If you encounter memory issues, reduce the number of parallel workers in scripts using `furrr` or `parallel`
- For Census API rate limits, add delays between calls
- Check the data directories are properly created and have write permissions

## Output

The analysis will generate:
- Synthetic control estimates
- Treatment effect plots
- Multiverse analysis results
- Combined visualization of results across methods

Results will be saved in their respective directories as specified in the directory structure.
