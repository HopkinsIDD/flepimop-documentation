# Outcomes for compartments

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
