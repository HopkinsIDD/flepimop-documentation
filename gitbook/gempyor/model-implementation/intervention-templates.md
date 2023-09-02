---
description: >-
  This section describes how to specify the interventions in flepiMoP to modify 
  parameters.
---

# Specifying interventions

**Interventions** are a powerful feature in _flepiMoP_ to enable user to modify any of the parameters being specified in the model. They can be used to mirror public health control interventions, like non-pharmaceutical interventions (NPIs), or can be used to modify any of the transmission model parameters or observation model parameters, enabling user-specified modifiers or inference-driven fitting.

In the `interventions` section of the configuration file the user can specify several possible types of interventions which will then be implemented in the model. Each intervention modifying a parameter during one or multiple time periods and for one or multiple specified subpopulations.

We currently support the following interventions types. Each of these is described in detail below:

* _SinglePeriodModifier:_ Modified a parameter during a single time period.
* _MultiPeriodModifier:_ Modifies a parameter by the same amount during a multiple time periods.
* _ModifierModifier:_ Modifies another intervention during a single time period.
* _StackedModifier:_ Combines two or more interventions multiplicatively.

Within flepiMoP, interventions are run as "scenarios". With scenarios, we can use the same configuration file to run multiple versions of the model where only the interventions applied differs.

The interventions section contains two sections: `interventions:scenarios`, which lists the name of the intervention that will run in each separate scenario, and `interventions:settings`, where the details of each intervention are specified (e.g.,  the parameter it acts on, the time it is active, and the subpopulation it is applied in). An example is outlined below

```
interventions:
  scenarios:
    -NameOfIntervention1
    -NameofIntervention2
  settings:
    NameOfIntervention1:
      ...
    NameOfIntervention2:
      ...
```

&#x20;The major benefit of specifying both "scenarios" and "interventions" is that the user can use the `StackedModifier` interventions to combine other interventions in different ways, and then run either the individual or combined interventions as scenarios. This way, each scenario may consist of one or more individual interventions, and each intervention may be part of multiple scenarios. This provides a shorthand way to quickly consider multiple different versions of a model that have different combinations of interventions occurring. For example, during an outbreak we could evaluate the impact of school closures, case isolation, and masking, or any one or two of these three measures. An example of a configuration file combining interventions to create new scenarios is given below:

```
interventions:
  scenarios:
    -SchoolClosures
    -AllNPIs
  settings:
    SchoolClosures:
      template:SinglePeriodModifier
      ...
    CaseIsolation:
      template:SinglePeriodModifier
      ...
    Masking:
      template:SinglePeriodModifier
      ....
    AllNPIs
      template: Stacked
      scenarios: ["SchoolClosures","CaseIsolation","Masking"]
```



{% hint style="info" %}
Each time a configuration file is run, the user much specify which intervention scenarios will be run. If not specified, the model will be run one time for each scenario.&#x20;
{% endhint %}

#### Example

\[Give a configuration file that tries to use all the possible option available. Based on simple SIR model with parameters beta and gamma in 2 subpopulations. maybe a SinglePeriodModifier on Beta for a lockdown and Gamma for isolation, one having a fixed value and one from a distribution, MultiPeriodModifier for school year in different places, ModifierModifer for ..., StackedModifier for .... ]

## **`interventions::scenarios`**

A list consisting of a subset of the interventions that are described in `interventions::settings`, each of which will be run as a separate scenario

## **`interventions::settings`**

....

table of options - if there are repeated options used across the different types of interventions, then make a table at the top and describe them there

(say that order of these and whether required or not might differ by intervention type?)

<table><thead><tr><th>Config item</th><th width="164">Required</th><th>Type/format</th><th>Description</th></tr></thead><tbody><tr><td>template</td><td>required</td><td>string</td><td>one of SinglePeriodModifier, MultiPeriodModifier, ModifierModifier, or StackedModifier</td></tr><tr><td>parameter</td><td>required</td><td>string</td><td>The parameter on which the intervention is acting. Must be a parameter defined in seir::parameters</td></tr><tr><td>period_start_date or periods::start_date</td><td>required</td><td>numeric, YYYY-MM-DD</td><td>The date when the intervention starts</td></tr><tr><td>period_end_date or periods::end_date</td><td>required</td><td>numeric, YYYY-MM-DD</td><td></td></tr><tr><td>subpop</td><td></td><td></td><td></td></tr><tr><td>groups</td><td></td><td></td><td></td></tr><tr><td>value</td><td>required</td><td></td><td></td></tr><tr><td>scenario(s)</td><td></td><td></td><td></td></tr></tbody></table>

