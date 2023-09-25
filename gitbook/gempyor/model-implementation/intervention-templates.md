---
description: >-
  This section describes how to specify the interventions in flepiMoP to modify 
  parameters.
---

# Specifying interventions

**Interventions** are a powerful feature in _flepiMoP_ to enable users to modify any of the parameters being specified in the model. They can be used to mirror public health control interventions, like non-pharmaceutical interventions (NPIs), or can be used to modify any of the transmission model parameters or observation model parameters, enabling user-specified modifiers or inference-driven fitting.

In the `interventions` section of the configuration file the user can specify several possible types of interventions which will then be implemented in the model. Each intervention modifies a parameter during one or multiple time periods and for one or multiple specified subpopulations.

We currently support the following intervention types. Each of these is described in detail below:

* `"SinglePeriodModifier"` – Modifies a parameter during a single time period
* `"MultiPeriodModifier"` – Modifies a parameter by the same amount during a multiple time periods
* `"ModifierModifier"` – Modifies another intervention during a single time period
* `"StackedModifier"` – Combines two or more interventions multiplicatively

Within _flepiMoP_, interventions are run as "scenarios". With scenarios, we can use the same configuration file to run multiple versions of the model where only the interventions applied differs.

The interventions section contains two sections: `interventions::scenarios`, which lists the name of the intervention that will run in each separate scenario, and `interventions::settings`, where the details of each intervention are specified (e.g.,  the parameter it acts on, the time it is active, and the subpopulation it is applied to). An example is outlined below

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

&#x20;The major benefit of specifying both "scenarios" and "interventions" is that the user can use the `"StackedModifier"` interventions to combine other interventions in different ways, and then run either the individual or combined interventions as scenarios. This way, each scenario may consist of one or more individual interventions, and each intervention may be part of multiple scenarios. This provides a shorthand to quickly consider multiple different versions of a model that have different combinations of interventions occurring. For example, during an outbreak we could evaluate the impact of school closures, case isolation, and masking, or any one or two of these three measures. An example of a configuration file combining interventions to create new scenarios is given below

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

\[Give a configuration file that tries to use all the possible option available. Based on simple SIR model with parameters `beta` and `gamma` in 2 subpopulations. Maybe a SinglePeriodModifier on `beta` for a lockdown and `gamma` for isolation, one having a fixed value and one from a distribution, MultiPeriodModifier for school year in different places, ModifierModifer for ..., StackedModifier for .... ]

## **`interventions::scenarios`**

A list consisting of a subset of the interventions that are described in `interventions::settings`, each of which will be run as a separate scenario. For example

```
interventions:
  scenarios:
    -SchoolClosures
    -AllNPIs
```

## **`interventions::settings`**

A formatted list consisting of the description of each intervention, including it's name, the parameter it acts on, the duration and amount of the change to that parameter, and the subset of subpopulations in which the intervention takes place. The list items are summarized in the table below and detailed in the sections below.

<table><thead><tr><th>Config item</th><th width="164">Required</th><th>Type/format</th><th>Description</th></tr></thead><tbody><tr><td><code>template</code></td><td>required</td><td>string</td><td>one of <code>SinglePeriodModifier</code>, <code>MultiPeriodModifier</code>, <code>ModifierModifier</code>, or <code>StackedModifier</code></td></tr><tr><td><code>parameter</code></td><td>required</td><td>string</td><td>The parameter on which the intervention is acting. Must be a parameter defined in <code>seir::parameters</code></td></tr><tr><td><code>period_start_date</code> or <code>periods::start_date</code></td><td>required</td><td>numeric, YYYY-MM-DD</td><td>The date when the intervention starts. Notation depends on value of <code>template.</code></td></tr><tr><td><code>period_end_date</code> or <code>periods::end_date</code></td><td>required</td><td>numeric, YYYY-MM-DD</td><td>The date when the intervention ends. Notation depends on value of <code>template.</code></td></tr><tr><td><code>subpop</code></td><td>required</td><td>String, or list of strings</td><td>The subpopulations to which the interventions will be applied, or <code>"all"</code> . Subpopulations must appear in the <code>geodata</code> file. </td></tr><tr><td><code>value</code></td><td>required</td><td>Distribution, or single value</td><td>The relative amount by which an intervention <em>reduces</em> the value of a parameter. </td></tr><tr><td><code>subpop_groups</code></td><td>optional</td><td>string or a list of lists of strings</td><td>A list of lists defining groupings of subpopulations, which defines how intervention values should be shared between them, or <code>'all'</code> in which case all subpopulations are put into one group with identical intervention values. By default, if parameters are chosen randomly from a distribution or fit based on data, they can have unique values in each subpopulation. </td></tr><tr><td><code>baseline_scenario</code></td><td>Used only for <code>ModifierModifier</code></td><td>String</td><td>Name of the intervention which will be modified by the intervention modifier</td></tr><tr><td><code>scenario</code>(s) (NOTE: RENAME)</td><td>Used only for <code>StackedModifier</code></td><td>List of strings</td><td>List of intervention names to be grouped  into the new intervention name</td></tr></tbody></table>

