# Python guidelines for developers

The "heart" of the pipeline, gempyor, is written in python taking advantage of just-in-time compilation (via. numba) and existing optimized librairies (numpy, pandas).

We use [black](https://github.com/psf/black), the _Uncomprolsmising Code Formatter_ before submitting pull-requests. It provides a consistent style, which is useful when diffing. We use a custom lenght of 120 characters as the baseline is short for scienfitic code. Here is the line to use to format your code:

```
black --line-length 120 . --exclude renv*
```

Please use type-hints as much as possible, as we are trying to move towards static checks.

### Structure of the main classes

The main classes, such as `Parameter`, `NPI`, `SeedingAndInitialConditions`,`Compartments` should tend to the same struture:

* a `writeDF`
* function to plot
* (TODO: detail pipeline internal API)

### Batch folder

Here are some notes useful to improve the batch submission:

Setup site wide Rprofile.

```
export R_PROFILE=$COVID_PATH/slurm_batch/Rprofile
```

> SLURM copies your environment variables by default. You don't need to tell it to set a variable on the command line for sbatch. Just set the variable in your environment before calling sbatch.

> There are two useful environment variables that SLURM sets up when you use job arrays:

> SLURM\_ARRAY\_JOB\_ID, specifies the array's master job ID number. SLURM\_ARRAY\_TASK\_ID, specifies the job array index number. https://help.rc.ufl.edu/doc/Using\_Variables\_in\_SLURM\_Jobs

SLURM does not support using variables in the #SBATCH lines within a job script (for example, #SBATCH -N=$REPS will NOT work). A very limited number of variables are available in the #SBATCH just as %j for JOB ID. However, values passed from the command line have precedence over values defined in the job script. and you could use variables in the command line. For example, you could set the job name and output/error files can be passed on the sbatch command line:

```
RUNTYPE='test'
RUNNUMBER=5
sbatch --job-name=$RUNTYPE.$RUNNUMBER.run --output=$RUNTYPE.$RUNUMBER.txt --export=A=$A,b=$b jobscript.sbatch
```

However note in this example, the output file doesn't have the job ID which is not available from the command line, only inside the sbatch shell script.

#### File descriptions

launch\_job.py and runner.py for non inference job

inference\_job.py launch a slurm or aws job, where it uses

* \`inference\_runner.sh\` and inference\_copy.sh for aws
* &#x20;batch/inference\_job.run for slurm
