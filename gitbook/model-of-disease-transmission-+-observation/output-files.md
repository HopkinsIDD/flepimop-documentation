---
description: >-
  (This section describes the location and contents of each of the output files
  produced during a non-inference model run)
---

# Model Output

The model will output 2-6 different types of files depending on whether the configuration file contains optional sections (such [interventions](model-implementation/intervention-templates.md), [outcomes](model-implementation/outcomes-for-compartments.md), and outcomes interventions) and whether [model inference](broken-reference) is conducted. These files contain the values of the variables for both the infection and (if included) observational model at each point in time and for each subpopulation. A new file of the same type is produced for each independent simulation and each intervention scenario. Other files report the values of the initial conditions, seeding, and model parameters for each subpopulation and independent simulation (since parameters may be chosen to vary randomly between simulations). When [model inference](broken-reference) is run, there are also file types reporting the model likelihood (relative to the provided data) and files for each iteration of the inference algorithm.&#x20;

Within the `model_output` directory in the project's directory, the files will be organized into folders named for the file types: seir, spar, snpi, hpar, hnpi, seed, llik, init (see descriptions below). Within each file type folder, files will further be organized by the simulation name (name in config), the intervention scenario names (specified with interventions:scenarios in config), and the run\_id (the date and time of the simulation, by default). For example:

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

Files in the `seir` folder contain the output of the infection model over time. They contain the value of every variable for each day of the simulation for every subpopulation.

For the example configuration file shown above, the `seir` file is

```
// Some code
```

The meanings of the columns are:

`mc_value_type` :  either `prevalence` or `incidence`. Variable values are reported both as a **prevalence** (number of individuals in that state measured instantaneously at the start of the day, equivalent to the meaning of the S, I, or R variable in the differential equations or their stochastic representation) and as **incidence** (total number of individuals who newly entered this state, from all other states, over the course of the 24-hour period comprising that calendar day).&#x20;

`mc_infection_state`, `mc_vaccination_status`, etc: The name of the compartment for which the value is reported, broken down into the infection stage for each state type.&#x20;

`mc_name`: The name of the compartment for which the value is reported, which is a concatenation of the compartment status in each state type.

`nodename_1`, `nodename_2`, etc: one column for each different subpopulation, containing the value of the number of individuals in the described compartment in that subpopulation at the given date. Note that these are named after the nodenames defined by the user in the geodata file.

`date`:  The calendar date in the simulation, in YYYY-MM-DD format

There will be a separate seir file output for each slot (independent simulation) and for each iteration of the simulation if [Model Inference](broken-reference) is conducted.&#x20;

## SPAR (Infection model parameter values)

The files in the `spar` folder contain the parameters that define the transitions in the seir compartmental structure.&#x20;

The `value` column gives the numerical values of the parameters defined in the corresponding column `parameter`.&#x20;

## SNPI (Infection model parameter intervention values)

Files in the `snpi` folder contain the parameter intervention values for each subpopulation.&#x20;

The columns are

`nodename`: The subpopulation to which this intervention parameter applies.

`npi_name`: The name of the intervention parameter.

`start_date`: The start date of this intervention, as defined in the configuration file.&#x20;

`end_date`: The end date of this intervention, as defined in the configuration file.&#x20;

`parameter`: The parameter to which the intervention applies, as defined in the configuration file.

`reduction`: The size of the reduction to the parameter either from the config, or fit by inference if that is run.

There will be a separate snpi file output for each slot (independent simulation) and for each iteration of the simulation if [Model Inference](broken-reference) is conducted.&#x20;

## HPAR (Observation model parameter values)

Files in the `hpar` folder contain the output parameters of the observational model. They contain the value of every parameter that applies to the outcomes model for every subpopulation.&#x20;

The meanings of the columns are&#x20;

`nodenames`: Values in this column are the nodenames as defined in the geodata file given by the user.

`quantity`: The values in this column are the types of parameter values described in the config. The options are `probability`, `delay`, and `duration`. These are the quantities to which there is some parameter defined in the config.

`outcome`: The values here are the outcomes to which this parameter applies. These are names of the outcome compartments defined in the model.

value: The values in this column are the parameter values of the quantity that apply to the given nodename and outcome.&#x20;

## HNPI (Observation model parameter intervention values)

Files in the `hnpi` folder contain any parameter intervention values that apply to the outcomes model. These parameters adjust any outcome parameters.&#x20;

`nodename`: The values of this column are the names of the nodes from the geodata file.

`npi_name`: The names/labels of the intervention parameters, defined by the user in the config file, which applies to the given node and time period.

`start_date`: The start date of the intervention in the given row.&#x20;

`end_date`: The end date of the intervention in the given row.&#x20;

`parameter`: The outcome parameter to which the intervention applies.

`reduction`: The values in this column are the reduction values of the intervention parameters, which apply to the given parameter in a given geoid. Note that these are strictly reductions **(see description on other page?)**; thus a negative value corresponds to an increase in the parameter, while a positive value corresponds to a decrease in the parameter.&#x20;

## SEED (Model initial conditions and seeding values)

## Inference runs only

During inference runs, an additional file type, `llik`, is created, which is described in the [Inference Model Output](../model-inference/inference-model-output.md) section









