# How to Run

### Installation

Git (Git, Github Desktop, Github, LFS) Docker R

#### Python

The python part of the model is named gempyor, and encompasses the everything to simulate the transmission of an infectious disease epidemic: health outcomes, interventions, transmission, ... but not e.g inference.

You probably want to install gempyor both on your local file system (so you can play with the model, analyse the results), and on docker (to replicate the environment for run, or to run inference locally).

**Installing gempyor locally**

You'd need a working python3.7+ installation. We recommend using the latest stable python release (python 3.11) to benefits from huge speed-ups and future proof your installation. You may want to use conda to manage package if you're not used to python.

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

**TODO update with new docker**

**Test your installation:**

You can test your installation by running `python` and then

```python
import gempyor
```

which should run without any error.

### Running locally

Via Docker/terminal only Via Docker/terminal + using R studio only Via Docker/terminal + Python only

Initiating Docker, downloading Github repos, setting up model, running model tests, finding and interpreting model output

### Running on AWS

Ie update of [AWS Instructions](https://github.com/HopkinsIDD/COVIDScenarioPipeline/wiki/AWS-Instructions) on current wiki

### Visualizing and interpreting results

(Output file structure, how to read files, sample report generation functions and visualizations, etc)
