---
description: Short internal tutorials on running locally using available options.
---

# Quick Start Guide



{% tabs %}
{% tab title="Anaconda" %}
### Setup (do it once)

#### Getting the repositories

In your preferred folder, clone the `COVIDScenarioPipeline` and `Flu_USA` repositories. You should have a directory structure as:

```
.../myparentfolder  # project folder
.../myparentfolder/Flu_USA
.../myparentfolder/COVIDScenarioPipeline
```

in `COVIDScenarioPipeline`, do `git checkout main` to make sure you are on the `main` branch. You can also use the GitHub App to clone and checkout if you prefer, it's great.

#### Installing the conda environment

The simplest way to get everything to work is to build an Anaconda environment. Install (or update) Anaconda on your computer. You can either use the command line (here) or the graphical user interface (you just tick the packages you want). With the command line it's this one-liner:

{% code overflow="wrap" %}
```bash
conda update conda # makes sure you have a recent conda instatllation

# be sure to copy the whole thing as a single line ! copy it to your text editor
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

Anaconda will take some time, to come up with a proposal that works will all dependencies. On my computer, it proposes versions `r-base==4.1.3` and `python==3.10.6`, which is great. But really anything recent is good. Install the packages. It'll create a conda environment named `covidSP` that has all the necessary packages. &#x20;

If you'd like you can install `rstudio` as a package as well, but I think your Rstudio should be able to find your conda R without problems.

{% hint style="info" %}
In fact, Anaconda is just the most reproducible way. You don't really need it. You can just carry on with the steps below without creating an environment if you'd like, and it might be easier. **How to do it?** just skip every line starting with `conda` and do not use the `--no-deps` flag when installing gempyor (so pip will install the dependencies). Then when running `local_install.R` there might be failures because some packages are missing. Install them as you usually do it from R. The rest is the same as this tutorial.
{% endhint %}

### How to run inference
{% endtab %}

{% tab title="Docker" %}
###
{% endtab %}

{% tab title="AWS" %}
### Amazon Web Services (AWS)

For large simulations, running the model on a cluster or cloud computing is required. AWS provides a good solution for this, if funding or credits are available (AWS can get very expensive). \
\

{% endtab %}
{% endtabs %}



