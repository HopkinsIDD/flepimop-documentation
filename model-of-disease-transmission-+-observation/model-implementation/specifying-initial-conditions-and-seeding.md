---
description: >-
  This section describes how to specify the values of each model state at the
  time the simulation starts, and how to make instantaneous changes to state
  values at other times (e.g., due to importations)
---

# Specifying initial conditions and seeding

## Overview

In order for the models specified previously to be dynamically simulated, the user must provide _initial conditions,_ in addition to the model structure and parameter values. Initial conditions describe the value of each variable in the model at the time point that the simulation is to start. For example, on day zero of an outbreak, we may assume that the entire population is susceptible except for one single infected individual. Alternatively, we could assume that some portion of the population already has prior immunity due to vaccination or previous infection. Different initial conditions lead to different model trajectories.&#x20;

FlepiMoP also allows users to specify instantaneous changes in values of model variables, at later times in the simulation. We call this "seeding". For example, some individuals in the population may travel or otherwise acquired infection from outside the population throughout the epidemic, and this important of infection could be specified with the seeding option. As another example, new genetic variants of the pathogen may arise due to mutation and selection that occurs within infected individuals, and this generation of new strains can also be modeled with seeding. Seeding allows individuals to change state at specified times in ways that do not depend on the model equations. In the first example, the individuals would be "seeded" into the infected compartment from the susceptible compartment, and in the second example, individuals would be seeded into the "infected with new variant" compartment from the "infected with wild type" compartment.

The seeding option can also be used as a convenient alternative way to specify initial conditions. By default, FlepiMoP initiates models by putting the entire population size (specified in the geodata file) in the first model compartment. If the desired initial condition is only slightly different than the default state, it may be more convenient to specify it with a few "seedings" that occur on the first day of the simulation. For example, for a simple SIR model where the desired initial condition is just a small number of infected individuals, this could be specified by a single seeding into the infected compartment from the susceptible compartment at time zero, instead of specifying the initial values of three separate compartments. For larger models, the difference becomes more relevant.&#x20;

The `seeding` and `initial_conditions` section of the configuration file are detailed below.&#x20;

This table is useful for a quick comparison of these sections

| Feature                               | initial\_conditions                                                                                                                                                                                                    | seeding                                                                                                                                                                                                          |
| ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Config section optional or required?  | Optional                                                                                                                                                                                                               | Optional                                                                                                                                                                                                         |
| Function of section                   | Specify number of individuals in each compartment at time zero                                                                                                                                                         | Allow for instantaneous changes in individuals' states                                                                                                                                                           |
| Default                               | Entire population in first compartment, zero in all other compartments                                                                                                                                                 | No seeding events. Entire population in first compartment, zero in all other compartments.                                                                                                                       |
| Requires input file?                  | Yes, .csv                                                                                                                                                                                                              | Yes, .csv                                                                                                                                                                                                        |
| Input description                     | Input is a list of compartment names, location names, and amounts of individuals in that compartment-location. All compartments must be listed unless a setting to default missing compartments to zero is turned on.  | Input is list of seeding events defined by source compartment, destination compartment, number of individuals transitioning, and date of movement. Compartments without seeding events don't need to be listed.  |
| Specifies an incidence or prevalence? | Amounts specified are prevalence values                                                                                                                                                                                | Amounts specified are instantaneous incidence values                                                                                                                                                             |
| Useful for?                           | Specifying initial conditions, especially if simulation does not start with a single infection introduced into a naive population.                                                                                     | Modeling importations, evolution of new strains, and specifying initial conditions                                                                                                                               |



## Specifying model seeding

```
seeding:
  method: FromFile
  seeding_file: data/seeding_2pop.csv
```

### seeding::method

FromFile

PoissonDistributed

NegativeBinomialDistributed

FolderDraw

### seeding::seeding\_file

### seeding::lambda\_file

The configuration items in the `seeding` section of the config file are

`seeding:method` Must be either "`NoSeeding"`, "`FromFile"`, "`PoissonDistributed"`, "`NegativeBinomialDistributed"`, or "`FolderDraw".`

`seeding::seeding_file` Only required for `method: “FromFile”.` Path to a .csv file containing the list of seeding events

`seeding::lambda_file` Only required for methods "`PoissonDistributed"` or "`NegativeBinomialDistributed".` Path to a .csv file containing the list of the events from which the actual seeding will be randomly drawn.&#x20;

`seeding::seeding_file_type`  Only required for method "`FolderDraw".` Either `seir` or `seed`

Details on implementing each seeding method and the options that go along with it are below

### seeding::method

#### &#x20;NoSeeding

If there is no seeding, then the amount of individuals in each compartment will be initiated using the values specified in the`initial_conditions` section and will only changed at later times based on the equations defined in the `seir` section. No other arguments are needed in the seeding section in this case

Example

```
seeding:
    method: “NoSeeding”
```

#### FromFile

This seeding method reads in a user-defined file with a list of seeding events (instantaneous transitions of individuals between compartments) including the time and place of the event, and the source and destination compartment of the individuals. For example, for the simple two-subpopulation SIR model where the outbreak starts with an 5 individuals in the small province from being infected from a source outside the population, the seeding section of the config could be specified as

```
seeding:
  method: "FromFile"
  seeding_file: seeding.csv
