# Slurm submission on MARCC

### Before anything

```
module purge
module load anaconda3/2022.05
module load git # needed for git
module load git-lfs # git-lfs
```

### Setup (do ONCE per user)

Load the right modules for the setup

{% code overflow="wrap" %}
```bash
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

Sometime that won't work, so do it in two times

{% code overflow="wrap" %}
```bash
# install all python stuff first
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz boto3

# activate the enviromnment and install the R stuff
conda activate covidSP
conda install -c conda-forge r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus
```
{% endcode %}

### Directory setup:

```
mkdir covidsp
cd covidsp
git clone https://github.com/HopkinsIDD/COVIDScenarioPipeline.git
cd COVIDScenarioPipeline
git checkout main
cd ..
git clone https://github.com/HopkinsIDD/Flu_USA.git
git clone https://github.com/HopkinsIDD/COVID19_USA.git
```

### AWS setup (in case you want runs to be save to s3 as well)

and setup your AWS credentials:

<pre><code>cd ~
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install -i ~/aws-cli -b ~/aws-cli/bin
<strong>./aws-cli/bin/aws --version
</strong></code></pre>

Then

```
./aws-cli/bin/aws configure
# AWS Access Key ID [None]: YOUR ID
# AWS Secret Access Key [None]: YOUR SECRET ID
# Default region name [None]: us-east-1
# Default output format [None]: json
```

### # To run a script

<pre><code>module purge
module load gcc/9.3.0

module load git
module load git-lfs
module load slurm
module load anaconda3/2022.05
conda activate covidSP


cd covidsp
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
export DATA_PATH=$(pwd)/COVID19_USA
#or export DATA_PATH=$(pwd)/Flu_USA

# set up variables important for inference-job.py
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
export CENSUS_API_KEY="4df5d5d71d8137c46690d7be3e7836f4d64c4194"
<strong>
</strong><strong># prepare the pipeline
</strong>cd $COVID_PATH
git pull	
git checkout main
Rscript local_install.R # warnings are ok; there should be no error 
pip install --no-deps -e gempyor_pkg/ 
git lfs pull



cd $DATA_PATH
git pull 
git checkout main

export VALIDATION_DATE="2023-01-22" &#x26;&#x26; 
   rm -rf model_output data/us_data.csv data-truth &#x26;&#x26;
   rm -rf data/mobility_territories.csv data/geodata_territories.csv &#x26;&#x26;
   rm -rf data/seeding_territories.csv &#x26;&#x26; 
   rm -rf data/seeding_territories_Level5.csv data/seeding_territories_Level67.csv

# if doing a resume
export RESUME_ID=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan15_tsvacc &#x26;&#x26;
  export RESUME_LOCATION=s3://idd-inference-runs/USA-20230116T053125
  
rm -rf $DATA_PATH/model_output
<strong>export CONFIG_PATH=config_FCH_R16_lowBoo_modVar_ContRes_blk4_Jan22_tsvacc.yml
</strong>Rscript $COVID_PATH/R/scripts/build_US_setup.R
Rscript $COVID_PATH/R/scripts/build_flu_data.R
# or  Rscript $COVID_PATH/R/scripts/build_covid_data.R

export COVID_RUN_INDEX=test

</code></pre>

\
If you get an error running local install, do \`nano \~/.Rprofile\`and put the following in it.

```r
local({r <- getOption("repos")
       r["CRAN"] <- "http://cran.r-project.org"
       options(repos=r)
})
```

### Then run the submission

<pre class="language-bash"><code class="lang-bash"><strong>python $COVID_PATH/batch/inference_job.py --slurm \
</strong><strong>                    -c $CONFIG_PATH \
</strong><strong>                    --pipepath $COVID_PATH \
</strong><strong>                    --data-path $DATA_PATH \
</strong><strong>                    --upload-to-s3 False \
</strong><strong>                    --id $COVID_RUN_INDEX \
</strong><strong>                    --fs-folder /data/struelo1/flepimop-runs \
</strong><strong>                    --restart-from-location $RESUME_LOCATION \
</strong><strong>                    --restart-from-run-id=$RESUME_ID
</strong></code></pre>

Default folder to copy the results: \`/data/struelo1/flepimop-runs\` where Shaun should have 20T. See [https://www.arch.jhu.edu/support/storage-and-filesystems/](https://www.arch.jhu.edu/support/storage-and-filesystems/)





### Error I got

* Check that the python comes from conda with \`which python\` otherwise reload the modules
* Don't use ipython it breaks click's flags



cleanup:

```
rm -r /data/struelo1/flepimop-runs/
```

```
cd $COVID_PATH;git pull;cd $DATA_PATH
```

```
rm *.out
```

```
rm -r model_output
```

### Slurm-fu

Check your running jobs:

```
squeue -u sloo2
```

where job\_id has your full array job\_id and each slot after the under-score. You can see their status (R: running, P: pending), how long they have been running and soo on.

To cancel a job

```
scancel JOB_ID
```