### SinglePeriodModifier

`SinglePeriodModifier` interventions enable the user to specify an intervention that multiplicatively reduces the `parameter` of interest. It take a `parameter`, and reduces it's value by `value` (new = (1-`value`) \* old) for the subpopulations listed in`subpop` during the time interval \[`period_start_date`, `period_end_date`]

For example, if you would like to create an intervention called `lockdown` that reduces transmission by 70% in the state of California and the District of Columbia between two dates, you could specify this with a SinglePeriodModifier, as in the example below

#### Example

```
intervention:
  settings:
    lockdown: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-03-15
      period_end_date: 2020-05-01
      subpop: ['06000', '11000']
      value: 0.7
```

#### Configuration options

`template: SinglePeriodModifier`

`parameter`: The name of the parameter that will be modified. This could be a parameter defined for the transmission model in [`seir::parameters`](compartmental-model-structure.md) or for the observational model in [`outcomes`](outcomes-for-compartments.md). If the parameter is used in multiple transitions in the model then all those transitions will be modified by this amount.&#x20;

`period_start_date`: The date when the intervention starts, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter after (inclusive of) this date.

`period_end_date`: The date when the intervention ends, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter before (inclusive of) this date.

`subpop:`A list of subpopulation names/ids in which the specified intervention will be applied. This can be a single `subpop`, a list, or the word `"all"` (specifying the interventions applies to all existing subpopulations in the model). The intervention will do nothing for any subpopulations not listed here.

