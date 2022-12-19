# Slurm submission on MARCC

### Setup (do ONCE per user)

Load the right modules for the setup

```
module purge
module load anaconda3/2022.05
module load git # needed for git
module load git-lfs # git-lfs
```

{% code overflow="wrap" %}
```bash
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

Sometime that won't work, so do it in two times

{% code overflow="wrap" %}
```
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz boto3
conda activate covidSP
conda install -c conda-forge r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

### AWS setup (in case you want runs to be save to s3 as well)

and setup your AWS credentials:

### # To run a script

```
module purge
module load anaconda3/2022.05
module load git
module load git-lfs
module load slurm
conda activate covidSP
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
export DATA_PATH=$(pwd)/Flu_USA
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
cd $COVID_PATH
Rscript local_install.R
pip install --no-deps -e gempyor_pkg/ 
git lfs pull
export CENSUS_API_KEY="4df5d5d71d8137c46690d7be3e7836f4d64c4194"
cd $DATA_PATH
git restore data/
rm -rf data/mobility_territories.csv data/geodata_territories.csv data/us_data.csv
export CONFIG_PATH=config_SMH_R1_lowVac_optImm_2022.yml
Rscript $COVID_PATH/R/scripts/build_US_setup.R
```

### Then run the submission

```
python $COVID_PATH/batch/inference_job.py --slurm -c $CONFIG_PATH --pipepath $COVID_PATH --data-path $DATA_PATH --upload-to-s3 False --fs-folder test_run
```

