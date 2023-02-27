# Introduction to configuration files

## About configuration files

FlepiMop is set up so that all parameters and other options for running the pipeline can be specified in a single "configuration" file (aka "config"). Users do not need to edit any other code files, or even be aware of their contents, to create and run complex model scenarios. Configuration files also provide a convenient record of model options and promote reproducibility of model results.

We use the `YAML` language syntax to write config files, which are typically named something like `config.yml`. The file is has simple plain text contents and follows a tabbed outline structure. When config files are read by the model code, a data structure encoding the model options is created.

Comments can be added to the config file by starting with the hash key (`#`) then a space. Comments can start anywhere on a line and continue until the end, but if they run over to a new line, a new # must be used at the start of the new line.

## Example

_(give a simple configuration for a toy model with two subpopulations, SEIR, single "cases" outcome, single seeded infection, single NPI that starts after some time?)_

When referring to config items (individual parameters), we use their full position in the outline. For example, in the sample config file above, we denote

```
spatial_setup:
  ...
  geodata: minimal
```

as `spatial_setup::geodata` having a value of `minimal`

## Notation

Parameters and other options specified in the configuration files can take on a variety of types of values, using the following notations

* **dates** are specified as \[year]-\[month]-\[day]. (e.g., 2020-01-31)
* **boolean** values are either "TRUE" or "FALSE"
* **files** names are ....
* **probability** is a float between 0 and 1
* **distribution** is a probability distribution from which a random value for the parameter is drawn each time a new simulation is run(or chain, if doing inference), and is specified with the following config structure:&#x20;

### Distributions

| Distribution | Parameters | Type/Format                                | Description                                                                                                                                                                      |
| ------------ | ---------- | ------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `fixed`      | `value`    | Any real number                            | Draws all values exactly equal to `value`                                                                                                                                        |
| `uniform`    | `low`      | Any real number                            | Draws all values randomly from a uniform distribution with range `[low, high]`                                                                                                   |
|              | `high`     | Any real number greater than `low`         |                                                                                                                                                                                  |
| `poisson`    | `lam`      | Any positive real number                   | Draws all values randomly from a Poisson distribution with rate parameter  (mean) `lam` (lambda)                                                                                 |
| `binomial`   | `size`     | Any non-negative integer                   | Draws all values randomly from a binomial distribution with number of trials (n) = `size` and probability of success on each trial (p) = `prob`                                  |
| ``           | `prob`     | Any number in \[0,1]                       |                                                                                                                                                                                  |
| `lognormal`  | `meanlog`  | Any real number                            | Draws all values randomly from a lognormal distribution (natural log, base _e_) with mean on a log scale of `meanlog` and standard deviation on a log scale of `sdlog`           |
|              | `sdlog`    | Any non-negative real number               |                                                                                                                                                                                  |
| `truncnorm`  | `mean`     | Any real number                            | Draws all values randomly from a truncated normal distribution with mean `mean` and standard deviation `sd`, truncated to have a maximum value of `a` and a minimum value of `b` |
|              | `sd`       | Any non-negative real number               |                                                                                                                                                                                  |
|              | `a`        | Any real number, or `-Inf`                 |                                                                                                                                                                                  |
|              | `b`        | Any real number greater than `a`, or `Inf` |                                                                                                                                                                                  |

## Configuration files sections

### Global header

These global configuration options typically sit at the top of the configuration file.

| Item                     | Required?                                   | Type/Format  | Description                                                                                    |
| ------------------------ | ------------------------------------------- | ------------ | ---------------------------------------------------------------------------------------------- |
| file\_is\_unedited       | **required** to not be there                | Remove it!   |                                                                                                |
| name                     | **required**                                | string       | typically named after the region/location you are modeling                                     |
| setup\_name              | **required**                                | string       | ??                                                                                             |
| start\_date              | **required**                                | date         | model simulation start date                                                                    |
| end\_date                | **required**                                | date         | model simulation end date                                                                      |
| data\_path               | **required? optional?**                     | folder path  | base path to folder where ground truth data files are stored                                   |
| start\_date\_groundtruth | **required?**                               | date         | model simulation start date                                                                    |
| end\_date\_groundtruth   | **required? optional?**                     | date         | model simulation end date                                                                      |
| nsimulations             | **required**                                | int          | number of simulations to run \[ALH This needs more clarity, is this repeats, what changes etc] |
| ~~dt~~                   | **required. MOVED NOW in SEIR:Integration** | float        | simulation time step in days \[ALH clarify, integration time step]                             |
| dynfilter\_path          | optional                                    | path to file | path to filtering text file \[ALH: deprecated??]                                               |
| smh\_round               | optional                                    | string       | ALH? What is point of this?                                                                    |
| report\_location\_name   | optional                                    | string       | ALH: deprecated??                                                                              |

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

This section specifies the population structure on which the model will be simulated, including the names and sizes of each subpopulation and the connectivity between them. More details [here](specifying-compartmental-model-parameters.md).

### `compartments` section

This section is where users can specify the variables (infection states) that will be tracked in the infectious disease transmission model. More details HERE. The other details of the model are specified in the `seir` section, including transitions between these compartments (`seir::transitions`), the names of the parameters governing the transitions (`seir::parameters`), and the numerical method used to simulate the equations over time (`seir::integration`)

### `seeding` section

This section is used to specify how infections are "seeded" into the compartments, where they are then governed by the model equations. This seeding can be used to create initial conditions (ie introductions at time zero) or to represent importations from an outside population that occur at any later time. Numeric values can be added to any compartment of the model, not just those representing "infected" states (e.g., immune individuals in the "R" state could be introduced at any point). More details HERE.

### `seir` section

This section is where users can specify the details of the infectious disease transmission model they wish to simulate (e.g., SEIR). This model describes the allowed transitions (`seir::transitions`) between the compartments that were specified in the `compartments` section, the values of the parameters involved in these transitions (`seir::parameters`), and the numerical method used to simulate the equations over time (`seir::integration`). The initial conditions of the model - the time and location where infection is first introduced - are specified in a separate `seeding` section (see below). More details HERE

### `outcomes` section

This section is where users can define new variables representing the observed quantities and how they are related to the underlying state variables in the model (e.g., the fraction of infections that are detected as cases). More details [here](specifying-observational-model/outcomes-module/outcomes-for-compartments.md).&#x20;

### `interventions` section

This section is where users can specify time-varying changes to parameters governing either the infectious disease model or the observational model. More details [HERE](specifying-interventions/intervention-templates.md).&#x20;

### `filtering` section

This section is where users can specify the details of how the model is fit to data, including what data streams they will be included and which outcome variables they represent and the likelihood functions describing the probability of the data given the model. More details HERE. &#x20;
