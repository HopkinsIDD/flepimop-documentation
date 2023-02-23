---
description: or any HPC using the slurm workload manager
---

# Running on Rockfish/MARCC 🪨🐠

## 🗂️ Files and folder organization

Rockfish administrators provided [several partitions](https://www.arch.jhu.edu/support/storage-and-filesystems/) with different properties. For our needs (storage intensive and shared environment), we work in the `/data/struelo1/` partition, where we have 20T of space. Our folders are organized as:

* **code-folder: ``**` ``/data/struelo1/flepimop-code/` where each user has its own subfolder, from where the repos are cloned and the runs are launched. e.g for user chadi, we'll find:
  * `/data/struelo1/flepimop-code/chadi/covidsp/Flu_USA`
  * `/data/struelo1/flepimop-code/chadi/COVID19_USA`
  * `/data/struelo1/flepimop-code/chadi/COVIDScenarioPipeline`
  * ...
  * (we keep separated repositories by users so that different versions of the pipeline are not mixed where we run several runs in parallel. Don't hesitate to create other subfolders in the code folder (`/data/struelo1/flepimop-code/chadi-flusight`, ...) if you need them.

{% hint style="warning" %}
Note that the repository is cloned **flat,** i.e `COVIDScenarioPipeline` is at the same level as the data repository, not inside it!
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

Now, type the following line so git remembers your credential and you don't have to enter your token 6 times per day:

```bash
git config --global credential.helper store
git config --global user.name "{NAME SURNAME}"
git config --global user.email YOUREMAIL@EMAIL.COM
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

## 🚀 Run inference using slurm (do everytime)

log-in to rockfish via ssh, then type:

`source /data/struelo1/flepimop-code/flepimop_init.sh`

which will prepare the environment and setup variables for the validation date, the resume location and the run index for this run

{% hint style="success" %}
Note that now the run-id of the run we resume from is automatically inferred by the batch script :)
{% endhint %}

<details>

<summary>what does this do || it returns an error</summary>

This script runs the following commands to setup up the environment, which you can run individually as well.

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

# And then it asks you some questions to setup some enviroment variables
```

and the it does some prompts to fix the following 3 enviroment variables. You can skip this part and do it later manually.&#x20;

```bash
export VALIDATION_DATE="2023-01-29"
export RESUME_LOCATION=s3://idd-inference-runs/USA-20230122T145824
export COVID_RUN_INDEX=FCH_R16_lowBoo_modVar_ContRes_blk4_Jan29_tsvacc
```

</details>

{% hint style="warning" %}
Check that the conda environment is activated: you should see `(covidSP)` on the left of your command-line prompt.
{% endhint %}

Then prepare the pipeline directory (if you have already done that and the pipeline hasn't been updated (`git pull` says it's up to date) then you can skip these steps&#x20;

```bash
cd /data/struelo1/flepimop-code/$USER
export COVID_PATH=$(pwd)/COVIDScenarioPipeline
cd $COVID_PATH
git checkout main
git pull
git lfs install
git lfs pull

#install gempyor and the R module. There should be no error, please report if not.
# Sometimes you might need to run the next line two times because inference depends
# on report.generation, which is installed later because of alphabetical order.
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

```bash
cd /data/struelo1/flepimop-code/$USER
export DATA_PATH=$(pwd)/Flu_USA
```

Now for any type of run:

```bash
cd $DATA_PATH
git pull 
git checkout main
```

Do some clean-up before your run. The fast way is to restore the `$DATA_PATH` git repository to its blank states (removes everything that does not come from git):

```bash
git reset --hard && git clean -f -d  # this deletes everything that is not on github in this repo !!!
```

<details>

<summary>I want more control over what is deleted</summary>

&#x20;if you prefer to have more control, delete the files you like, e.g

If you still want to use git to clean the repo but want finer control or to understand how dangerous is the command, [read this](https://stackoverflow.com/questions/1090309/git-undo-all-working-dir-changes-including-new-files).

```bash
rm -rf model_output data/us_data.csv data-truth &&
   rm -rf data/mobility_territories.csv data/geodata_territories.csv &&
   rm -rf data/seeding_territories.csv && 
   rm -rf data/seeding_territories_Level5.csv data/seeding_territories_Level67.csv

# don't delete model_output if you have another run in //
rm -rf $DATA_PATH/model_output
# delete log files from previous runs
rm *.out
```

</details>

Then run the preparatory script and you are good

```bash
export CONFIG_PATH=config_FCH_R16_lowBoo_modVar_ContRes_blk4_Jan29_tsvacc.yml
Rscript $COVID_PATH/R/scripts/build_US_setup.R

# For covid do
Rscript $COVID_PATH/R/scripts/build_covid_data.R

# For Flu do
Rscript $COVID_PATH/R/scripts/build_flu_data.R
```

Now you may want to test that it works :&#x20;

```bash
Rscript $COVID_PATH/R/scripts/full_filter.R -c $CONFIG_PATH -j 1 -n 1 -k 1 
```

If this fails, you may want to investigate this error. In case this succeeds, then you can proceed by first deleting the model\_output:

```
rm -r model_output
```

### Launch your inference batch job

Type the following command to launch:&#x20;

```bash
python $COVID_PATH/batch/inference_job.py --slurm 2>&1 | tee $COVID_RUN_INDEX_submission.log
```

This command infers everything from you environment variables, if there is a resume or not, what is the run\_id, etc. The part after the "2" makes sure this file output is redirected to a script for logging, but has no impact on your submission.

If you'd like to have more control, you can specify the arguments manually:

<pre class="language-bash"><code class="lang-bash"><strong>python $COVID_PATH/batch/inference_job.py --slurm \
</strong><strong>                    -c $CONFIG_PATH \
</strong><strong>                    --pipepath $COVID_PATH \
</strong><strong>                    --data-path $DATA_PATH \
</strong><strong>                    --upload-to-s3 True \
</strong><strong>                    --id $COVID_RUN_INDEX \
</strong><strong>                    --fs-folder /data/struelo1/flepimop-runs \
</strong><strong>                    --restart-from-location $RESUME_LOCATION
</strong></code></pre>

**Commit files to Github.** After the job is successfully submitted, you will now be in a new branch of the data repo. Commit the ground truth data files to the branch on github and then return to the main branch:

<pre><code><strong>git add data/ 
</strong>git commit -m"scenario run initial" 
branch=$(git branch | sed -n -e 's/^\* \(.*\)/\1/p')
git push --set-upstream origin $branch

git checkout main
git pull
</code></pre>

### Monitor your run

TODO JPSEH WRITE UP TO HERE

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

### Running an interactive session

To check your code prior to submitting a large batch job, it's often helpful to run an interactive session to debug your code and check everything works as you want. On 🪨🐠 this can be done using `interact` like the below line, which requests an interactive session with 4 cores, 24GB of memory, for 12 hours.&#x20;

```
interact -p defq -n 4 -m 24G -t 12:00:00
```

The options here are `[-n tasks or cores]`, `[-t walltime]`, `[-p partition]` and `[-m memory]`, though other options can also be included or modified to your requirements. More details can be found on the [ARCH User Guide](https://marcc.readthedocs.io/Slurm.html#request-interactive-jobs).&#x20;

### Moving files to your local computer

Often you'll need to move files back and forth between Rockfish and your local computer. To do this, you can use Open-On-Demand, or any other command line tool.

`scp -r <user>@rfdtn1.rockfish.jhu.edu:"<file path of what you want>" <where you want to put it in your local>`

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

### The helper script

in `/data/struelo1/flepimop-code/flepimop_init.sh`

```bash
echo ">>> running some useful commands  ^= ^v, please wait"
module purge
module load gcc/9.3.0
module load git
module load git-lfs
module load slurm
module load anaconda3/2022.05
conda activate covidSP
export CENSUS_API_KEY="6a98b751a5a7a6fc365d14fa8e825d5785138935"  # joseph's key
export COVID_STOCHASTIC=false
export COVID_RESET_CHIMERICS=TRUE
export COVID_PATH=/data/struelo1/flepimop-code/$USER/COVIDScenarioPipeline
echo "done  ^|^e"

echo "Doing some inference_run setup, hit enter to skip a question if not relevant or you're not planning on doing a run"

export TODAY=`date --rfc-3339='date'`
echo "Let's set up a flepiMoP run. Today is the $TODAY"

echo "(1/3) Please input the validation date:"
read input
export VALIDATION_DATE="$input"
echo -e ">>> set VALIDATION DATE to $VALIDATION_DATE \n"

echo "(2/3) Please input the resume location (empty if no resume, or a s3:// url or a local folder):"
read input
export RESUME_LOCATION="$input"
echo -e ">>> set RESUME_LOCATION to $RESUME_LOCATION \n"

echo "(3/3) Please provide the Run Index for the current run:"
read input
export COVID_RUN_INDEX="$input"
echo -e ">>> set  COVID_RUN_INDEX to $COVID_RUN_INDEX \n"

echo "DONE. if no error please manually set export CONFIG_PATH=YOURCONFIGPATH.yml"
echo "(in case of error, override manually some variables or rerun this script)"

```

