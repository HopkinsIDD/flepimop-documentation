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
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus r-cdltools r-cowplot 
```
{% endcode %}

If the dependencies are hard to satisfy, the above line might take so long that it's killed by the watchdog of rockfish. In this case only, you can create the enviroment in two shorter commands:

{% code overflow="wrap" %}
```bash
# install all python stuff first
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz boto3

# activate the enviromnment and install the R stuff
conda activate covidSP
conda install -c conda-forge r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus r-cdltools r-cowplot 
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

changed because no space in home dir, now /data/struelo1/flepimop-code/sloo2/

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

cd /data/struelo1/flepimop-code/sloo2/covidsp
export COVID_PATH=$(pwd)/COVIDScenarioPipeline

# COVID
export DATA_PATH=$(pwd)/COVID19_USA

#F lu:
export DATA_PATH=$(pwd)/Flu_USA

# set up variables important for inference-job.py
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
export CENSUS_API_KEY="4df5d5d71d8137c46690d7be3e7836f4d64c4194"
<strong>
</strong><strong># prepare the pipeline
</strong>cd $COVID_PATH
git pull	
git checkout main-flu-subfix2 # replace with main once 'cdlTools
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

rm -rf $DATA_PATH/model_output

# if doing a resume

  
<strong>
</strong>Rscript $COVID_PATH/R/scripts/build_US_setup.R

# covid
export RESUME_ID=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan15_tsvacc &#x26;&#x26;
  export RESUME_LOCATION=s3://idd-inference-runs/USA-20230116T053125
export CONFIG_PATH=config_FCH_R16_lowBoo_modVar_ContRes_blk4_Jan22_tsvacc.yml
Rscript $COVID_PATH/R/scripts/build_covid_data.R
export COVID_RUN_INDEX=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan22_tsvacc

# Flu
export RESUME_ID=FCH_R3_highVE_pesImm_2022_Jan15 &#x26;&#x26;
  export RESUME_LOCATION=s3://idd-inference-runs/USA-20230115T202608
export CONFIG_PATH=config_FCH_R3_highVE_pesImm_2022_Jan22.yml
Rscript $COVID_PATH/R/scripts/build_flu_data.R
export COVID_RUN_INDEX=FCH_R3_highVE_pesImm_2022_Jan22

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
</strong><strong>                    --upload-to-s3 True \
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
rm -r model_output
cd $COVID_PATH;git pull;cd $DATA_PATH
rm *.out
```

### Get a notification on your phone/mail when a run is done

We use [ntfy.sh](https://ntfy.sh) for notification. Install ntfy on your Iphone or Android device. Then subscribe to the channel `ntfy.sh/flepimop_alerts` where you'll receive notifications when runs are done.

* End of job notifications goes as urgent priority.

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