`value:`The fractional reduction of the parameter during the time period the intervention is active.  This can be a scalar number, or a distribution using the notation described in the [Distributions](introduction-to-configuration-files.md#distributions) section. The new parameter value will be

```
new_parameter_value = old_parameter_value * (1 - value)
```

`subpop_groups:` An optional list of lists specifying which subsets of subpopulations in subpop should share parameter values; when parameters are drawn from a distribution or fit to data. See [`subpop_groups`](intervention-templates.md#interventions-settings-groups) section below for more details.&#x20;

### MultiPeriodModifier

`MultiPeriodModifier` interventions enable the user to specify an intervention that multiplicatively reduces the `parameter` of interest by `value` (new = (1-`value`) \* old) for the subpopulations listed in `subpop` during multiple different time intervals each defined by a `start_date` and `end_date.`

For example, if you would like to describe the impact that transmission in schools has on overall disease spread, you could create an intervention that increases transmission by 30% during the dates that K-12 schools are in session in different regions (e.g., Massachusetts and Florida):

#### Example

```
school_year:
  template: MultiPeriodModifier
  parameter: beta
  groups:
    - subpop: ["25000"] 
      periods:
        - start_date: 2021-09-09
          end_date: 2021-12-23
        - start_date: 2022-01-04
          end_date: 2022-06-22
    - subpop: ["12000"]
      periods:
        - start_date: 2021-08-10
          end_date: 2021-12-17
        - start_date: 2022-01-04
          end_date: 2022-05-27
  value: -0.3
```

#### Configuration options

`template: MultiPeriodModifier`

`parameter`: The name of the parameter that will be modified. This could be a parameter defined for the transmission model in [`seir::parameters`](compartmental-model-structure.md) or for the observational model in [`outcomes`](outcomes-for-compartments.md). If the parameter is used in multiple transitions in the model then all those transitions will be modified by this amount.&#x20;

`groups:` A list of subpopulations (`subpops`) or groups of them, and time periods the intervention will be active in each of them

* `groups:subpop` A list of subpopulation names/ids in which the specified intervention will be applied. This can be a single `subpop`, a list, or the word `"all" (`specifying the interventions applies to all existing subpopulations in the model). The intervention will do nothing for any subpopulations not listed here.
* `groups: periods` A list of time periods, each defined by a start and end date, when the intervention will be applied
  * `groups:periods:start_date` The date when the intervention starts, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter after (inclusive of) this date.
  * `groups:periods:end_date` The date when the intervention ends, in YYYY-MM-DD format. The intervention will only reduce the value of the parameter before (inclusive of) this date.

`value:`The fractional reduction of the parameter during the time period the intervention is active.  This can be a scalar number, or a distribution using the notation described in the [Distributions](introduction-to-configuration-files.md#distributions) section. The new parameter value will be

```
new_parameter_value = old_parameter_value * (1 - value)
```

`subpop_groups:` An optional list of lists specifying which subsets of subpopulations in subpop should share parameter values; when parameters are drawn from a distribution or fit to data. See [`subpop_groups`](intervention-templates.md#interventions-settings-groups) section below for more details.&#x20;

### ModifierModifier

`ModifierModifier` interventions allow the user to specify an intervention that acts to modify the value of _another intervention,_ as opposed to modifying a baseline parameter value.  The intervention multiplicatively reduces the `intervention` of interest by `value` (new = (1-`value`) \* old) for the subpopulations listed in `subpop` during  the time interval \[`period_start_date`, `period_end_date`].

For example, ModifierModifier could be used to describe a social distancing policy that is in effect between two dates and reduces transmission by 60% if followed by the whole population, but part way through this period, adherence to the policy drops to only 50% of in one of the subpopulations population:

#### Example

```
intervention:
  settings:
    social_distancing: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-03-15
      period_end_date: 2020-06-30
      subpop: ['all']
      value: 0.6
    fatigue: 
      template: ModifierModifier
      baseline_scenario: social_distancing
      parameter: beta
      period_start_date: 2020-05-01
      period_end_date: 2020-06-30
      subpop: ['large_province']
      value: 0.5
```

Note that this configuration is identical to the following alternative specification

```
intervention:
  settings:
    social_distancing_initial: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-03-15
      period_end_date: 2020-04-31
      subpop: ['all']
      value: 0.6
    social_distancing_fatigue_sp: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-05-01
      period_end_date: 2020-06-30
      subpop: ['small_province']
      value: 0.6
    social_distancing_fatigue_lp: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-05-01
      period_end_date: 2020-06-30
      subpop: ['large_province']
      value: 0.3
```

However, there are situations when the `ModiferModifier` notation is more convenient, especially when doing parameter fitting. &#x20;

#### Configuration options

`template: ModifierModifier`

`baseline_scenario:` The name of the intervention which will be modified by this modifier intervention

`parameter`: The name of the parameter in the `baseline_scenario` that will be modified.&#x20;

`period_start_date`: The date when the intervention modifier starts, in YYYY-MM-DD format. The intervention modifier will only reduce the value of the other intervention after (inclusive of) this date.

`period_end_date`: The date when the intervention modifier ends, in YYYY-MM-DD format. The intervention modifier will only reduce the value of the other intervention before (inclusive of) this date.

`subpop:`A list of subpopulation names/ids in which the specified intervention modifier will be applied. This can be a single `subpop`, a list, or the word `"all"` (specifying the interventions applies to all existing subpopulations in the model). The intervention will do nothing for any subpopulations not listed here.

`value:`The fractional reduction of the baseline intervention during the time period the modifier intervention is active.  This can be a scalar number, or a distribution using the notation described in the [Distributions](introduction-to-configuration-files.md#distributions) section. The new parameter value will be

```
new_intervention_value = old_intervention_value * (1 - value)
```

and so the value of the underlying parameter that was modified by the baseline intervention will be

```
new_parameter_value = original_parameter_value * (1 - baseline_intervention_value * (1 - value) )
```

`subpop_groups:` An optional list of lists specifying which subsets of subpopulations in subpop should share parameter values; when parameters are drawn from a distribution or fit to data. See [`subpop_groups`](intervention-templates.md#interventions-settings-groups) section below for more details.&#x20;

### StackedModifier

Apply a multiplicative (or additive) combination two or more interventions.

## interventions::settings::groups

`subpop_groups:` For any of the intervention types, `subpop_groups` is an optional list of lists specifying which subsets of subpopulations in `subpop` should share parameter values; when parameters are drawn from a distribution or fit to data. All other subpopulations not listed will have unique intervention values unlinked to other areas. If the value is `'all',` then all subpopulations will be assumed to have the same intervention value. When the `subpop_groups` option is not specified, all subpopulations will be assuemd to have unique values of the intervention.&#x20;

For example, for a model of disease spread in Canada where we want to specify that the (to be varied) value of an intervention should be the same in all the Atlantic provinces (Nova Scotia, Newfoundland, Prince Edward Island, and New Brunswick), the same in all the prairie provinces (Manitoba, Saskatchewan, Alberta), the same in the three territories (Nunavut, Northwest Territories, and Yukon), and yet take unique values in Ontario, Quebec, and British Columbia, we could write

```
intervention:
  settings:
    lockdown: 
      template: SinglePeriodModifier
      parameter: beta
      period_start_date: 2020-03-15
      period_end_date: 2020-05-01
      subpop: 'all'
      subpop_groups: [['NS','NB','PE','NF'],['MB','SK','AB'],['NV','NW','YK']]
      value: 
        distribution: unofirm
        low: 0.3
        high: 0.7
        
```
