# Introduction to configuration files

## About configuration files

FlepiMop is set up so that all parameters and other options for running the pipeline can be specified in a single "configuration" file (aka "config"). Users do not need to edit any other code files, or even be aware of their contents, to create and run complex model scenarios. Configuration files also provide a convenient record of model options and promote reproducibility of model results. 

We use the `YAML` language syntax to write config files, which are typically named something like `config.yml`. The file is has simple plain text contents and follows a tabbed outline structure. When config files are read by the model code, a data structure encoding the model options is created. 

## Example

(give a simple configuration for a toy model with two subpopulations, SEIR, single "cases" outcome,   single seeded infection, single NPI that starts after some time?)


When refering to config items (individual parameters), we use their full position in the outline. For example, in the sample config file above, we denote
```
spatial_setup:
  ...
  geodata: minimal
```
as `spatial_setup::geodata` having a value of `minimal`


## Notation

Parameters and other options specified in the configuration files can take on a variety of types of values, using the following notations

* **dates** are specified as [year]-[month]-[day]. (e.g., 2020-01-31)
* **boolean** values are either "TRUE" or "FALSE"
* **files** names are ....
* **probability** is a float between 0 and 1
* [UPDATE NEEDED] distribution is the following config structure:

### Distributions


| Distribution | Parameters | Type/Format | Description |
|--------------|------------|-------------|-------------|
| fixed        | value|||
| uniform      | low|||
|              | high|||
| normal       | |||
| truncnorm    | |||

## Configuration files sections

### Global header

These global configuration options typically sit at the top of the configuration file.

| Item                 | Required?    | Type/Format | Description |
|----------------------|--------------|-------------|-------------|
| file_is_unedited     | **required** to not be there | Remove it! | |
| name                 | **required** | string | typically named after the region/location you are modeling |
| start_date           | **required** | date | model simulation start date |
| end_date             | **required** | date | model simulation end date |
| start_date_groundtruth           | **required** | date | model simulation start date |
| end_date_groundtruth             | **required** | date | model simulation end date |
| nsimulations         | **required** | int | number of simulations to run [ALH This needs more clarity, is this repeats, what changes etc] |
| dt                   | **required** | float | simulation time step in days [ALH clarify, integration time step] |
| dynfilter_path       | optional | path to file | path to filtering text file [ALH: deprecated??]|
| smh_round       | optional | string | |
| report_location_name | optional | string | ALH: deprecated??|

For example, to simulate ... 

```
name: Hawaii 
start_date: 2020-01-31 
end_date: 2020-12-31 
nsimulations: 1000
dt: 0.25
report_location_name: Hawaii
```

### `spatial_setup` section

This section specifies the population structure on which the model will be simulated, including the names and sizes of each subpopulation and the connectivity between them. More details here: [LINK]

### `seir` section

This section is where uses can specify the structure of the infectious disease transmission model they wish to simulate (e.g., SEIR), including the disease compartments (`seir::compartments`), the allowed transitions between them (`seir::transitions`), the names of the parameters governing the transitions (`seir::parameters`), and the numerical method used to simulate the equations over time (`seir::integration`). 

The initial conditions of the model - the time and location where infection is first introduced - are specified in a separate `seeding` section (see below).

### `seeding` section

TBA

### `interventions` section

TBA

### `outcomes` section

TBA

### `filtering` section

TBA
