# Specifying observational model

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



Nothing changes except how to specify the source of outcomes that comes from the seir sims. Before `incidI` was defined as the incidence of individuals in compartment I1 (i.e new infected). Now there may be incidI for different variants, so the source from seir file is different.

Say we have a config with the compartment section of the `seir` module as:

```
  compartments:
    infection_stage: ["S", "E", "I1", "I2", "I3", "R"]
    vaccination_stage: ["0dose", "1dose", "2dose"]
    variant_type: ["var0", "var1"]
```

and we want `incidH` from `incidI`. There are a different ways:

### Backward compatibility (do not use, except for old configs)

Using `incidI` still works. In that case the source is the incidence (new arrivals) in the meta compartment `infection_stage` and compartment `I1`. Both has to be defined to use this compatibility option.

```
    incidH:
        source: incidI
        probability:
          value:
            distribution: fixed
            value: .1
...
```

### Filtering (simple example)

This does the same thing as before: select incidence in compartments `I1` (hence the source is the sum of the incidence in `I1_0dose_var0`, `I1_0dose_var1`, `I1_1dose_var0` and so on...

```
    incidH:
        source: 
          incidence:
            infection_stage: "I1"
...
```

this also works: it filters every dose and every variant, but just I1. Here we could e.g just filter for unvaccinated and 1 dose-

```
      incidH:                              
        source: 
          incidence:
            infection_stage: "I1"
            vaccination_stage: ["0dose", "1dose", "2dose"]
            variant_type: ["var0", "var1"]
...
```

### Full thing

Here we have an example were the probabilities from each dose are not the same necessarily, but we want still to have an `incidH` with all hospitalisations for the report and as a source for later outcomes such as incidICU

```
      incidH_0dose:
        source: 
          incidence:
            infection_stage: ["I1"]                             
            vaccination_stage: ["0dose"]
        probability:
          value:
            distribution: fixed
            value: .1
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 7
          name: incidH_0dose_curr
      incidH_1dose:
        source: 
          incidence:
            infection_stage: ["I1"]
            vaccination_stage: "1dose"
        probability:
          value:
            distribution: fixed
            value: .05
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 7
          name: incidH_1dose_curr
      incidH_2dose:
        source: 
          incidence:
            infection_stage: ["I1"]
            vaccination_stage: ["2dose"]
        probability:
          value:
            distribution: fixed
            value: .005
        delay:
          value:
            distribution: fixed
            value: 7
        duration:
          value:
            distribution: fixed
            value: 7
          name: incidH_2dose_curr
      incidH:                                                         # all incidH
        sum: ['incidH_2dose', 'incidH_1dose', 'incidH_0dose']
      incidH_curr:                                              
        sum: ['incidH_2dose_curr', 'incidH_1dose_curr', 'incidH_0dose_curr']
...
```

after this, something like `incidICU` can be configured with source any of the defined compartments (from all hosp: use `incidH`but from only unvaccinated use `incidH_0dose`).

Everything else work as before (e.g NPIs, ...). **But don't forget to do the sum of the durations also as it's not automatic** (maybe should be ?)

### Things to know for using the files

1. `p_comp` does not exist anymore in the hosp files. ==> removing the sum across p\_comp in R should do the trick if it was not using the different dosage.
2. `incidI` does not exist anymore in hosp files (except when using the backward compatibility way, where it is re-created). If you'd want that it has to be created as an outcome with probability 1 (and can be defined for all variants or only some).
3. when using relative probabilities (with `param_from_file`) the `source` column is not necessary anymore.
