---
description: >-
  This section describes how to specify the interventions in flepiMoP to modify 
  parameters.
---

# Specifying interventions

**Interventions** are a powerful feature in _flepiMoP_ to enable user to modify any of the parameters being specified in the model. They can be used to mirror public health control interventions, like non-pharmaceutical interventions (NPIs) or can be used to modify any of the transmission model parameters or observation model parameters, enabling user-specified modifiers or inference-driven fitting.&#x20;

Each intervention is specified as one of several possible types in the configuration file to tell _gempyor_ how to implement the intervention in the model. Each of these intervention types enable specified methods of modifying a parameter during one or multiple specified time periods and for one or multiple specified subpopulations.

We currently support the following interventions types. Each of these is described in detail below:

* _SinglePeriodModifier:_ Modified a parameter during a single time period.
* _MultiPeriodModifier:_ Modifies a parameter during a multiple time periods.
* _ModifierModifier:_ Modifies another intervention during a single time period.
* _StackedModifier:_ Combines two or more interventions multiplicatively.

## SinglePeriodModifier

_**SinglePeriodModifier interventions**_ enable the user to specify an intervention that multiplicatively reduces the `parameter` of interest.&#x20;

For example, if you would like an intervention that mirrors a lockdown intervention and reduces transmission by 50%

Take a `parameter`, and reduce it's value by `value` (new = (1-`value`) \* old) for `subpop` during \[`period_start_date`, `period_end_date`]

#### Config Example (SinglePeriodModifier)

```
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

### Parameters

#### `template`

The `template` option specifies the type of the intervention. In

#### `parameter`

A parameter name to reduce. Reducing a parameter will modify the parameter's value, and so any transitions or outcomes using that parameter will be affected.

#### `period_start_date`

A date when the intervention starts. The intervention will only reduce the value of the parameter after (inclusive) this date.

#### `period_end_date`

A date when the intervention ends. The intervention will only reduce the value of the parameter before (inclusive) this date.

#### `subpop`

A list of subpopulation names/ids that the specified intervention applies to. This can be a single `subpop`, a list, or the word `"all"`, specifying the interventions applies to all existing subpopulations in the model. The intervention will do nothing for any subpopulations not listed here.

#### `value`

The percentage reduction by which to reduce the parameter. The new parameter value will be

```
new_value = old_value * (1 - reduction)
```

## MultiPeriodModifier

### Config Example (MultiPeriodModifier)

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

### Parameters

#### `parameter`

A parameter name to reduce. Reducing a parameter will modify the parameter's value, and so any transitions or outcomes using that parameter will be affected.

#### `groups`

A list of subpops and times the intervention should affect.

**`subpop`**

A list of subpop names to affect, or the word 'all'. The intervention will do nothing for any subpops not listed here. If 'all', every subpop listed in the subpop file will be affected.

####

## ModifierModifier

Modify another intervention.

## StackedModifier

Apply a multiplicative (or additive) combination two or more interventions.

*
