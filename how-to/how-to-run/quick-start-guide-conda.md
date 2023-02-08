---
description: Short internal tutorial on running locally using an "Anaconda" environment.
---

# Running with conda locally

### Setup (do this once)

#### Installing the `conda` environment

The simplest way to get everything to work is to build an Anaconda environment. Install (or update) Anaconda on your computer. You can either use the command line (here) or the graphical user interface (you just tick the packages you want). With the command line it's this one-liner:

{% code overflow="wrap" %}
```bash
conda update conda # makes sure you have a recent conda instatllation

# be sure to copy the whole thing as a single line ! copy it to your text editor
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

Anaconda will take some time, to come up with a proposal that works will all dependencies.&#x20;

This creates a `conda` environment named `covidSP` that has all the necessary packages. &#x20;

If you'd like you can install `rstudio` as a package as well, but I think your Rstudio should be able to find your conda R without problems.

<details>

<summary>Can I run without conda?</summary>

Anaconda is the most reproducible way to run our model. However, you can still proceed without it. You can just carry on with the steps below without creating an environment.

**How to do it?** Just skip every line starting with `conda` and do not use the `--no-deps` flag when installing gempyor (so pip will install the dependencies). When running `local_install.R` there may be failures because some packages are missing. Install them as you usually do from R. The rest is the same as this tutorial.

</details>

### How to run inference

#### Fill the environment variables (do this every time)

First, you'll need to fill in some variables that are used by the model. This can be done in a script (an example is provided at the end of this page). For your first time, it's better to run each command individually to be sure it exits successfully.&#x20;

First, in `myparentfolder` populate the folder name variables:

```bash
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
export DATA_PATH=$(pwd)/Flu_USA
```

Note: you can replace `Flu_USA` with any relevant data folder you wish, e.g., `COVID19_USA`.

Then, export variables for some flags and the census API key (you can use your own):

{% code overflow="wrap" %}
```bash
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
export CENSUS_API_KEY="6a98b751a5a7a6fc365d14fa8e825d5785138935"
```
{% endcode %}

#### Install the packages (do this every time the COVIDScenarioPipeline repository has changed)

Activate your conda environment, if you have one.

```bash
conda activate covidSP
```

In this conda environment, commands with R and python will uses this environment's R and python. Go into the Pipeline repo (making sure it is up to date on your favorite branch) and do the installation required of the repository:

```bash
cd $COVID_PATH   # it'll move to the COVIDScenarioPipeline/ directory
Rscript local_install.R               # Install the R stuff, will fail
Rscript local_install.R               # Install the R stuff, will sucedd
pip install --no-deps -e gempyor_pkg/ # install gempyor
git lfs install
git lfs pull
```

{% hint style="danger" %}
When running `local_install.R` the first time, you will get an error:&#x20;

<pre><code><strong>ERROR: dependency ‘report.generation’ is not available for package ‘inference’
</strong><strong>[...]
</strong><strong>installation of package ‘./R/pkgs//inference’ had non-zero exit status
</strong></code></pre>

and the second time it'll finish successfully (no non-zero exit status at the end). That's because there is a circular dependency in this file (inference requires report.generation which is built after) and will hopefully get fixed.&#x20;

For subsequent runs, once is enough because the package is already installed once.
{% endhint %}

<details>

<summary>Help! I still have errors</summary>

If you get an error because no cran mirror is selected, just create in your home directory a `.Rprofile`file:

{% code title="~/.Rprofile" lineNumbers="true" %}
```r
local({r <- getOption("repos")
       r["CRAN"] <- "http://cran.r-project.org" 
       options(repos=r)
})
```
{% endcode %}

Perhaps this should be added to the top of the local\_install.R script #todo

</details>

### Run the code

Everything is now ready. 🎉 Let's do some clean-up in the data folder (these files might not exist, but it's good practice to make sure your simulation isn't re-using some old files).

```bash
cd $DATA_PATH       # goes to Flu_USA
git restore data/
rm -rf data/mobility_territories.csv data/geodata_territories.csv data/us_data.csv
rm -r model_output/ # delete the outputs of past run if there are
```

Stay in `$DATA_PATH`, select a config, activate the conda enviroment (if not already done) and build the setup. Then, run inference:

```bash
conda activate covidSP
export CONFIG_PATH=config_SMH_R1_lowVac_optImm_2022.yml
Rscript $COVID_PATH/R/scripts/build_US_setup.R
Rscript $COVID_PATH/R/scripts/full_filter.R -j 1 -n 1 -k 3
```

where:

* `n` is the number of parallel inference slots,
* `j` is the number of CPU cores it'll use in your machine,
* `k` is the number of iterations per slots.

It should run successfully and create a lot of files in `model_output/`.&#x20;

Okay, I'll make a part 2 on how to plot and debug what happened. You can also try to knit the Rmd file in `COVIDScenarioPipeline/gempyor_pkg/docs` which will show you how to analyze these files.

### Do it all with a script

The following script does all the above commands in an easy script. Save it in `myparentfolder` as `quick_setup_flu.sh`. Then, just go to `myparentfolder` and type `source quick_setup_flu.sh` and it'll do everything for you!

<pre class="language-bash" data-title="quick_setup_flu.sh" data-line-numbers><code class="lang-bash">export COVID_PATH=$(pwd)/COVIDScenarioPipeline
export DATA_PATH=$(pwd)/Flu_USA
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
cd $COVID_PATH
Rscript local_install.R
pip install --no-deps -e gempyor_pkg/ # before: python setup.py develop --no-deps
git lfs install
git lfs pull
export CENSUS_API_KEY="6a98b751a5a7a6fc365d14fa8e825d5785138935"
cd $DATA_PATH
<strong>git restore data/
</strong>rm -rf data/mobility_territories.csv data/geodata_territories.csv data/us_data.csv
export CONFIG_PATH=config_SMH_R1_lowVac_optImm_2022.yml
Rscript $COVID_PATH/R/scripts/build_US_setup.R
echo "might want to rm -R model_output"
echo "now you can type: Rscript \$COVID_PATH/R/scripts/full_filter.R -j 1 -n 1 -k 1"
</code></pre>



