---
description: >-
  (This section describes the location and contents of each of the output files
  produced during a non-inference model run)
---

# Model Output

The model will output 2-6 different types of files depending on whether the configuration file contains optional sections (such [interventions](model-implementation/specifying-interventions/intervention-templates.md), [outcomes](model-implementation/specifying-observational-model/outcomes-module/outcomes-for-compartments.md), and outcomes interventions) and whether [model inference](broken-reference) is conducted. These files contain the values of the variables for both the infection and (if included) observational model at each point in time and for each subpopulation. A new file of the same type is produced for each independent simulation and each intervention scenario. Other files report the values of the initial conditions, seeding, and model parameters for each subpopulation and independent simulation (since parameters may be chosen to vary randomly between simulations). When [model inference](broken-reference) is run, there are also file types reporting the model likelihood (relative to the provided data) and files for each iteration of the inference algorithm.&#x20;

Within the `model_output` directory in the project's directory, the files will be organized into folders named for the file types: seir, spar, snpi, hpar, hnpi, seed, llik (see descriptions below). Within each file type folder, files will further be organized by the simulation name (name in config), the intervention scenario names (specified with interventions:scenarios in config), and the run\_id (the date and time of the simulation, by default). For example:

* flepimop\_sample
  * model\_output
    * seir
      * sample\_2pop
        * None
          * 2023.05.24.02/12/48.
            * 000000001.2023.05.24.02/12/48..seir.csv
    * spar
    * snpi

The name of each individual file contains (in order) the .... Describe filing naming conventions

Each file is a data table that is by default saved as a [parquet files](https://parquet.apache.org/) (a compressed representation that can be opened and manipulated with minimal memory) but can alternatively be saved as a .csv. See options for specifying output type in [Other Configuration Options.](model-implementation/other-configuration-options.md)&#x20;

The example files outputs we show were generated with the following configuration file

```
TBA
```

## SEIR (Infection model output)

Files in the `seir` folder contains the output of the infection model over time. They contain the value of every variable for each day of the simulation for every subpopulation.

For the example configuration file shown above, the `seir` file is

```
// Some code
```

The meanings of the columns are:

`mc_value_type` :  either `prevalence` or `incidence`. Variable values are reported both as a **prevalence** (number of individuals in that state measured instantaneously at the start of the day, equivalent to the meaning of the S, I, or R variable in the differential equations or their stochastic representation) and as **incidence** (total number of individuals who newly entered this state, from all other states, over the course of the 24-hour period comprising that calendar day).&#x20;

`mc_infection_state`, `mc_vaccination_status`, etc: The name of the compartment for which the value is reported, broken down into the infection stage for each state type.&#x20;

`mc_name`: The name of the compartment for which the value is reported, which is a concatenation of the compartment status in each state type.

`geoid_1`, `geoid_2`, etc: one column for each different subpopulation, containing the value of the number of individuals in the described compartment in that subpopulation at the given date

`date`:  The calendar date in the simulation.&#x20;

There will be a separate seir file output for each slot (independent simulation) and for each iteration of the simulation if [Model Inference](broken-reference) is conducted.&#x20;

## SPAR (Infection model parameter values)

## SNPI (Infection model parameter intervention values)

## HPAR (Observation model parameter values)

## HNPI (Observation model parameter intervention values)

## SEED (Model initial conditions and seeding values)

## Inference runs only

During inference runs, an additional file type, `llik`, is created, which is described in the [Inference Model Output](../model-inference/inference-model-output.md) section









