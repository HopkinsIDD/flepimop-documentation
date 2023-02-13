# Specifying interventions

## Overview

Every intervention has a template, which tells gempyor how to implement the intervention from the config. We currently support the following interventions:

* Reduce
  * Modify a parameter during a single time period
* MultiTimeReduce
  * Modify a parameter during a multiple time periods
* ReduceIntervention
  * Modify another intervention during a single time period
* Stacked
  * Combine two or more interventions multiplicatively

This section lets you  specify custom intervention scenarios.

`scenarios` specifies which settings to run. This does not need to include all items defined in `settings`.

| Config Item  | Required?    | Type/Format                      | Description |
|--------------|--------------|----------------------------------|-------------|
| scripts_path | **required** | path name                        | |
| scenarios    | **required** | list of strings for scenario names | |
| settings     | **required** | See section below                | |


### `interventions::settings::[setting_name]`

Each string in `scenarios` should have a corresponding named setting in `settings`.

Right now, there are three types of templates: Reduce, ReduceR0 and Stacked. The ReduceR0 template is a special and redundant case of the Reduce template for the the `r0` parameter. The Stacked template allows you to combine multiple interventions (Reduce or Stacked) together into a single intervention scenario.

| Item              | Required?             | Type/Format                                     | Description |
|-------------------|-----------------------|-------------------------------------------------| ------------|
| template          | **required**          | "Reduce", "ReduceR0" or "Stacked"              | |
| parameter         | required for Reduce | "alpha", "r0", "gamma", "sigma" | Specify the parameter associated with the intervention reduction (alpha = mixing coefficient, r0 = basic reproductive number, gamma = inverse of the infectious period, sigma = inverse of the incubation period |
| period_start_date | optional for Reduce, ReduceR0 | | date between global `start_date` and `end_date`; default is global `start_date` |
| period_end_date   | optional for Reduce, ReduceR0 | | date between global `start_date` and `end_date`; default is global `end_date`  |
| value             | required for Reduce, ReduceR0 | distribution | |
| affected_geoids   | optional for Reduce, ReduceR0 | | list of geoids, which must be in geodata        |
| fatigue_rate::distribution | optional for Reduce, ReduceR0 | distribution | Indicates the rate of intervention fatigue  | 
| fatigue_frequency_days | optional for Reduce, ReduceR0 | numeric | Number of days it takes for the NPI to reach the new value |
| fatigue_min | optional for Reduce, ReduceR0 | numeric | Minimum intervention effectivness for fatiguing interventions, clip values below this minimum |
| fatigue_type | optional for Reudce, Reduce R0 | "geometric" | If specified, produce a geometric fatiguing rate instead of a linear fatiguing rate |

```
interventions:
  scenarios:
    - None
    - Scenario1
    - Scenario2
  settings:
    None:
      template: ReduceR0
      period_start_date: 2020-04-01
      period_end_date: 2020-05-15
      value:
        distribution: fixed
        value: 0
    Wuhan:
      template: Reduce
      parameter: r0
      period_start_date: 2020-04-01
      period_end_date: 2020-05-15
      value:
        distribution: uniform
        low: .81
        high: .89
    UK:
      template: ReduceR0
      period_start_date: 2020-05-16
      period_end_date: 2020-05-31
      value:
        distribution: uniform
        low: .71
        high: .83
    Scenario2:
      template: Reduce      
      parameter: r0                       # Parameter to reduce
      period_start_date: 2020-02-01
      period_end_date: 2020-05-15
      value:
        distribution: uniform            # Value of reduction as it was specified before.
        low: .6
        high: .7
      fatigue_rate:
        distribution: uniform            # Value of fatigue
        low: .1
        high: .2
      fatigue_frequency_days: 4*7         # Number of days for the NPI to reach a new_value
      fatigue_min: .2                  # 0 if unspecified, clip when the NPI reaches this value.
      #fatigue_type: geometric     # if there, produce geometric fatigue (so reduction of reduction)
    Scenario1:
      template: Stacked
      scenarios:
        - Wuhan
        - UK
```


## Reduce

Take a `parameter`, and reduce it's value by `value` (new = (1-`value`) \* old) for `affected_geoids` during \[`period_start_date`, `period_end_date`]

### Config Example (Reduce)

```
local_variance:
  template: Reduce
  parameter: r0
  period_start_date: 2020-01-01
  period_end_date: 2021-12-31
  affected_geoids: ['06000', '11000']
  value:
    distribution: truncnorm
    mean: .5
    sd: 0.1
    a: 0
    b: 1
```

This intervention reduces `r0` by a random number drawn from a normal distribution with mean 0.5 and standard deviation 0.1. It only applies between to dates between January 01, 2020 and December 31, 2021 and to locations 06000 and 11000.

### Parameters

#### `parameter`

A parameter name to reduce. Reducing a parameter will modify the parameter's value, and so any transitions or outcomes using that parameter will be affected.

#### `period_start_date`

A date when the intervention starts. The intervention will only reduce the value of the parameter after (inclusive) this date.

#### `period_end_date`

A date when the intervention ends. The intervention will only reduce the value of the parameter before (inclusive) this date.

#### `affected_geoids`

A list of geoid names to affect, or the word 'all'. The intervention will do nothing for any geoids not listed here. If 'all', every geoid listed in the geoid file will be affected.

#### `value`

The percentage reduction by which to reduce the parameter. The new parameter value will be

```
new_value = old_value * (1 - reduction)
```

## MultiTimeReduce

### Config Example (MultiTimeReduce)

```
school_year_R13:
  template: MultiTimeReduce
  parameter: r0
  groups:
    - affected_geoids: ["01000"]
      periods:
        - start_date: 2021-08-16
          end_date: 2021-11-23
        - start_date: 2022-01-03
          end_date: 2022-03-19
    - affected_geoids: ["02000"]
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

### Parameters

#### `parameter`

A parameter name to reduce. Reducing a parameter will modify the parameter's value, and so any transitions or outcomes using that parameter will be affected.

#### `groups`

A list of places and times the intervention should affect.

**`affected_geoids`**

A list of geoid names to affect, or the word 'all'. The intervention will do nothing for any geoids not listed here. If 'all', every geoid listed in the geoid file will be affected.

####

## ReduceIntervention

Modify another intervention.

## Stacked

Apply a multiplicative (or additive) combination two or more interventions.

*
