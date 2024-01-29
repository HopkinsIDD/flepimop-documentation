# Before any run: organizing your folders

In order to run any model with flepiMoP, you need access to two separate directories: One containing the flepiMoP code (**code** directory), and another containing the specific input files to run the model of your choosing (and to save the output from that model) (**project** directory). The [flepiMoP code](https://github.com/HopkinsIDD/flepiMoP) is available in a public repository on Github which can be pulled locally to serve as the code directory. We highly recommend also using Github to create your project directory. To get familiar with the code, we recommend starting with our example configurations by making a fork of  [flepimop\_sample](https://github.com/HopkinsIDD/flepimop\_sample). If you need to create your own project repository from scratch, see instructions [below](before-any-run.md#create-a-project-repository).

For both the project repository and flepiMoP code repository, make sure you're on the correct branch, then pull updates from Github. Take note of the local file path to each directory.

{% hint style="info" %}
These directories can be located on your computer wherever you prefer, as you can tell flepiMoP where they are, but we recommend you clone these **flat**, e.g.

```
parentdirectory
├── flepiMoP
├── flepimop_sample
└── project_directory_1
```
{% endhint %}

### 📂 Create a project repository from `flepimop_sample`&#x20;

In order to create a project repository from the [flepimop\_sample](https://github.com/HopkinsIDD/flepimop\_sample) repository follow these steps:

1. Fork the  [flepimop\_sample](https://github.com/HopkinsIDD/flepimop\_sample) repository to your desired github giving it a repository name. Instructions for forking a repository are available [here](https://docs.github.com/en/get-started/quickstart/fork-a-repo).
2. Clone the repository locally using the command:\
   `git clone <my repository location>`
3.  Now creating a code directory by cloning the [flepiMoP code](https://github.com/HopkinsIDD/flepiMoP) repository in the directory where you cloned your fork of `flepimop_sample`:

    `git clone https://github.com/HopkinsIDD/flepiMoP`

Now you are ready to run flepiMoP using your preferred method (see below).

### 📂 Create a project repository from scratch

Create a repository for your project on Github, naming it _something other than_ flepiMoP. This repository will eventually contain the configuration file specifying your model, any other input files or data, and will be where the model output will be stored. These files are always kept completely separately from the universal flepimop code that runs the model.&#x20;

{% hint style="info" %}
How to create a repository on Github: [https://docs.github.com/en/get-started/quickstart/create-a-repo](https://docs.github.com/en/get-started/quickstart/create-a-repo)
{% endhint %}

### Populate the repository

#### Add config file

Put your model configuration file(s) directly in this repository.

#### Create a data folder

This data folder (which can have any name, but for simplicity can just be called **data**), should contain your file specifying the [population structure, mobility files](../gempyor/model-implementation/specifying-population-structure.md), and [seeding/initial conditions](../gempyor/model-implementation/specifying-initial-conditions-and-seeding.md).&#x20;

## 🏃🏽‍♀️ Running the code

{% hint style="warning" %}
If you have any trouble or questions while trying to run flepimop, please report them on the [GitHub Q\&A](https://github.com/HopkinsIDD/flepiMoP/discussions/categories/q-a).
{% endhint %}

### Deciding how to run

The code is written in a combination of [R](https://www.r-project.org/) and [Python](https://www.python.org/). The Python part of the model is a package called [_gempyor_](../gempyor/model-description.md), and includes all the code to simulate the epidemic model and the observational model and apply time-dependent interventions. The R component conducts the (optional) parameter inference, and all the (optional) provided pre and post processing scripts are also written in R. Most uses of the code require interacting with components written in both languages, and thus making sure that both are installed along with a set of required packages. However, Python alone can be used to do forward simulations of the model using _gempyor_.&#x20;

Because of the need for multiple software packages and dependencies, we describe different ways you can run the model, depending on the requirements of your model setup.  See [Quick Start Guide](quick-start-guide.md) for a quick introduction to using gempyor and flepiMoP. We also provide some more [advanced](advanced-run-guides/) ways to run our model, particularly for longer chains.&#x20;

