# File descriptions

## COVIDScenarioPipeline

[https://github.com/HopkinsIDD/COVIDScenarioPipeline](https://github.com/HopkinsIDD/COVIDScenarioPipeline)

Current branch: `main`

This repository contains all the code underlying the mathematical model and the data fitting procedure, as well as ...

To actually run the model, this repository folder must be located inside a location folder (e.g. `COVID19_USA`) which contains additional files describing the specifics of the model to be run (i.e. the config file), all the necessary data/parameters, and ... and

### **/R**

#### **/R/scripts**

* filter\_MC.R
* full\_filter.R
* build\_US\_setup.R
* build\_nonUS\_setup.R
*

#### **/R/pkgs**

*   **covidcommon**

    * config.R
    * DataUtils.R
    * file\_paths.R
    * safe\_eval.R
    * compartments.R


*   **hospitalization**

    * hospdeath.R


*   **inference**

    * groundtruth.R
    * functions.R
    * filter\_MC\_runner\_funcs.R


*   **config.writer**

    * create\_config\_data.R
    * process\_npi\_list.R
    * yaml\_utils.R


* **report.generation**
  * DataLoadFuncs.R
  * ReportBuildUtils.R
  * ReportLoadData.R
  * setup\_testing\_environment.R

### /renv

### **/gempyor\_pkg**

#### **/gempyor\_pkg/src/gempyor/**

* seir.py - Contains the core code for simulating the mathematical model. Takes in the model definition and parameters from the config, and outputs a file with a timeseries of the value of each state variable (# of individuals in each compartment)
* steps\_rk.py -&#x20;
* steps\_source.py -&#x20;
* outcomes.py - Contains the core code for generating the outcome variables. Takes in the output of the mathematical model and parameters from the config, and outputs a file with a timeseries of the value of each outcome (observed) variable
* setup.py
* compartments.py
* parameters.py
* /NPI/

#### **/gempyor\_pkg/docs**

* Rinterface.Rmd
* interface.ipynb

### /test

### /data

Depreciated? Should be removed

### /vignettes

Depreciated? Should be removed

### /doc

Depreciated? Should be removed

### /batch

### /slurm\_batch

## COVID19\_USA Repository

[https://github.com/HopkinsIDD/COVID19\_USA](https://github.com/HopkinsIDD/COVID19/\_USA)

Current branch: `main`

### **/R**

Contains R scripts for generating model input parameters from data, writing config files, or processing model output. Most of the files in here are historic (specific to a particular model run) and not frequently used. Important scripts include:

* get\_vacc\_rate\_and\_outcomes\_R13.R - this pulls vaccination coverage and variant prevalence data specific to rounds (either empirical, or specified by the scenario), and adjusts these data to the formats required for the model. Several data files are created in this process: variant proportions for each scenario, vaccination rates by age and dose. A file is also generated that defines the outcome ratios (taking in to account immune escape, cross protection and VE).

#### **/R/scripts/config\_writers**

Scripts to generate config files for particular submissions to the Scenario Modeling Hub. Most of this functionality has now been replaced by the config writer package ()

#### **R/scripts/postprocess**

Scripts to process the output of model runs into data formats and plots used for Scenario Modeling Hub and Forecast Hub. These scripts pull runs from AWS S3 buckets and processes and formats them to specifications for submissions to Scenario Modeling Hubs, Forecast Hubs and FluSight. These formatted files are saved and the results visualized. This script uses functions defined in /COVIDScenarioPipeline/R/scripts/postprocess.

* run\_sum\_processing.R

### **/data**

Contains data files used in parameterizing the model for COVID-19 in the US (such as creating the population structure, describing vaccine efficacy, describing parameter alterations due to variants, etc). Some data files are re-downloaded frequently using scripts in the pipeline (us\_data.csv) while others are more static (geodata, mobility)

Important files and folders include

* geodata.csv
* geodata\_2019\_statelevel.csv
* mobility.csv
* mobility\_territories\_2011-2015\_statelevel.csv
* outcomes\_ratios.csv
* US\_CFR\_shift\_dates\_v3.csv
* US\_hosp\_ratio\_corrections.cs
* seeding\_agestrat\_RX.csv

#### **/data/shp**

"Shape-files" (.shp) that .....

#### **/data/outcomes**

* usa-geoid-params-output\_V2.parquet

**/data/intervention\_tracking**

Data files containing the dates that different non pharmaceutical interventions (like mask mandates, stay-at-home orders, school closures) were implemented by state

#### **/data/vaccination**

Files used to create the config elements related to vaccination, such as vaccination rates by state by age and vaccine efficacy by dose

**/data/variant**

Files created in the process of downloading and analyzing data on variant proportions

### **/manuscripts**

Contains files for scientific manuscripts using results from the pipeline. Not up to date

### **/config**

Contains an archive of configuration files used for previous model runs

### **/old\_configs**

Same as above. Contains an archive of configuration files used for previous model runs

### **/scripts**

Depreciated - to be removed? - contains rarely used scripts

### **/notebook**

Depreciated - to be removed? - contains rarely used notebooks to check model input. Might be used in some unit tests?

### **/NPI**

empty?

### **/ScenarioHub**

Depreciated - to be removed?
