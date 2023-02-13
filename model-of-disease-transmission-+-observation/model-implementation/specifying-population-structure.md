# Specifying population structure

## Overview

## Items and options

ALH:This needs to be updated, meanings of some of these unclear and many likely deprecated. Add default values. All variables need to be connected to how they're described in the Model Description section

| Config Item       | Required?    | Type/Format | Description |
|-------------------|--------------|-------------|-------------|
| base_path         | **required** | path to folder | base path for spatial files |
| setup_name        | **required** | string | spatial folder name |
| geodata           | **required** | path to file relative to `base_path` | |
| mobility          | **required** | path to file relative to base_path | |
| popnodes          | **required** | string | name of population column in geodata |
| nodenames         | **required** | string | name of location nodes column in geodata |
| census_year       | optional     | integer (year) | | 
| state_level      | optional     | ?| | 
| modeled_states    | optional     | list of location codes | vector of locations that will be modeled |
| include_in_report | optional     | boolean | name of boolean column in geodata |
| shapefile_name    | optional     | path to file relative to base_path | |
| shapefile         | optional     | path to file relative to base_path; identical to shapefile_name | |
| nonUS_mobility_setup | required for non-US locations    | path to file relative to base_path | |
| nonUS_pop_setup   | required for non-US locations | path to file relative to base_path | |
| geoid_params_file | required for non-US locations if running age-specific hospitalization adjustment | path to file with geoid-specific relative risks of health outcomes | |

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

### `geodata` file

* `geodata` is a .csv with column headers, with at least two columns: `nodenames` and `popnodes`.
* `nodenames` is the name of a column in `geodata` that specifies the geo IDs of an area. This column must be unique.
* `popnodes` is the name of a column in `geodata` that specifies the population of the `nodenames` column.
* `include_in_report` is the name of an optional column in `geodata` that specifies which `nodenames` are included in the report. Models may include more locations than simply the location of interest.

#### Example geodata file format

```
geoid,population,include_in_report
10001,1000,TRUE
20002,2000,FALSE
```

### `mobility` file

The `mobility` file is a .csv file (it has to contains .csv as extension) with long form comma separated values. Columns have to be named `ori, dest, amount` with amount being the amount of individual going from place `ori` to place `dest`. Unassigned relations are assumed to be zero. `ori` and `dest` should match exactly the `nodenames` column in `geodata.csv`

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
