# Other configuration options

* Stochastic vs deterministic
* Integration method (`seir:integration`)
* Anything about automatic report generation
* The entire `importation` section (seed based on airline data)

## `importation` section (optional) 

This section is optional. It is used by the [covidImportation package](https://github.com/HopkinsIDD/covidImportation) to import global air importation data for seeding infections into the United States.

If you wish to include it, here are the options.

| Config Item                   | Required?    | Type/Format | Description | 
|-------------------------------|--------------|-------------|-------------|
| census_api_key                | **required** | string      | [get an API key](https://api.census.gov/data/key_signup.html)|
| travel_dispersion             | **required** | number      | how dispersed daily travel data is; default = 3. |
| maximum_destinations          | **required** | integer     | Number of airports to limit importation to | 
| dest_type                     | **required** | categorical | location type |
| dest_country                  | **required** | string (Country) | ISO3 code for country of importation. Currently only USA is supported |
| aggregate_to                  | **required** | categorical | location type to aggregate to |
| cache_work                    | **required** | boolean | whether to save case data |
| update_case_data              | **required** | boolean | deprecated; whether to update the case data or used saved |
| draw_travel_from_distribution | **required** | boolean | whether to add additional stochasticity to travel data; default is FALSE |
| print_progress                | **required** | boolean | whether to print progress of importation model simulations |
| travelers_threshold           | **required** | integer | include airports with at least the `travelers_threshold` mean daily number of travelers | 
| airport_cluster_distance      | **required** | numeric | cluster airports within `airport_cluster_distance` km |
| param_list                    | **required** | See section below | see below |

### `importation::param_list`

| Config Item            | Required?    | Type/Format | Description |
|------------------------|--------------|-------------|-------------|
| incub_mean_log         | **required** | numeric | incubation period, log mean |
| incub_sd_log           | **required** | numeric | incubation period, log standard deviation |
| inf_period_nohosp_mean | **required** | numeric | infectious period, non-hospitalized, mean |
| inf_period_nohosp_sd   | **required** | numeric | infectious period, non-hospitalized, sd |
| inf_period_hosp_mean_log  | **required** | numeric | infectious period, hospitalized, log-normal mean |
| inf_period_hosp_sd_log  | **required** | numeric | infectious period, hospitalized, log-normal sd| 
| p_report_source        | **required** | numeric | reporting probability, Hubei and elsewhere |
| shift_incid_days       | **required** | numeric | mean delay from infection to reporting of cases; default = -10 |
| delta                  | **required** | numeric | days per estimations period |

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

| Config Item                       | Required?    | Type/Format     | Description               |
|-----------------------------------|--------------|-----------------|---------------------------|
| data_settings::pop_year           | | integer  | |
| plot_settings::plot_intervention  | | boolean | |
| formatting::scenario_labels_short | | list of strings; one for each scenario in `interventions::scenarios` | |
| formatting::scenario_labels       | | list of strings; one for each scenario in `interventions::scenarios` | |
| formatting::scenario_colors       | | list of strings; one for each scenario in `interventions::scenarios` | |
| formatting::pdeath_labels         | | list of strings | |
| formatting::display_dates         | | list of dates | |
| formatting::display_dates2         |optional | list of dates | a 2nd string of display dates that can optionally be supplied to specific report functions |

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