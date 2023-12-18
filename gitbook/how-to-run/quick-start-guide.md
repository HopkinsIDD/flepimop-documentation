---
description: >-
  Instructions to get started with using gempyor and flepiMoP, using some
  provided example configs to help you.
---

# Quick Start Guide

##  📂 Access model files (this gets repeated on each run page- does this matter)

As is the case for any run, first see the [Before any run](before-any-run.md) section to ensure you have access to the correct files needed to run. On your local machine, determine the file paths to:

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

(this is repeated from before any run - need or remove?)

The code is written in a combination of [R](https://www.r-project.org/) and [Python](https://www.python.org/). The Python part of the model is a package called [_gempyor_](../gempyor/model-description.md), and includes all the code to simulate the epidemic model and the observational model and apply time-dependent interventions. The R component conducts the (optional) parameter inference, and all the (optional) provided pre and post processing scripts are also written in R. Most uses of the code require interacting with components written in both languages, and thus making sure that both are installed along with a set of required packages. However, Python alone can be used to do forward simulations of the model using _gempyor_.&#x20;

First, ensure you have python and R installed. You need a working python3.7+ installation. We recommend using the latest stable python release (python 3.11) to benefit from huge speed-ups and future-proof your installation.&#x20;

### Define environment variables

First, you'll need to fill in some variables that are used by the model. This can be done in a script (an example is provided at the end of this page). For your first time, it's better to run each command individually to be sure it exits successfully.&#x20;

First, in `myparentfolder` populate the folder name variables for the paths to the flepimop code folder and the project folder:

```bash
export FLEPI_PATH=$(pwd)/flepiMoP
export DATA_PATH=$(pwd)/flepimop_sample
```

Go into the code directory (making sure it is up to date on your favorite branch) and do the installation required of the repository:

{% code overflow="wrap" %}
```bash
cd $FLEPI_PATH # move to the flepimop directory

# Install R packages
Rscript build/local_install.R 

# Install Python package gempyor (along with dependencies)
pip install -e flepimop/gempyor_pkg/ 
```
{% endcode %}

Each installation step may take a few minutes to run.&#x20;

{% hint style="info" %}
Note: These installations take place on your local operating system. You will need an active internet connection for installation, but not for other steps of running the model. \
If you are concerned about disrupting your base environment that you use for other projects, try using [Docker](running-with-docker-locally.md) or [conda](quick-start-guide-conda.md) instead. &#x20;
{% endhint %}

## 🚀 Run the code

Everything is now ready 🎉 .&#x20;

To get started, let's start with just running a forward simulation (non-inference). This is run directly from the Python package _gempyor_.&#x20;

First, navigate to the project folder and make sure to delete any old model output files that are there.

```bash
cd $DATA_PATH       # goes to your project repository
rm -r model_output/ # delete the outputs of past run if there are
```

### Non-inference run

Stay in the `DATA_PATH` folder, and run a simulation directly from forward-simulation Python package gempyor. Call `gempyor-simulate` providing the name of the configuration file you want to run (ex. `config.yml`)&#x20;

```
gempyor-simulate -c config.yml
```

This will produce a `model_output` folder, which you can look at by loading ...&#x20;



(sloo got up to here)

The next step depends on what sort of simulation you want to run: One that includes inference (fitting model to data) or only a forward simulation (non-inference). Inference is run from R, while forward-only simulations are run directly from the Python package `gempyor`.



### Inference run

An inference run requires a configuration file that has the `inference` section. Stay in the `$DATA_PATH` folder, and run the inference script, providing the name of the configuration file you want to run (ex. `config.yml`)&#x20;

```bash
Rscript  $FLEPI_PATH/flepimop/main_scripts/inference_main.R -c config.yml
```

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

You can put all of this together into a single script that can be run all at once:&#x20;

<pre><code>docker pull hopkinsidd/flepimop:latest-dev
docker run -it \
  -v &#x3C;dir1>:/home/app/flepimop \
  -v &#x3C;dir2>:/home/app/drp \
hopkinsidd/flepimop:latest-dev
<strong>export FLEPI_PATH=/home/app/flepimop/
</strong>export DATA_PATH=/home/app/drp/
cd $FLEPI_PATH
Rscript build/local_install.R
pip install --no-deps -e flepimop/gempyor_pkg/
cd $DATA_PATH
rm -rf model_output
Rscript $FLEPI_PATH/flepimop/main_scripts/inference_main.R -j 1 -n 1 -k 1 -c config.yml
</code></pre>

###









#### On MacOS

Python 3 is installed by default on recent MacOS installation. If it is not, you might want to check [homebrew](https://brew.sh) and ... You can also install gempyor within a virtual env or any conda enviroment.

#### On Linux

#### On Windows

...\
\
Once `gempyor` is successfully installed locally \[add text about what that looks like], you will need to make sure the executable file `gempyor-seir.exe` is runnable via command line. To do this, you will need to add the directory where it was created to PATH. Follow the instructions [here](https://techpp.com/2021/08/26/set-path-variable-in-windows-guide/) to add the directory where this .exe file is located to PATH. This can be done via GUI or CLI.

