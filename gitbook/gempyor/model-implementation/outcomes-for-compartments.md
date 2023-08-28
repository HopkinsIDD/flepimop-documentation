---
description: >-
  This page describes how to specify the outcomes section of the configuration
  file
---

# Specifying observational model

## Thinking about `outcomes` variables

Our pipeline allows users to encode variables in two different ways. The first way is via the state variables of the compartmental model if disease transmission, which are specified ... . This model should include all variables that influence the natural course of the epidemic (ie, all variables that feed back into the model by influencing the rate of change of other variables). For example, the number of infected individuals influences the rate at which new infections occur, and the number of immune individuals influences the number of individuals at risk of acquiring infection.&#x20;

However, these intrinsic model parameters may be difficult to observe in the real world and so directly comparing model predictions to data might not make sense. Instead, the observable outcomes of infection may include only a subset of individuals in any state, and may only be observed with a time delay. Thus, we allow users to define new `outcome` variables that are functions of the underlying model variables.

There are several benefits to encoding variables as `outcomes` wherever possible. Firstly, because these variables don't feed back to the model they can be calculated after the model run is completed, and different variations of the rules for determining these outcomes can be calculated on the same model output (for example, a scenario with high case detection rate and another with low case detection rate).  Secondly, `outcome` variables can be specified to occur stochastically, even if the model is run completely deterministically. This allows users to capture stochasticity that may be important for rare outcomes (e.g., severe disease or death), even while saving computational time when running the original model. Thirdly, `outcome` variables can be specified to occur with arbitrary delay distributions that are difficult to encode in traditional (Markovian) compartmental models of infectious disease dynamics.&#x20;

Variables should not be included as outcomes if they influence the infection trajectory. The choice of what variables to include in the compartmental disease model vs the outcomes section may be very model specific. For example, hospitalizations due to infection could be encoded as an outcome variable that is some fraction of infections, but if we believe hospitalized individuals are isolated from the population and don't contribute to onwards infection, this would not be the best choice. Similarly, we could include deaths due to infection as an outcome variable that is also some fraction of infections, but if we think that the incidence of deaths feeds back into the population's perception of risk of infection and influences contact behavior (and if we are including such feed backs in our model), then deaths should be in the compartmental model instead.&#x20;

The `outcomes` section is not required in the config. However, it is highly recommended to include it, even if the only outcome variable is set to be equivalent to one of the infection model variables. If the model is being fit to data, then the `outcomes` section is required, as only outcome variables can be compared to data.&#x20;

As an example, imagine we are simulating an SIR-style model and want to compare it to real epidemic data in which cases of infection and death from infection are reported. Our model doesn't explicitly include death, but supposed we know that 1% of all infections eventually lead to death, and that death occurs on average 3 weeks after infection. We know that not all infections are reported as cases, but aren't sure of the case detection rate, and so consider two possibilities: that 90% of infections are detected, and that only 50% are detected. Assume we know that cases are reported 2 days after infection begins. The model and `outcomes` section of the config for these outcomes, which we call `incidC` (daily incidence of cases) and `incidD` (daily incidence of death) would be

<pre><code>compartments:
  infection_state: ["S", "I", "R"]
  
seir:
  transitions:
    # infection
    - source: [S]
      destination: [I]
      proportional_to: [[S], [I]]
      rate: [beta]
      proportion_exponent: 1
    # recovery
    - source: [I]
      destination: [R]
      proportional_to: [[I]]
      rate: [gamma]
      proportion_exponent: 1
  parameters:
    beta: 0.1
    gamma: 0.2
<strong>
</strong><strong>outcomes:
</strong>  method: delayframe
  scenarios:
    - high_case_detection_scenario
    - low_case_detection_scenario
  settings:
      high_case_detection_scenario:
        incidC:
          source:
            incidence:
              infection_state: "I"
          probability: 0.9
          delay: 2
        incidD:
          source:
            incidence:
              infection_state: "D"
          probability: 0.01
          delay: 21 
      low_case_detection_scenario:
        incidC:
          source:
            incidence:
              infection_state: "I"
          probability: 0.5
          delay: 2
        incidD:
          source:
            incidence:
              infection_state: "D"
          probability: 0.01
          delay: 21 
</code></pre>

in the following sections we describe in more detail how this specification works

## Specifying `outcomes` in the configuration file

### outcomes::method



### outcomes::paths

### outcomes::scenarios



### outcomes::settings

#### Source

#### Probability

#### Delay

#### Duration

## Examples

Consider a disease described by an SIR model in a population that is divided into two age groups, adults and children, which experience the disease separately. We are interested in comparing the predictions of the model to real-world data, but we know we cannot observe every infected individual. Instead, we have two types of outcomes that are observed.&#x20;

First, via syndromic surveillance, we have a database that records how many individuals in the population are experiencing symptoms from the disease at any given time. Suppose careful cohort studies have shown that 50% of infected adults and 80% of infected children will develop symptoms, and that symptoms occur in both age groups around 3 days after infection (following a log-normal distribution with logmean X and log standard deviation of Y). The duration that symptoms persist is also variable, following a ...&#x20;

Secondly, via laboratory surveillance we have a database of every positive test result for the infection. We assume the test is 100% sensitive and specific. Only individuals with symptoms are tested, and they are always tested exactly 1 day after their symptom onset. We are unsure what portion of symptomatic individuals are seeking out testing, but are interested in considering two extreme scenarios: 95% of symptomatic individuals are tested, or only 75% of individuals are tested.&#x20;

The configuration file we could use to model this situation includes:

```
// Some code

compartments:

seir:

outcomes:
  method: delayframe
  paths:
    params_from_file: FALSE
  scenarios:
    - high_testing
    - low_testing
  settings:
    high_testing:
      incidC_adults:
      incidC_children:
      incidC_total:
      incidT_total:
        source: incidC_total
        probability:
        delay:
        distribution:
    low_testing:
```



### OLD

```
#outcomes:
#  method: delayframe                   # Only fast is supported atm. Makes fast delay_table computations. Later agent-based method ?
#  paths:
#    param_from_file: TRUE               #
#    param_subpop_file: <path.csv>       # OPTIONAL: File with param per csv. For each param in this file 
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
# * <b>{param_subpop_file}</b> is a csv with columns subpop, parameter, value. Parameter is constructed as, e.g for comp1:
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
