---
description: >-
  (ie fixed value or from distribution. applies to compartmental model and
  observational model)
---

# Specifying model parameters

TBA : This will include some info on specifying parameters that is general to different types of parameters. For example, how to specify parameters from a distribution instead of fixed value, or how to specify them being from an external timeseries.&#x20;

ALH: This all needs to be updated, very old. Link to a page showing how to specify distributions

And, describe common parameter types that are created from dataseries, like

* Vaccination data
* Variant data

| Config Item        | Required?    | Type/Format             | Description |
|--------------------|--------------|-------------------------|-------------|
| parameters::alpha  | optional     | fraction                | Transmission dampening parameter; Default is 1.0 and reasonable values for respiratory viruses range from 0.88-0.99|
| parameters::sigma  | **required** | fraction or probability | Inverse of the incubation period in days |
| parameters::gamma  | **required** | distribution            | Inverse of the infectious period in days |
| parameters::R0s    | **required** | distribution            | Basic reproduction number |


```
seir:
  parameters:
    alpha: 0.5
    sigma: 1 / 5.2
    gamma:
      distribution: uniform
      low: 1 / 6
      high: 1 / 2.6
    R0s:
      distribution: uniform
      low: 3.5
      high: 4
```


