---
description: >-
  This page describes how users specify the names, sizes, and connectivities of
  the different subpopulations comprising the total population to be modeled
---

# Specifying population structure

## Overview

The `spatial_setup` section of the configuration file is where users can input the information required to define a population structure on which to simulate the model. The options allow the user to determine the population size of each subpopulation that makes up the overall population, and to specify the amount of mixing that occurs between each pair of subpopulations.

An example configuration file with the global header and the spatial\_setup section is below:

```
name: test_simulation
data_path: data
model_output_dirname: model_output
start_date: 2020-01-01
end_date: 2020-12-31
nslots: 100

spatial_setup:
  geodata: geodata.csv
  mobility: mobility.csv
  popnodes: population
```

## Items and options

| Config Item | Required?    | Type/Format  | Description                                |
| ----------- | ------------ | ------------ | ------------------------------------------ |
| geodata     | **required** | path to file | path to file relative to `data_path`       |
| popnodes    | **required** | string       | name of population column in `geodata`     |
| nodenames   | **required** | string       | name of location nodes column in `geodata` |
| mobility    | **required** | path to file | path to file relative to `data_path`       |

### `geodata` file

* `geodata` is a .csv with column headers, with at least two columns: `nodenames` and `popnodes`.
* `nodenames` is the name of a column in `geodata` that specifies unique geographical identification strings for each subpopulation.
* `popnodes` is the name of a column in `geodata` that specifies the population size (number of individual residents) of the subpopulation given in the`nodenames` column.
* `include_in_report` is the name of an optional column in `geodata` that specifies which `nodenames` are included in the report. Models may include more locations than simply the location of interest.

#### Example geodata file format

```
subpop,population,include_in_report
10001,1000,TRUE
20002,2000,FALSE
```

### `mobility` file

The `mobility` file is a .csv file (it has to contain .csv as extension) with long form comma separated values. Columns have to be named `ori`, `dest`, `amount,` with amount being the average number individuals moving from the origin subpopulation `ori` to destination subpopulation `dest` on any given day. Details on the mathematics of this model of contact are explained in the [Model Description section](../model-description.md#mixing-between-subpopulations). Unassigned relations are assumed to be zero. The location entries in the `ori` and `dest` columns should match exactly the `nodenames` column in `geodata.csv`

#### Example mobility file format

```
ori, dest, amount
10001, 20002, 3
20002, 10001, 3
```

It is also possible, but **not recommended** to specify the `mobility` file as a .txt with space-separated values in the shape of a matrix. This matrix is symmetric and of size K x K, with K being the number of rows in `geodata`. The above example corresponds to

```
0 3
3 0
```

## Examples

#### Example 1

To simulate a simple population structure with two subpopulations, a large province with 10,000 individuals and a small province with only 1,000 individuals, where every day 100 residents of the large province travel to the small province and interact with residents there, and 50 residents of the small province visit the large province

```
spatial_setup:
  geodata: geodata.csv
  mobility: mobility.csv
  popnodes: population
```

`geodata.csv` contains the population structure (with `nodename` column `subpop` and `popnodes` column `population`)

```
subpop,          population
large_province, 10000
small_province, 1000
```

`mobility.csv` contains

```
ori,            dest,           amount
large_province, small_province, 100
small_province, large_province, 50
```

#### Example 2

(Give example with US states)
