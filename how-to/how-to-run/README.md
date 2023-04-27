# How to Run

Several different platforms can be used to run flepiMoP. Before you get started, we recommend you install some software and packages. Some of these are required, while some are helper tools or specific to your preferred pipeline.

This section provides instructions for installation and how to run on each platform. We recommend `conda` for beginners, and AWS for large simulations.&#x20;

{% content-ref url="quick-start-guide-conda.md" %}
[quick-start-guide-conda.md](quick-start-guide-conda.md)
{% endcontent-ref %}

{% content-ref url="running-with-docker-locally.md" %}
[running-with-docker-locally.md](running-with-docker-locally.md)
{% endcontent-ref %}

{% content-ref url="slurm-submission-on-marcc.md" %}
[slurm-submission-on-marcc.md](slurm-submission-on-marcc.md)
{% endcontent-ref %}

{% content-ref url="../../deprecated-pages/running-with-docker-on-aws/" %}
[running-with-docker-on-aws](../../deprecated-pages/running-with-docker-on-aws/)
{% endcontent-ref %}

## Setup requirements

The following are required to run flepiMoP regardless of what platform you are running it on.&#x20;

* Git (Git, LFS), R, Python

Instructions on their individual installations for each platform can be found within each instruction guide linked above.

#### 📂 Getting the repositories

In your preferred folder, clone the `COVIDScenarioPipeline` and relevant data (`COVID19_USA` or `Flu_USA)` repositories. You should have a directory structure as:

```
.../myparentfolder  # project folder
.../myparentfolder/Flu_USA
.../myparentfolder/COVIDScenarioPipeline
```

In `COVIDScenarioPipeline`, do `git checkout main` to make sure you are on the `main` branch. You can also use the GitHub App to clone and checkout if you prefer.

{% hint style="warning" %}
In general repositories are cloned **flat,** i.e `COVIDScenarioPipeline` is at the same level as the data repository, not inside it!

_**FOR AWS,**_ `COVIDScenarioPipeline` must be nested **within** data repositories.&#x20;
{% endhint %}

In each of our "Running with..." pages, we will assume you have all the necessary repositories all cloned ready to go.

#### 🐍 Python&#x20;

The python part of the model is named `gempyor`, and encompasses everything to simulate the transmission of an infectious disease epidemic: health outcomes, interventions, transmission.

**Installing `gempyor`**

You need a working python3.7+ installation. We recommend using the latest stable python release (python 3.11) to benefit from huge speed-ups and future-proof your installation. You may want to use `conda` to manage the packages if you're not used to python. Instructions for this are in 'Running with conda locally'.&#x20;







TODO: remove the below when finished organising subpages (complete instructions are in subpages)

**Using conda (recommended for beginner)**

After you install conda, let's create a conda environment with everything needed:

```python
conda create -n covidSP python=3.10 # TODO change to 3.11 when released
conda activate covidSP
conda install -c conda-forge jupyterlab ipykernel dask click confuse matplotlib numba">=0.53" numpy pandas pyarrow pytest scipy seaborn sympy tqdm python-graphviz
```

then you can activate this new conda environment with `conda activate covidSP`, and install gempyor as:

```
pip install --no-deps -e gempyor_pkg/
```

You'd need to work in this environment when using gempyor (you can set that from Rstudio, or using reticulate).

**Without conda**

If you are confident you'll be using python just for this, you may install gempyor and all its dependencies using:

```bash
pip install -e gempyor_pkg/
```

where `-e` ensures that when you pull the pipeline repository again, there is no need for re-installing when pulling a new version. In case you installed manually the dependencies, and you don't want to risk updating them because of your other python projects you may use:

```
pip install --no-deps -e gempyor_pkg/
```

Otherwise, you can use you favorite tool to create a virtual environment and run the above line from here.

**Installing gempyor in the docker**

In the docker we have to update pip before:

```
/home/app/python_venv/bin/python3.7 -m pip install --upgrade pip # needs new pip for toml file
pip install -e gempyor_pkg/
```



**Test your installation:**

You can test your installation by running `python` and then

```python
import gempyor
```

which should run without any error.
