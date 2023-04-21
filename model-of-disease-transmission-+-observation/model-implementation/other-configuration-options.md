# Other configuration options

* Stochastic vs deterministic
* Integration method (`seir:integration`)
* Anything about automatic report generation
* The entire `importation` section (seed based on airline data)

## US-specific options

global: smh\_round, setup\_name, disease

spatial\_setup: census\_year, modeled\_states, state\_level

#### For US-specific population structures

For creating US-based population structures using the helper script `build_US_setup.R` which is run before the main model simulation script, the following extra parameters can be specified

| Config Item     | Required? | Type/Format            | Description                                                                                                         |
| --------------- | --------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------- |
| census\_year    | optional  | integer (year)         | Determines the year for which census population size data is pulled.                                                |
| state\_level    | optional  | boolean                | Determines whether county-level population-size data is instead grouped into state-level data (TRUE). Default FALSE |
| modeled\_states | optional  | list of location codes | A vector of locations that will be modeled; others will be ignored                                                  |

#### Example 2

To simulate an epidemic across all 50 states of the US or a subset of them, users can take advantage of built in machinery to create geodata and mobility files for the US based on the population size and number of daily commuting trips reported in the US Census.&#x20;

Before running the simulation, the script `build_US_setup.R` can be run to get the required population data files from online census data and filter out only states/territories of interest for the model. More details are provided in the How to Run section.&#x20;

This example simulates COVID-19 in the New England states, assuming no transmission from other states, using 2019 census data for the population sizes and a pre-created file for estimated interstate commutes during the 2011-2015 period.

```
spatial_setup:
  census_year: 2010
  state_level: TRUE
  geodata: geodata_2019_statelevel.csv
  mobility: mobility_2011-2015_statelevel.csv
  popnodes: pop2019est
  nodenames: geoid
  modeled_states:
    - CT
    - MA
    - ME
    - NH
    - RI
    - VT
  
```

`geodata.csv` contains&#x20;

```
USPS	geoid	pop2019est
AL	01000	4876250
AK	02000	737068
AZ	04000	7050299
AR	05000	2999370
CA	06000	39283497
.....
```

`mobility_2011-2015_statelevel.csv` contains

```
ori	dest	amount
01000	02000	198
01000	04000	292
01000	05000	570
01000	06000	1030
01000	08000	328
.....
```

## `importation` section (optional)

This section is optional. It is used by the [covidImportation package](https://github.com/HopkinsIDD/covidImportation) to import global air importation data for seeding infections into the United States.

If you wish to include it, here are the options.

| Config Item                      | Required?    | Type/Format       | Description                                                                             |
| -------------------------------- | ------------ | ----------------- | --------------------------------------------------------------------------------------- |
| census\_api\_key                 | **required** | string            | [get an API key](https://api.census.gov/data/key\_signup.html)                          |
| travel\_dispersion               | **required** | number            | how dispersed daily travel data is; default = 3.                                        |
| maximum\_destinations            | **required** | integer           | Number of airports to limit importation to                                              |
| dest\_type                       | **required** | categorical       | location type                                                                           |
| dest\_country                    | **required** | string (Country)  | ISO3 code for country of importation. Currently only USA is supported                   |
| aggregate\_to                    | **required** | categorical       | location type to aggregate to                                                           |
| cache\_work                      | **required** | boolean           | whether to save case data                                                               |
| update\_case\_data               | **required** | boolean           | deprecated; whether to update the case data or used saved                               |
| draw\_travel\_from\_distribution | **required** | boolean           | whether to add additional stochasticity to travel data; default is FALSE                |
| print\_progress                  | **required** | boolean           | whether to print progress of importation model simulations                              |
| travelers\_threshold             | **required** | integer           | include airports with at least the `travelers_threshold` mean daily number of travelers |
| airport\_cluster\_distance       | **required** | numeric           | cluster airports within `airport_cluster_distance` km                                   |
| param\_list                      | **required** | See section below | see below                                                                               |

### `importation::param_list`

| Config Item                  | Required?    | Type/Format | Description                                                    |
| ---------------------------- | ------------ | ----------- | -------------------------------------------------------------- |
| incub\_mean\_log             | **required** | numeric     | incubation period, log mean                                    |
| incub\_sd\_log               | **required** | numeric     | incubation period, log standard deviation                      |
| inf\_period\_nohosp\_mean    | **required** | numeric     | infectious period, non-hospitalized, mean                      |
| inf\_period\_nohosp\_sd      | **required** | numeric     | infectious period, non-hospitalized, sd                        |
| inf\_period\_hosp\_mean\_log | **required** | numeric     | infectious period, hospitalized, log-normal mean               |
| inf\_period\_hosp\_sd\_log   | **required** | numeric     | infectious period, hospitalized, log-normal sd                 |
| p\_report\_source            | **required** | numeric     | reporting probability, Hubei and elsewhere                     |
| shift\_incid\_days           | **required** | numeric     | mean delay from infection to reporting of cases; default = -10 |
| delta                        | **required** | numeric     | days per estimations period                                    |

```
importation:
  census_api_key: "fakeapikey00000"
  travel_dispersion: 3
  maximum_destinations: Inf
  dest_type: state
  dest_county: USA
  aggregate_to: airport
  cache_work: TRUE
  update_case_data: TRUE
  draw_travel_from_distribution: FALSE
  print_progress: FALSE
  travelers_threshold: 10000
  airport_cluster_distance: 80
  param_list:
    incub_mean_log: log(5.89)
    incub_sd_log: log(1.74)
    inf_period_nohosp_mean: 15
    inf_period_nohosp_sd: 5
    inf_period_hosp_mean_log: 1.23
    inf_period_hosp_sd_log: 0.79
    p_report_source: [0.05, 0.25]
    shift_incid_days: -10
    delta: 1
```

## `report` section

The `report` section is completely optional and provides settings for making an R Markdown report. For an example of a report, see the Supplementary Material of our [preprint](https://www.medrxiv.org/content/10.1101/2020.06.11.20127894v1)

If you wish to include it, here are the options.

| Config Item                         | Required? | Type/Format                                                          | Description                                                                                |
| ----------------------------------- | --------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| data\_settings::pop\_year           |           | integer                                                              |                                                                                            |
| plot\_settings::plot\_intervention  |           | boolean                                                              |                                                                                            |
| formatting::scenario\_labels\_short |           | list of strings; one for each scenario in `interventions::scenarios` |                                                                                            |
| formatting::scenario\_labels        |           | list of strings; one for each scenario in `interventions::scenarios` |                                                                                            |
| formatting::scenario\_colors        |           | list of strings; one for each scenario in `interventions::scenarios` |                                                                                            |
| formatting::pdeath\_labels          |           | list of strings                                                      |                                                                                            |
| formatting::display\_dates          |           | list of dates                                                        |                                                                                            |
| formatting::display\_dates2         | optional  | list of dates                                                        | a 2nd string of display dates that can optionally be supplied to specific report functions |

```
report:
  data_settings:
    pop_year: 2018
  plot_settings:
    plot_intervention: TRUE
  formatting:
    scenario_labels_short: ["UC", "S1"]
    scenario_labels:
      - Uncontrolled
      - Scenario 1
    scenario_colors: ["#D95F02", "#1B9E77"]
    pdeath_labels: ["0.25% IFR", "0.5% IFR", "1% IFR"]
    display_dates: ["2020-04-15", "2020-05-01", "2020-05-15", "2020-06-01", "2020-06-15"]
    display_dates2: ["2020-04-15", "2020-05-15", "2020-06-15"]
```