### SinglePeriodModifier

_**SinglePeriodModifier**_** interventions** enable the user to specify an intervention that multiplicatively reduces the `parameter` of interest.

For example, if you would like an intervention that mirrors a lockdown intervention and reduces transmission by 50%

Take a `parameter`, and reduce it's value by `value` (new = (1-`value`) \* old) for `subpop` during \[`period_start_date`, `period_end_date`]

#### Config Example (SinglePeriodModifier) (FIX)

```
intervention:
  settings:
    lockdown: 
      template: SinglePeriodModifier
      parameter: r0
      period_start_date: 2020-01-01
      period_end_date: 2021-12-31
      subpop: ['06000', '11000']
      value:
        distribution: truncnorm
        mean: .5
        sd: 0.1
        a: 0
        b: 1
```

This intervention reduces `r0` by a random number drawn from a normal distribution with mean 0.5 and standard deviation 0.1. It only applies between to dates between January 01, 2020 and December 31, 2021 and to locations 06000 and 11000.

#### Configuration options

`template: SinglePeriodModifier`

`parameter`: The name of the parameter that will be modified. This could be a parameter defined for the transmission model in seir::parameters or ... . If the parameter is used in multiple transitions in the model then all those transitions will be modified by this amount. The parameter could also be one that occurs in the `outcomes` section defining the observational model (see next section)

`period_start_date: The` date when the intervention starts, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter after (inclusive of) this date.

`period_end_date: The` date when the intervention ends, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter before (inclusive of) this date.

`subpop:`A list of subpopulation names/ids in which the specified intervention will be applied. This can be a single `subpop`, a list, or the word `"all" (`specifying the interventions applies to all existing subpopulations in the model). The intervention will do nothing for any subpopulations not listed here.

`value:`The fractional reduction of the parameter during the time period the intervention is active. The new parameter value will be

```
new_parameter_value = old_parameter_value * (1 - value)
```

## MultiPeriodModifier

\[describe]

#### Config Example (MultiPeriodModifier) (FIX)

```
school_year_R13:
  template: MultiPeriodModifier
  parameter: r0
  groups:
    - subpop: ["01000"]
      periods:
        - start_date: 2021-08-16
          end_date: 2021-11-23
        - start_date: 2022-01-03
          end_date: 2022-03-19
    - subpop: ["02000"]
      periods:
        - start_date: 2021-08-16
          end_date: 2021-11-23
        - start_date: 2022-01-03
          end_date: 2022-03-19
  value:
    distribution: truncnorm
    mean: .5
    sd: 0
    a: 0
    b: 1
```

#### Configuration options

`template: SinglePeriodModifier`

`parameter`: The name of the parameter that will be modified. This could be a parameter defined for the transmission model in seir::parameters or ... . If the parameter is used in multiple transitions in the model then all those transitions will be modified by this amount. The parameter could also be one that occurs in the `outcomes` section defining the observational model (see next section)

`groups:` A list of subpops and times the intervention should affect. \[need to expand]&#x20;

* `groups:subpop` A list of subpopulation names/ids in which the specified intervention will be applied. This can be a single `subpop`, a list, or the word `"all" (`specifying the interventions applies to all existing subpopulations in the model). The intervention will do nothing for any subpopulations not listed here.
* `groups: periods` A list of time periods, each defined by a start and end date, when the intervention will be applied
  * `groups:periods:start_date` The date when the intervention starts, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter after (inclusive of) this date.
  * `groups:periods:end_date` The date when the intervention ends, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter before (inclusive of) this date.

`value:`The fractional reduction of the parameter during the time period the intervention is active. The new parameter value will be

```
new_parameter_value = old_parameter_value * (1 - value)
```

##

## ModifierModifier

\[DESCRIBE] Modify another intervention.

Example config section

Configuration options

## StackedModifier

Apply a multiplicative (or additive) combination two or more interventions.

#### `template`

#### `scenarios`
