# Specifying population structure

## Overview

This section of the configuration file is where users can input the information required to define a population structure on which to simulate the model. The options allow the user to determine the population size of each subpopulation that makes up the overall population, and to specify the amount of mixing that occurs between each pair of subpopulations.&#x20;

## Items and options

ALH:This needs to be updated, meanings of some of these unclear and many likely deprecated. Add default values. All variables need to be connected to how they're described in the Model Description section

| Config Item            | Required?                                                                        | Type/Format                                                        | Description                                |
| ---------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------ | ------------------------------------------ |
| geodata                | **required**                                                                     | path to file                                                       | path to file relative to `data_path`       |
| popnodes               | **required**                                                                     | string                                                             | name of population column in `geodata`     |
| nodenames              | **required**                                                                     | string                                                             | name of location nodes column in `geodata` |
| mobility               | **required**                                                                     | path to file                                                       | path to file relative to `data_path`       |
| include\_in\_report    | optional                                                                         | boolean                                                            | name of boolean column in geodata          |
| nonUS\_mobility\_setup | required for non-US locations                                                    | path to file relative to base\_path                                | ALH: Don't think actually needed           |
| nonUS\_pop\_setup      | required for non-US locations                                                    | path to file relative to base\_path                                | ALH: Don't think actually needed           |
| geoid\_params\_file    | required for non-US locations if running age-specific hospitalization adjustment | path to file with geoid-specific relative risks of health outcomes | ALH:Deprecated?                            |

#### For US-specific population structures

For creating US-based population structures using the helper script `build_US_setup.R` which is run before the main model simulation script, the following extra parameters can be specified

| Config Item     | Required? | Type/Format            | Description                                                                                                         |
| --------------- | --------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------- |
| census\_year    | optional  | integer (year)         | Determines the year for which census population size data is pulled.                                                |
| state\_level    | optional  | boolean                | Determines whether county-level population-size data is instead grouped into state-level data (TRUE). Default FALSE |
| modeled\_states | optional  | list of location codes | A vector of locations that will be modeled; others will be ignored                                                  |

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

The `mobility` file is a .csv file (it has to contain .csv as extension) with long form comma separated values. Columns have to be named `ori`, `dest`, `amount,` with amount being the average number individuals moving from place `ori` to place `dest` on any given day. Unassigned relations are assumed to be zero. The location entries in the `ori` and `dest` columns should match exactly the `nodenames` column in `geodata.csv`

#### Example mobility file format

```
ori, dest, amount
10001, 20002, 3
20002, 10001, 3
```

It is also possible, but NOT RECOMMENDED to specify the `mobility` file as a .txt with space-separated values in the shape of a matrix. This matrix is symmetric and of size K x K, with K being the number of rows in `geodata`. The above example corresponds to

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
  nodenames: geoid
```

`geodata.csv` contains

```
geoid,          population
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

To simulate an epidemic across all 50 states of the US or a subset of them, users can take advantage of built in machinery to create geodata and mobility files for the US based on the population size and number of daily commuting trips reported in the US Census.&#x20;

Before running the simulation, the script `build_US_setup.R` can be run to get the required population data files from online census data and filter out only states/territories of interest for the model. More details are provided in the How to Run section.&#x20;

This example simulates COVID-19 in the New England states, assuming no transmission from other states, using 2019 census data for the population sizes and a pre-created file for estimated interstate commutes during the 2011-2015 period.

```
spatial_setup:
  census_year: 2010
  state_level: TRUE
  geodata: geodata_2019_statelevel.csv
  mobility: mobility_2011-2015_statelevel.csv
  popnodes: pop2019est
  nodenames: geoid
  modeled_states:
    - CT
    - MA
    - ME
    - NH
    - RI
    - VT
  
```

`geodata.csv` contains&#x20;

```
USPS	geoid	pop2019est
AL	01000	4876250
AK	02000	737068
AZ	04000	7050299
AR	05000	2999370
CA	06000	39283497
.....
```

`mobility_2011-2015_statelevel.csv` contains

```
ori	dest	amount
01000	02000	198
01000	04000	292
01000	05000	570
01000	06000	1030
01000	08000	328
.....
```
