# Outcomes module

The SEIR module computes the infections dynamics on the defined spatial setup. The main output of this module is the daily new infection per spatial node.

The Outcomes modules computes any outcomes (such as incident and current hospitalisation, ICU, death) based on the SIER simulation. Indeed, the Outcomes module should be run after an SEIR simulation is performed.

### Config

```
#outcomes:
#  method: delayframe                   # Only fast is supported atm. Makes fast delay_table computations. Later agent-based method ?
#  paths:
#    param_from_file: TRUE               #
#    param_place_file: <path.csv>       # OPTIONAL: File with param per csv. For each param in this file 
#  scenarios:                           # Outcomes scenarios to run
#    - low_death_rate
#    - mid_death_rate
#  settings:                            # Setting for each scenario
#    low_death_rate:
#      new_comp1:                               # New compartement name 
#        source: incidence                      # Source of the new compartement: either an previously defined compartement or "incidence" for diffI of the SEIR
#        probability:  <random distribution>           # Branching probability from source
#        delay: <random distribution>                  # Delay from incidence of source to incidence of new_compartement
#        duration: <random distribution>               # OPTIONAL ! Duration in new_comp. If provided, the model add to it's output "new_comp1_curr" with current amount in new_comp1
#      new_comp2:                               # Example for a second compatiment
#        source: new_comp1                      
#        probability: <random distribution> 
#        delay: <random distribution> 
#        duration: <random distribution>
#      death_tot:                               # Possibility to combine compartements for death.
#        sum: ['death_hosp', 'death_ICU', 'death_incid']
#         
#    mid_death_rate:
#      ...
#
# ## Input Data
#
# * <b>{param_place_file}</b> is a csv with columns place, parameter, value. Parameter is constructed as, e.g for comp1:
#                probability: Pnew_comp1|source
#                delay:       Dnew_comp1
#                duration:    Lnew_comp1


# ## Output Data
# * {output_path}/model_output/{spatial_setup::setup_name}_[scenario]/[simulation ID].hosp.parquet

```

### Equivalent config for outcomes and hospitalization

The outcomes module is meant as a replacement of the hospitalization module. The config below produce the same result between hospitalization and outcomes.

```
hospitalization:
  paths:
    output_path: hospitalization
    run_age_adjust: TRUE
  parameters:
    time_hosp: [log(7), 0.3]
    time_disch: [log(11.5), log(1.22)]
    time_ICU: [log(3), 0.3]
    time_ICUdur: [log(8), 0.2]
    time_ventdur: [log(7), 0.2]
    time_vent: [log(1), 0.4]
    time_onset_death: [2.84, 0.52]
    p_death: [.0025, .005, .01]
    p_death_names: ["low","med","high"]
    p_hosp_inf: [0.025, 0.05, 0.1]

outcomes:
  method: delayframe        
  param_from_file: True
  param_place_file: COVIDScenarioPipeline/Outcomes/data/usa-geoid-params-output.parquet
  categories: ['10-20', '20-30']
  scenarios:
    - low
    - med
    - high
  #  - high_death_rate
  settings:
    high_death_rate:
      hospitalization:
        source: incidI
        probability:
          value:
            distribution: uniform
            low: .1
            high: .1
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 7
          name: hosp_curr
      deaths:
        source: hospitalization
        probability:
          value:
            distribution: fixed
            value: .01
        delay:
          value:
            distribution: fixed
            value: 7
    low:
      incidH:
        source: incidI
        probability:
          value:
            distribution: uniform
            low: .1
            high: .3
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 12
          name: hosp_curr
      incidICU:
        source: incidH
        probability:
          value:
            distribution: fixed
            value: 0.354506482   # Average of file
        delay:
          value:
            distribution: fixed
            value: 3
        duration:
          value:
            distribution: fixed
            value: 8
          name: icu_curr
      incidVent:
        source: incidICU
        probability:
          value:
            distribution: fixed
            value: 0.43866538
        delay:
          value:
            distribution: fixed
            value: 1
        duration:
          value:
            distribution: fixed
            value: 7
          name: vent_curr
      incidD:
        source: incidI
        probability:
          value:
            distribution: fixed
            value: 0.0025
        delay:
          value:
            distribution: fixed
            value: 20
    med:
      incidH:
        source: incidI
        probability:
          value:
            distribution: uniform
            low: .1
            high: .3
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 12
          name: hosp_curr
      incidICU:
        source: incidH
        probability:
          value:
            distribution: fixed
            value: 0.354506482   # Average of file
        delay:
          value:
            distribution: fixed
            value: 3
        duration:
          value:
            distribution: fixed
            value: 8
          name: icu_curr
      incidVent:
        source: incidICU
        probability:
          value:
            distribution: fixed
            value: 0.43866538
        delay:
          value:
            distribution: fixed
            value: 1
        duration:
          value:
            distribution: fixed
            value: 7
          name: vent_curr
      incidD:
        source: incidI
        probability:
          value:
            distribution: fixed
            value: 0.005
        delay:
          value:
            distribution: fixed
            value: 20
    high:
      incidH:
        source: incidI
        probability:
          value:
            distribution: uniform
            low: .1
            high: .3
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 12
          name: hosp_curr
      incidICU:
        source: incidH
        probability:
          value:
            distribution: fixed
            value: 0.354506482   # Average of file
        delay:
          value:
            distribution: fixed
            value: 3
        duration:
          value:
            distribution: fixed
            value: 8
          name: icu_curr
      incidVent:
        source: incidICU
        probability:
          value:
            distribution: fixed
            value: 0.43866538
        delay:
          value:
            distribution: fixed
            value: 1
        duration:
          value:
            distribution: fixed
            value: 7
          name: vent_curr
      incidD:
        source: incidI
        probability:
          value:
            distribution: fixed
            value: 0.01
        delay:
          value:
            distribution: fixed
            value: 20
      incidC:
        source: incidI
        probability:
          value:
            distribution: fixed
            value: 0.01
        delay:
          value:
            distribution: fixed
            value: 20      
```
