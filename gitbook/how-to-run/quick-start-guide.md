---
description: >-
  Instructions to get started with using gempyor and flepiMoP, using some
  provided example configs to help you.
---

# Quick Start Guide

{% hint style="danger" %}
DEV NOTE: Currently flepimop\_sample configs only work on the breaking-improvments branch of flepimop\_sample and gemyor-simulate is only available on the breaking-improvments branch of flepiMoP
{% endhint %}

##  📂 Access model files

As is the case for any run, first see the [Before any run](before-any-run.md) section to ensure you have access to the correct files needed to run.&#x20;

To get started with flepiMoP, we recommend forking and cloning _flepimop\_sample_ and using the provided example configuration files. To run any model configurations, copy the appropriate configs to the main directory to run them.&#x20;

On your local machine, determine the file paths to:

* the directory containing the flepimop code (likely the folder you cloned from Github), which we'll call `FLEPI_PATH`
* the directory containing your project code including input configuration file and population structure (again likely from Github), which we'll call `DATA_PATH`

{% hint style="info" %}
For example, if you clone your Github repositories into a local folder called Github and are using [_flepimop\_sample_](https://github.com/HopkinsIDD/flepimop\_sample) as a project repository, your directory names could be\
\
_**On Mac:**_&#x20;

\<dir1> = /Users/YourName/Github/flepiMoP

\<dir2> = /Users/YourName/Github/flepimop\_sample\
\
_**On Windows:**_ \
\<dir1> = C:\Users\YourName\Github\flepiMoP

\<dir2> = C:\Users\YourName\Github\flepimop\_sample\


(hint: if you navigate to a directory like `C:\Users\YourName\Github` using `cd C:\Users\YourName\Github`, modify the above `<dir1>` paths to be `.\flepiMoP` and `.\flepimop_sample)`



:warning: Note again that these are best cloned **flat.**
{% endhint %}

## 🧱 Set up&#x20;

### Before running

The code is written in a combination of [R](https://www.r-project.org/) and [Python](https://www.python.org/). The Python part of the model is a package called [_gempyor_](../gempyor/model-description.md), and includes all the code to simulate the epidemic model and the observational model and apply time-dependent interventions. The R component conducts the (optional) parameter inference, and all the (optional) provided pre and post processing scripts are also written in R. Most uses of the code require interacting with components written in both languages, and thus making sure that both are installed along with a set of required packages. However, Python alone can be used to do forward simulations of the model using _gempyor_.&#x20;

First, ensure you have python and R installed. You need a working python3.7+ installation. We recommend using the latest stable python release (python 3.11) to benefit from huge speed-ups and future-proof your installation. We also recommend installing Rstudio to interact with the R code and for exploring your model outputs.

{% hint style="info" %}
_**On Mac**_ 🍏

Python 3 is installed by default on recent MacOS installation. If it is not, you might want to check [homebrew](https://brew.sh/) and install the appropriate installation.&#x20;
{% endhint %}

#### Install packages

To install _gempyor_ and all the necessary dependencies, go to the flepiMoP directory and run the following command line command:

```bash
# Install Python package gempyor (along with dependencies)
pip install -e flepimop/gempyor_pkg/ 
```

{% hint style="danger" %}
_**A warning for Windows**_\
Once _gempyor_ is successfully installed locally, you will need to make sure the executable file `gempyor-seir.exe` is runnable via command line. To do this, you will need to add the directory where it was created to PATH. Follow the instructions [here](https://techpp.com/2021/08/26/set-path-variable-in-windows-guide/) to add the directory where this .exe file is located to PATH. This can be done via GUI or CLI.
{% endhint %}

If you just want to [run a forward simulation](quick-start-guide.md#non-inference-run), _gempyor_ is all you need.

To [run an inference run](quick-start-guide.md#inference-run) and to explore your model outputs using provided post-processing functionality, the following R packages need to be installed.&#x20;

Run the following command in R to install the necessary R packages:

<pre class="language-r" data-overflow="wrap"><code class="lang-r"><strong># while in R
</strong><strong>install.packages(c("readr","sf","lubridate","tidyverse","gridExtra","reticulate","truncnorm","xts","ggfortify","flextable","doParallel","foreach","optparse","arrow","devtools","cowplot","ggraph"))
</strong></code></pre>

Other helper packages are also provided in _flepiMoP_ and can be installed by going to the _flepiMoP_ repository and running the following from command line:

{% code overflow="wrap" %}
```bash
# Install R packages
Rscript build/local_install.R 
```
{% endcode %}

Each installation step may take a few minutes to run.&#x20;

{% hint style="info" %}
Note: These installations take place on your local operating system. You will need an active internet connection for installation, but not for other steps of running the model. \
If you are concerned about disrupting your base environment that you use for other projects, try using [Docker](advanced-run-guides/running-with-docker-locally.md) or [conda](advanced-run-guides/quick-start-guide-conda.md) instead. &#x20;
{% endhint %}

### Define environment variables

First, you'll need to fill in some variables that are used by the model. This can be done in a script (an example is provided at the end of this page). For your first time, it's better to run each command individually to be sure it exits successfully.&#x20;

First, in `myparentfolder` populate the folder name variables for the paths to the flepimop code folder and the project folder:

```bash
export FLEPI_PATH=$(pwd)/flepiMoP
export DATA_PATH=$(pwd)/flepimop_sample
```

## 🚀 Run the code

Everything is now ready 🎉 .&#x20;

The next step depends on what sort of simulation you want to run: One that includes inference (fitting model to data) or only a forward simulation (non-inference). Inference is run from R, while forward-only simulations are run directly from the Python package `gempyor`.

First, navigate to the project folder and make sure to delete any old model output files that are there.

```bash
cd $DATA_PATH       # goes to your project repository
rm -r model_output/ # delete the outputs of past run if there are
```

For the following examples we use an example config from _flepimop\_sample_, but you can provide the name of any configuration file you want.&#x20;

To get started, let's start with just running a forward simulation (non-inference).&#x20;

### Non-inference run

Stay in the `DATA_PATH` folder, and run a simulation directly from forward-simulation Python package gempyor. Call `gempyor-simulate` providing the name of the configuration file you want to run. For example here, we use `config_sample_2pop.yml` from _flepimop\_sample_.&#x20;

```
gempyor-simulate -c config_sample_2pop.yml
```

This will produce a `model_output` folder, which you can look using provided post-processing functions and scripts.&#x20;

We recommend using `model_output_notebook.Rmd` from _flepimop\_sample_ as a starting point to interact with your model outputs. First, modify the yaml preamble in the notebook, then knit this markdown. This will produce some nice plots of the prevalence of infection states over time. You can edit this markdown to produce any figures you'd like to explore your model output.

### Inference run

An inference run requires a configuration file that has the `inference` section. Stay in the `$DATA_PATH` folder, and run the inference script, providing the name of the configuration file you want to run.  For example here, we use `config_sample_2pop_inference.ym`l from _flepimop\_sample_.&#x20;

{% code overflow="wrap" %}
```bash
Rscript  $FLEPI_PATH/flepimop/main_scripts/inference_main.R -c config_sample_2pop_inference.yml
```
{% endcode %}

This will run the model and create a lot of output files in `$DATA_PATH/model_output/`.&#x20;

The last few lines visible on the command prompt should be:

> \[\[1]]
>
> \[\[1]]\[\[1]]
>
> \[\[1]]\[\[1]]\[\[1]]
>
> NULL

If you want to quickly do runs with options different from those encoded in the configuration file, you can do that from the command line, for example

```
Rscript $FLEPI_PATH/flepimop/main_scripts/inference_main.R -j 1 -n 1 -k 1 -c config.yml
```

where:

* `n` is the number of parallel inference slots,
* `j` is the number of CPU cores to use on your machine (if `j` > `n`, only `n` cores will actually be used. If `j` <`n`, some cores will run multiple slots in sequence)
* `k` is the number of iterations per slots.

Again, it is helpful to run the model output notebook to explore your model outputs. Knitting this file for an inference run will also provide an analysis of your fits: the acceptance probabilities, likelihoods overtime, and the fits against the provided ground truth.&#x20;

These configs and notebooks should be a good starting point for getting started with flepiMoP. To explore other running options, see [How to run: Advanced](advanced-run-guides/).&#x20;
