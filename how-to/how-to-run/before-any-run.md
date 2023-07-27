# Before any run

In order to run any model with flepiMoP, you need access to two separate directories: One containing the flepiMoP code (**code** directory), and another containing the specific input files to run the model of your choosing (and to save the output from that model) (**project** directory). The [flepiMoP code](https://github.com/HopkinsIDD/flepiMoP) is available in a public repository on Github which can be pulled locally to serve as the code directory. We highly recommend also using Github to create your project directory; see [flepimop\_sample](https://github.com/HopkinsIDD/flepimop\_sample) for an example. If you need to create your own project repository from scratch, see instructions [below](before-any-run.md#create-a-project-repository).

For both the project repository and flepiMoP code repository, make sure you're on the correct branch, then pull updates from Github. Take note of the local file path to each directory.

### Create a project repository

Create a repository for your project on Github, naming it _something other than_ flepimop. This repository will eventually contain the configuration file specifying your model, any other input files or data, and will be where the model output will be stored. These files are always kept completely separately from the universal flepimop code that runs the model.&#x20;

{% hint style="info" %}
How to create a repository on Github: [https://docs.github.com/en/get-started/quickstart/create-a-repo](https://docs.github.com/en/get-started/quickstart/create-a-repo)
{% endhint %}

### Populate the repository

#### Add config file

Put your model configuration file(s) directly in this repository

#### Create a data folder

This data folder (which can have any name, but for simplicity can just be called **data**), should contain your file specifying the [population structure, mobility files](../../model-of-disease-transmission-+-observation/model-implementation/specifying-population-structure.md), and [seeding/initial conditions](../../model-of-disease-transmission-+-observation/model-implementation/specifying-initial-conditions-and-seeding.md).&#x20;

### Deciding how to run

(Fill in with pros and cons of different methods, which users they are best for)

##
