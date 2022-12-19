---
description: Testing the COVID Scenario Pipeline
---

# Running with docker locally

**1. Checkout projects from GitHub:**

We need to clone the CovidScenarioPipeline from Git Hub. Make sure to clone the ‘main\_testing’ branch. The link is as follows: [Co](https://github.com/HopkinsIDD/COVIDScenarioPipeline)[vid Scenario Pipeline](https://github.com/HopkinsIDD/COVIDScenarioPipeline)

**2. Run Docker Image**

Docker is a software platform that allows you to build, test, and deploy applications quickly. Docker packages software into standardized units called containers that have everything the software needs to run including libraries, system tools, code, and runtime. A docker container is an environment which is isolated from the rest of the operating system i.e. you can create files, programs, delete and everything but that will not affect your OS. It is a local virtual OS within your OS. One installs all the packages and dependencies in the said container and deploys their application in the container. This is the reason why we install various Python and R dependencies, every time we run a docker container.

Docker with respect to the Covid Scenario Pipeline enables you to run and install software without installing the dependencies in the system. You instead install the required files to the docker container. To understand the basics of docker refer to the following: [Docker Basics](https://www.docker.com/)

To install docker for windows refer to the following link: [Installing Docker](https://docs.docker.com/desktop/windows/install/)

The following is a good tutorial for introduction to docker: [Docker Tutorial](https://www.youtube.com/watch?v=gFjxB0Jn8Wo\&list=PL6gx4Cwl9DGBkvpSIgwchk0glHLz7CQ-7)

To run the entire pipeline we use the command prompt. To open the command prompt type “Command Prompt" in the search bar and open the command prompt. Here is a tutorial video for navigating through the command prompt: [Command Prompt Tutorial](https://www.youtube.com/watch?v=A3nwRCV-bTU)

To run the pipeline, we refer to two parts simultaneously – the data folder which has the data with respect to the area being considered like the seeding data and mobility data. The second part is the CovidScenarioPipeline folder which utilizes the data and processes it.

To test, we use the test folder (test\_documentation\_inference\_us in this case) in the CovidScenariPipeline as the data repository folder. We run the docker container and set the paths.

> `docker run -it -v <CD1>\:/home/app/csp -v <CD2>:/home/app/drp hopkinsidd/covidscenariopipeline:latest-dev`

In this command we run the docker image `hopkinsidd/covidscenariopipeline`. The `-v` command is used to allocate space from Docker and mount it at the given location. We create storage spaces for both the data folder and the CovidScenarioPipeline folder and mount it to a path called `drp` and `csp` within the docker app respectively.

You need to change `<CD1>` to the path of the CovidScenarioPipeline folder and `<CD2>` to the path of the test\_documentation\_inferecene\_us folder present in the path ..\COVIDScenarioPipeline\test\test\_documentation\_inference\_us

**3. Creating Environment Variables**

We must create 3 environment variables namely: `CENSUS_API_KEY`, `COVID_PATH` and `DATA_PATH`.

The Census Data Application Programming Interface (API) is an API that gives the public access to raw statistical data from various Census Bureau data programs. To create the CENSUS\_API\_KEY variable, we run the following command.

> `export CENSUS_API_KEY=<CD3>`

You need to copy your API Key in place of `<CD3>`. To acquire the API Key, click on [here](https://api.census.gov/data/key\_signup.html)

After you enter your details, you should receive an email using which you can activate your key and then use it.

_Note: Do not enter the API Key in quotes, copy the key as it is._

We then create the `COVID_PATH` and `DATA_PATH` environment variables, to which we assign values of the `csp` and the `drp` paths within the docker app. We run the following commands to do so.

> `export COVID_PATH=/home/app/csp/`

> `export DATA_PATH=/home/app/drp/`

**4. The docker container needs certain local R packages and Python Requirements to be installed:**

Change the directory to the Covid Path:

> `cd $COVID_PATH`

Run the required files to complete the installation process:

1. Installing relevant R files:

> `Rscript local_install.R`

1. We then need to install the gempyor package. To know about the gempyor package read the following document: ..\COVIDScenarioPipeline\gempyor\_pkg\docs\Rinterface.html

> `pip install gempyor_pkg/`

_Note: These installations take place in the docker container and not the Operating System. They must be made once while starting the container and need not be done for every time you are running tests, provided they have been installed once._

**5. We then run the corresponding Rscripts to run the test:**

After we have done the necessary setup we then need to run the relevant Rscripts. We first need to change the working directory to the `DATA_PATH`.

To do so we run the command.

> `cd $DATA_PATH`

To run the test on the given configuration file, we initially need to create the population seeding file(geodata) and the population mobility file (mobility). To do so we run the following command.

> `Rscript $COVID_PATH/R/scripts/build_US_setup.R -c config.yml`

This should run without any error. To check if the script did run successfully. Go to the test\_documentation\_inference\_us folder. You will now see a new folder called ‘data’ present in it. In the data folder there should be 2 csv sheets named: ‘geodata.csv’ and ‘mobility.csv’

After you have run the following command:

> `Rscript $COVID_PATH/R/scripts/full_filter.R -c config.yml -p $COVID_PATH -j 1 -b 1 -k 1`

This should run without any error. Following should be the last few lines visible on the command prompt:

> \[\[1]]
>
> \[\[1]]\[\[1]]
>
> \[\[1]]\[\[1]]\[\[1]]
>
> NULL

To check if the script did run successfully. Go to the test\_documentation\_inference\_us folder. A folder called model\_output should be created. If that folder exists then the pipeline is functional and runs succesfully.
