# Intervention templates

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
