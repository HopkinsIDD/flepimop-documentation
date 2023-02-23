# Specifying population structure

## Overview

## Items and options

ALH:This needs to be updated, meanings of some of these unclear and many likely deprecated. Add default values. All variables need to be connected to how they're described in the Model Description section

| Config Item            | Required?                                                                        | Type/Format                                                        | Description                                |
| ---------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------ |
| geodata                | **required**                                                                     | path to file                                                       | path to file relative to `data_path`       |
| popnodes               | **required**                                                                     | string                                                             | name of population column in `geodata`     |
| nodenames              | **required**                                                                     | string                                                             | name of location nodes column in `geodata` |
| mobility               | **required**                                                                     | path to file                                                       | path to file relative to `data_path`       |
| census\_year           | optional                                                                         | integer (year)                                                     |                                            |
| state\_level           | optional                                                                         | ?                                                                  |                                            |
| modeled\_states        | optional                                                                         | list of location codes                                             | vector of locations that will be modeled   |
| include\_in\_report    | optional                                                                         | boolean                                                            | name of boolean column in geodata          |
| shapefile\_name        | optional                                                                         | path to file relative to data\_path                                |                                            |
| shapefile              | optional                                                                         | path to file relative to data\_path; identical to shapefile\_name  |                                            |
| nonUS\_mobility\_setup | required for non-US locations                                                    | path to file relative to base\_path                                |                                            |
| nonUS\_pop\_setup      | required for non-US locations                                                    | path to file relative to base\_path                                |                                            |
| geoid\_params\_file    | required for non-US locations if running age-specific hospitalization adjustment | path to file with geoid-specific relative risks of health outcomes | ALH:Deprecated?                            |

```
spatial_setup:
  base_path: data/HI
  setup_name: HI
  geodata: geodata.csv
  mobility: mobility.txt
  popnodes: population
  nodenames: geoid
  include_in_report: include_in_report
  modeled_states:
    - HI
  census_year: 2010
  shapefile: shp/counties_2010_HI.shp
  shapefile_name: shp/counties_2010_HI.shp
```

```
spatial_setup:
  setup_name: disney
  geodata: geodata.csv
  mobility: mobility.txt
  popnodes: population
  nodenames: geoid
  modeled_states:
    - magickingdom
    - epcot
```

### `geodata` file

* `geodata` is a .csv with column headers, with at least two columns: `nodenames` and `popnodes`.
* `nodenames` is the name of a column in `geodata` that specifies unique geographical identification strings for each subpopulation.&#x20;
* `popnodes` is the name of a column in `geodata` that specifies the population size (number of individual residents) of the subpopulation given in the`nodenames` column.
* `include_in_report` is the name of an optional column in `geodata` that specifies which `nodenames` are included in the report. Models may include more locations than simply the location of interest.

#### Example geodata file format

```
geoid,population,include_in_report
10001,1000,TRUE
20002,2000,FALSE
```

### `mobility` file

The `mobility` file is a .csv file (it has to contain .csv as extension) with long form comma separated values. Columns have to be named `ori`, `dest`, `amount,` with amount being the average number individuals moving from place `ori` to place `dest` on any given day. Unassigned relations are assumed to be zero. `ori` and `dest` should match exactly the `nodenames` column in `geodata.csv`

#### Example mobility file format

```
ori, dest, amount
10001, 20002, 3
20002, 10001, 3
```

It is also possible, but NOT RECOMMENDED to specify the `mobility` file as a .txt with space-separated values in the shape of a matrix. This matrix is symmetric and of size K x K, with K being the number of rows in `geodata`:

```
0 3
3 0
```

### `shapefile` file

ALH: What is this??

## Examples
