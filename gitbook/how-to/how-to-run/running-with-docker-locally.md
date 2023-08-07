---
description: >-
  Short internal tutorial on running locally using a "Docker" container. Note
  that these directions are updated from the previous to remove specifics for US
  COVID-19 models and include hints for Macs
---

# Running with Docker locally (updated) 🛳

{% hint style="info" %}
**Helpful tools**

To understand the basics of docker refer to the following: [Docker Basics](https://www.docker.com/)

To install docker for Mac, refer to the following link: [Installing Docker for Mac](https://docs.docker.com/desktop/install/mac-install/). Pay special attention to the specific chip your Mac has (Apple Silicon vs Intel), as installation files and directions differ

To install docker for Windows, refer to the following link: [Installing Docker for Windows](https://docs.docker.com/desktop/windows/install/)

The following is a good tutorial for introduction to docker: [Docker Tutorial](https://www.youtube.com/watch?v=gFjxB0Jn8Wo\&list=PL6gx4Cwl9DGBkvpSIgwchk0glHLz7CQ-7)

To run the entire pipeline we use the command prompt. To open the command prompt type “Command Prompt" in the search bar and open the command prompt. Here is a tutorial video for navigating through the command prompt: [Command Prompt Tutorial](https://www.youtube.com/watch?v=A3nwRCV-bTU)

We recommend detaching from and stopping your Docker Container, but not deleting it. You can then restart and reattach to it next time you need it, and will not have to re-install everything.
{% endhint %}

{% hint style="danger" %}
If you have a newer Mac computer that runs with an Apple Silicon chip, you may encounter errors. Here are a few tips to avoid them:

* Make sure you have Mac OS 11 or above
* Install any minor updates to the operating system
* Install Rosetta 2 for Mac&#x20;
  * In terminal type `softwareupdate --install-rosetta`
* Make sure you've installed the Docker version that matches with the chip your Mac has (Intel vs Apple Silicon).
* Update Docker to the latest version
  * On Mac, updating Docker may require you to uninstall Docker before installing a newer version. To do this, open the Docker Desktop application and click the Troubleshoot icon (the small icon that looks like an insect at the top right corner of the window). Click the Uninstall button. Once this process is completed, open Applications in Finder and move Docker to the Trash. If you get an error message that says Docker cannot be deleted because it is open, then open Activity Monitor and stop all Docker processes. Then put Docker in the Trash. Once Docker is deleted, install the new Docker version appropriate for your Mac chip. After reinstallation is complete, restart your computer.
{% endhint %}



<details>

<summary>Getting errors?</summary>

#### When initially running the Docker image

:warning: _The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm64/v8) and no specific platform was requested_

#### When installing R packages

#### When installing Python package gempyor

#### When running flepimop or gempyor&#x20;

:warning: MADV\_DONTNEED does not work (memset will be used instead) : (This is the expected behaviour if you are running under QEMU)



</details>

## Setup Docker

See the [Before any run](before-any-run.md) section to ensure you have access to the correct files needed to run. On your local machine, determine the file paths to:

* the directory containing the flepimop code (likely the folder you cloned from Github), which we'll call `<dir1>`
* the directory containing your project code including input configuration file and population structure (again likely from Github), which we'll call `<dir2>`

{% hint style="info" %}
For example, if you clone your Github repositories into a local folder called Github and are using the flepimop\_sample as a project repository, your directory names could be\
\
_**On Mac:**_&#x20;

\<dir1> = /Users/YourName/Github/flepiMoP

\<dir2> = /Users/YourName/Github/flepimop\_sample\
\
_**On Windows:**_ \
\<dir1> = C:\Users\YourName\Github\flepiMoP

\<dir2> = C:\Users\YourName\Github\flepimop\_sample\


(hint: if you navigate to a directory like `C:\Users\YourName\Github` using `cd C:\Users\YourName\Github`, modify the above `<dir1>` paths to be `.\flepiMoP` and `.\flepimop_sample)`\


Note that Docker file and directory names are case sensitive
{% endhint %}

### **Run Docker image**

{% hint style="info" %}
Current Docker image: `/hopkinsidd/flepimop:latest-dev`
{% endhint %}

Docker is a software platform that allows you to build, test, and deploy applications quickly. Docker packages software into standardized units called **containers** that have everything the software needs to run including libraries, system tools, code, and runtime. This means you can run and install software without installing the dependencies in the system.

A docker container is an environment which is isolated from the rest of the operating system i.e. you can create files, programs, delete and everything but that will not affect your OS. It is a local virtual OS within your OS.&#x20;

For flepiMoP, we have a docker container that will help you get running quickly!&#x20;

First, make sure you have the latest version of the flepimop Docker (`hopkinsidd/flepimop)` downloaded on your machine by opening your terminal application and entering:

```
docker pull hopkinsidd/flepimop:latest-dev
```

Next, run the Docker image by entering the following, replace `<dir1>` and `<dir2>` with the path names for your machine (no quotes or brackets, just the path text).

In Mac, Windows and Linux, run Docker as follows:

```
docker run -it \
  -v <dir1>:/home/app/flepimop \
  -v <dir2>:/home/app/drp \
hopkinsidd/flepimop:latest-dev
```

{% hint style="danger" %}
On windows, make sure the path to dir1 and dir2 are written using forward slashes `/` instead of backslashes \\. Make sure there is no trailing slashes at the end.
{% endhint %}

In this command, we run the Docker container, creating a volume and mounting (`-v`) your code and project directories into the container. Creating a volume and mounting it to a container basically allocates space in Docker for it to mirror - and have read and write access - to files on your local machine.&#x20;

The folder with the flepiMoP code `<dir2>` will be on the path `flepimop` within the Docker environment, while the project folder will be at the path `drp.`&#x20;

{% hint style="success" %}
You now have a local Docker container installed, which includes the R and Python versions required to run flepiMop with all the required packagers already installed!&#x20;
{% endhint %}

{% hint style="info" %}
You don't need to re-run the above steps every time you want to run the model. When you're done using Docker for the day, you can simply "detach" from the container and pause it, without deleting it from your machine. Then you can re-attach to it when you next want to run the model.&#x20;
{% endhint %}

### Define environment variables

Create environmental variables for the paths to the flepimop code folder and the project folder:

```bash
export FLEPI_PATH=/home/app/flepimop/
export DATA_PATH=/home/app/drp/
```

Go into the code directory and do the installation the R and Python code packages

```bash
cd $FLEPI_PATH # move to the flepimop directory
Rscript build/local_install.R # Install R packages
pip install --no-deps -e flepimop/gempyor_pkg/ # Install Python package gempyor
```

Each installation step may take a few minutes to run.

{% hint style="info" %}
Note: These installations take place in the Docker container and not the local operating system. They must be made once while starting the container and need not be done for every time you run a model, provided they have been installed once. You will need an active internet connection for pulling the Docker image and installing the R packages (since some are hosted online), but not for other steps of running the model
{% endhint %}

## Run the code

Everything is now ready 🎉  The next step depends on what sort of simulation you want to run: One that includes inference (fitting model to data) or only a forward simulation (non-inference). Inference is run from R, while forward-only simulations are run directly from the Python package gempyor.

In either case, navigate to the project folder and make sure to delete any old model output files that are there

```bash
cd $DATA_PATH       # goes to your project repository
rm -r model_output/ # delete the outputs of past run if there are
```

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

### Non inference run

Stay in the `$DATA_PATH` folder, and run a simulation directly from forward-simulation Python package `gempyor.`&#x20;

To run an SIR-style model only (config does not need to have `outcomes` section), call `gempyor-seir` providing the name of the configuration file you want to run (ex. `config.yml`)&#x20;

```
gempyor-seir -c config.yml
```

{% hint style="warning" %}
It is currently required that all configuration files have an `interventions` section. There is currently no way to simulate a model with no interventions, though this functionality is expected soon. For now, simply create an intervention that has value zero.&#x20;
{% endhint %}

To run an SIR + observational model (configuration file has `seir` and `outcomes` section)

```
gempyor-outcomes -c config.yml
```

## Summary

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
gempyor-seir -c config.yml
</code></pre>