```

Where seeding.csv contains:

```
place, date, amount, source_infection_stage, destination_infection_stage
small_province, 2020-02-01, 5, S, E
```

`seeding::seeding_file` must contain the following columns:

* `place` - the name of the subpopulation (geoid) in which the seeding event takes place. Seeding cannot move individuals between different subpopulations.&#x20;
* `date` - the date the seeding event occurs, in YYYY-MM-DD format
* `amount` - an integer value for the the amount of individuals who transition between states in the seeding event
* `source_*`  and  `destination_*` - For each compartment group (ie infection stage, vaccination stage, age group), a different column describes the status of individuals before and after the transition described by the seeding event. For example, for a model where individuals are stratified by age and vaccination status,  and a 1-day vaccination campaign for young children and the elderly moves a large number of individuals into a vaccinated state, this file could be something like

```
place, date, amount, source_infection_stage, source_vaccine_doses, source_age_group, destination_infection_stage, destination_vaccine_doses, destination_age_group
anytown, 1950-03-15, 452, S, 0dose, under5years, S, 1dose, under5years
anytown, 1950-03-16, 527, S, 0dose, 5_10years, S, 1dose, 5_10years
anytown, 1950-03-17, 1153, S, 0dose, over65years, S, 1dose, over65years
```

#### PoissonDistributed or NegativeBinomialDistributed

These methods are very similar to FromFile, except the seeding value used in the simulation is randomly drawn from the seeding value specified in the file, with an average value equal to file value. These methods can be useful when the true seeded value is unknown, and only an observed value is available which is assumed to be observed with some uncertainty. The input requirements are the same for both distributions:

```
seeding:
  method: "PoissonDistributed"
  lambda_file: seeding.csv
```

or

```
seeding:
  method: "NegativeBinomialDistributed"
  lambda_file: seeding.csv
```

and the `lambda_file` has the same format requirements as the `seeding_file` for the FromFile method described above.&#x20;

For `method::PoissonDistributed`, the seeding value for each seeding event is drawn from a Poisson distribution with mean and variance equal to the value in the `amount` column. For`method::NegativeBinomialDistributed`, seeding is drawn from a negative binomial distribution with mean `amount` and  variance `amount+5` (so identical to PoissonDistributed for large values of `amount` but has higher variance for small values).&#x20;

#### FolderDraw

TBA

## Specifying model initial conditions

The configuration items in the `initial_conditions` section of the config file are

`initial_conditions:method` Must be either "`Default"`, `"SetInitialConditions"`, "`FromFile"`, or "`FolderDraw".`

`initial_conditions:initial_conditions_file` Only required for `method: “SetInitialConditions”` and `method: “FromFile”.` Path to a .csv or .parquet file containing the list of initial conditions for each compartment.&#x20;

`initial_conditions::allow_missing_nodes` Optional for all methods, determines what will happen if `initial_conditions_file` is missing values for some subpopulations.  If FALSE, the default behavior, or unspecified, an error will occur if subpopulations are missing. If TRUE, then for subpopulations missing from the `initial_conditions` file, it will be assumed that all individuals begin in the first compartment (The “first” compartment depends on how the model was specified, and will be the compartment that contains the first named category in each compartment group.)

`initial_conditions::allow_missing_compartments`  Only applies to methods "`FromFile"` or "`FolderDraw"`. If FALSE, the default behavior, or unspecified, an error will occur if any compartments are missing for any subpopulation. If TRUE, then it will be assumed there are zero individuals in compartments missing from the `initial_conditions file`.

Details on implementing each initial conditions method and the options that go along with it are below

### initial\_conditions::method

#### Default

The default initial conditions are that initial value of all compartments for each subpopulation will be zero, except for the first compartment, whose value will be the population size. The “first” compartment depends on how the model was specified, and will the the compartment that contains the first named category in each compartment group.&#x20;

For example, for the following configuration file&#x20;

```
TBA
```

with the accompanying geodata file

```
// Some code
```

The model will be started with 1000 individuals in the S\_\[TBA] compartment.&#x20;

#### SetInitialCondition

With this method users can specify arbitrary initial conditions in a convenient formatted input csv file.

For example

```
// Some code
initial_conditions:
    method: SetInitialCondition
    allow_missing_nodes: TRUE
```

where initial\_conditions.csv contains

`initial_conditions::initial_conditions_file` must contain the following columns:

* `comp` - the concatenated name of the compartment for which an initial condition is being specified. The order of the compartment groups in the name must be the same as the order in which these groups are defined in the config for the model, e.g. you cannot say unvaccinated\_S
* `place` -  the name of the subpopulation (geoid) for which the initial condition is being specified. By default, all subpopulations must be listed in this file, unless the `allow_missing_nodes` option is set to TRUE.&#x20;
* `amount`  - the value of the initial condition.&#x20;

For each subpopulation, if there are compartments that are not listed in `SetInitialConditions`, it will be assumed there are zero individuals in them. Note that there is currently no mechanism to ensure that the sum of the values of the initial conditions in all compartments in a location adds up to the total population of that location.&#x20;

#### FromFile

Similar to SetInitialConditions, with this method users can specify arbitrary initial conditions in a formatted input file. However, the format of the input file is different and it can be either a .csv or .parquet file.&#x20;

For example, for the input&#x20;

```
initial_conditions:
  method: "FromFile"
  initial_conditions_file: pathtoyourfile.csv
  allow_missing_compartments: FALSE
  allow_missing_nodes: FALSE
```



\[TBA]

#### FolderDraw

