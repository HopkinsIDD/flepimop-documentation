---
description: or any HPC using the slurm workload manager
---

# Running on Rockfish/MARCC

## 🗂️ Files and folder organization

Rockfish administrators provided [several partitions](https://www.arch.jhu.edu/support/storage-and-filesystems/) with different properties. For our needs (storage intensive and shared environment), we work in the `/data/struelo1/` partition, where we have 20T of space. Our folders are organized as:

* **code-folder: ``**` ``/data/struelo1/flepimop-code/` where each user has its own subfolder, from where the repos are cloned and the runs are launched. e.g for user chadi, we'll find:
  * `/data/struelo1/flepimop-code/chadi/covidsp/Flu_USA`
  * `/data/struelo1/flepimop-code/chadi/COVID19_USA`
  * `/data/struelo1/flepimop-code/chadi/COVIDScenarioPipeline`
  * ...
  * (we keep separated repositories by users so that different versions of the pipeline are not mixed where we run several runs in parallel. Don't hesitate to create other subfolders in the code folder (`/data/struelo1/flepimop-code/chadi-flusight`, ...) if you need it.

{% hint style="warning" %}
Note that the repository is cloned **flat,** i.e COVIDScenarioPipeline is in the same level as the data repository, not inside it !
{% endhint %}

* **output folder:**`/data/struelo1/flepimop-runs` stores the run outputs. After an inference run finishes, it's output and the logs files are copied from the `$DATA_PATH/model_output` to `/data/struelo1/flepimop-runs/THISRUNJOBNAME` where the jobname is usually of the form `USA-DATE.`

## Login on rockfish

Using ssh from your terminal, type in:

```
ssh {YOUR ROCKFISH USERNAME}@login.rockfish.jhu.edu
```

and enter your password when prompted. You'll be into rockfish's login node, which is a remote computer whose only purpose is to prepare and launch computations on so-called compute nodes.

## 🧱 Setup (to be done only once per USER )

Load the right modules for the setup:

```bash
module purge
module load gcc/9.3.0
module load anaconda3/2022.05  # very important to pin this version as other are buggy
module load git                # needed for git
module load git-lfs            # git-lfs
```

Now you need to create the conda environment. This command is quite long you'll have the time to brew some nice coffee ☕️:

{% code overflow="wrap" %}
```bash
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz boto3 slack_sdk r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus r-cdltools r-cowplot 
```
{% endcode %}

<details>

<summary>Expand if the conda does not work for no particular reason (just interrupted)</summary>

If the dependencies are hard to satisfy, the above line might take so long that it's killed by the watchdog of rockfish. In this case, only, you can create the environment in two shorter commands:

{% code overflow="wrap" %}
```bash
# install all python stuff first
conda create -c conda-forge -n covidSP numba pandas numpy seaborn tqdm matplotlib click confuse pyarrow sympy dask pytest scipy graphviz boto3 slack_sdk

# activate the enviromnment and install the R stuff
conda activate covidSP
conda install -c conda-forge r-readr r-sf r-lubridate r-tigris r-tidyverse r-gridextra r-reticulate r-truncnorm r-xts r-ggfortify r-flextable r-doparallel r-foreach r-arrow r-optparse r-devtools r-tidycensus r-cdltools r-cowplot 
```
{% endcode %}

</details>

### Create the directory structure

type the following commands. $USER is a variable that contains your username.

<pre class="language-bash"><code class="lang-bash"><strong>cd /data/struelo1/flepimop-code/
</strong><strong>mkdir $USER
</strong><strong>cd $USER
</strong>git clone https://github.com/HopkinsIDD/COVIDScenarioPipeline.git
git clone https://github.com/HopkinsIDD/Flu_USA.git
git clone https://github.com/HopkinsIDD/COVID19_USA.git
# or any other data directories
</code></pre>

### Setup your AWS credentials (allows to copy runs to s3)

This can be done in a second step --  but is necessary in order to push and pull to s3. Setup your AWS credentials by:

<pre class="language-bash"><code class="lang-bash">cd ~ # go in your home directory
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
./aws/install -i ~/aws-cli -b ~/aws-cli/bin
<strong>./aws-cli/bin/aws --version
</strong></code></pre>

Then run `./aws-cli/bin/aws configure`  and use the following :

```
# AWS Access Key ID [None]: YOUR ID
# AWS Secret Access Key [None]: YOUR SECRET ID
# Default region name [None]: us-west-2
# Default output format [None]: json
```

That's it, you are all setup to run.

## 🚀 Run inference using slurm (do everytime)

log-in to rockfish via ssh, then type

`source /data/struelo1/flepimop-code/flepimop_init.sh`

<details>

<summary>what does this do || it returns an error</summary>

This script runs the following commands, which you can run individually as well.

```bash
module purge
module load gcc/9.3.0
module load git
module load git-lfs
module load slurm
module load anaconda3/2022.05
conda activate covidSP
export CENSUS_API_KEY={A CENSUS API KEY}
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE

```

</details>

Then, check that the conda environment is activated: you should see `(covidSP)` on the left of your command-line prompt. Prepare the pipeline directory (if you have already done that and the pipeline hasn't been updated (`git pull` says it's up to date) then you can skip these steps&#x20;

```bash
cd /data/struelo1/flepimop-code/$USER
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
cd $COVID_PATH
git checkout main-flu-subfix2    # replace with main once this PR is merged.
git pull
git lfs pull

#install gempyor and the R module. There should be no error, please report if not.
# Sometime you might need to run the next line two times because inference depends
# on report.generation, which is installed later because alphabetical order.
# (or if you know R well enough to fix that 😊)

Rscript local_install.R # warnings are ok; there should be no error.
pip install --no-deps -e gempyor_pkg/
```

Now flepiMoP is ready 🎉. Now you need to set $DATA\_PATH to your data folder. For a COVID-19 run, do:

```bash
cd /data/struelo1/flepimop-code/$USER
export DATA_PATH=$(pwd)/COVID19_USA
```

for Flu do:&#x20;

<pre class="language-bash"><code class="lang-bash">cd /data/struelo1/flepimop-code/$USER
export DATA_PATH=$(pwd)/Flu_USA

# set up variables important for inference-job.py
export CENSUS_API_KEY="4df5d5d71d8137c46690d7be3e7836f4d64c4194"
<strong>
</strong>
cd $DATA_PATH
git pull 
git checkout main

export VALIDATION_DATE="2023-01-29" &#x26;&#x26; 
   rm -rf model_output data/us_data.csv data-truth &#x26;&#x26;
   rm -rf data/mobility_territories.csv data/geodata_territories.csv &#x26;&#x26;
   rm -rf data/seeding_territories.csv &#x26;&#x26; 
   rm -rf data/seeding_territories_Level5.csv data/seeding_territories_Level67.csv

rm -rf $DATA_PATH/model_output
rm *.out

# if doing a resume

  
<strong>
</strong>Rscript $COVID_PATH/R/scripts/build_US_setup.R

# covid
export RESUME_ID=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan22_tsvacc &#x26;&#x26;
  export RESUME_LOCATION=s3://idd-inference-runs/USA-20230122T145824
export CONFIG_PATH=config_FCH_R16_lowBoo_modVar_ContRes_blk4_Jan29_tsvacc.yml
Rscript $COVID_PATH/R/scripts/build_covid_data.R
<strong>export COVID_RUN_INDEX=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan29_tsvacc
</strong>
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

### Launch your inference batch job

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



### Monitor your run

Two types of logfiles: in \`$DATA\_PATH\`: slurm-JOBID\_SLOTID.out and and filter\_MC logs:

\`\`\`tail -f /data/struelo1/flepimop-runs/USA-20230130T163847/log\_FCH\_R16\_lowBoo\_modVar\_ContRes\_blk4\_Jan29\_tsvacc\_100.txt

\`\`\`



###

## Common errors

* Check that the python comes from conda with `which python` if some weird package missing errors arrive. Sometime conda magically disappears.&#x20;
* Don't use `ipython` as it breaks click's flags



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

## How to use slurm

Check your running jobs:

```
squeue -u $USER
```

where job\_id has your full array job\_id and each slot after the under-score. You can see their status (R: running, P: pending), how long they have been running and soo on.

To cancel a job

```
scancel JOB_ID
```



## Installation notes

These steps are already done an affects all users, but might be interesting in case you'd like to run on another cluster

#### Install slack integration

So our 🤖-friend can send us some notifications once a run is done.

```
cd /data/struelo1/flepimop-code/
nano slack_credentials.sh
# and fill the file:
export SLACK_WEBHOOK="{THE SLACK WEBHOOK FOR CSP_PRODUCTION}"
export SLACK_TOKEN="{THE SLACK TOKEN}"
```



