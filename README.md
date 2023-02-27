# Home

Welcome to the FlepiMoP wiki!

The **Flexible Epidemic Modeling Pipeline** (“FlepiMoP”, formerly the COVID Scenario Pipeline, “CSP”) is an open-source software package designed by researchers in the [Johns Hopkins Infectious Disease Dynamics Group](http://www.iddynamics.jhsph.edu/) to simulate mathematical models of infectious disease dynamics.&#x20;

It was initially designed in early 2020 and was routinely used to provide projections of the emerging COVID-19 epidemic to different health authorities world-wide. Nowdays, the COVID-19 Scenario Pipeline provide its COVID-19 projections in the US to CDC-funded model aggregation sites, the [COVID-19 Forecast Hub](https://covid19forecasthub.org/) and the [COVID-19 Scenario Modeling Hub](https://covid19scenariomodelinghub.org/) and influenza projections to [FluSight ](https://www.cdc.gov/flu/weekly/flusight/index.html)and to the [Flu Scenario Modeling Hub](https://fluscenariomodelinghub.org).

However, the pipeline is much more general and can be used to simulate the dynamics of any infection that can be expressed as a [compartmental epidemic model](https://en.wikipedia.org/wiki/Compartmental\_models\_in\_epidemiology). These include applications in chemical reaction kinetics, pharmacokinetics, within-host disease dynamics, or applications in the social sciences.

In addition to producing forward simulations given a specified model and parameter values, the pipeline can also attempt to optimize unknown parameters (e.g. transmission rate, case detection rate, intervention  efficacy) to fit the model to datasets the user provides (e.g. hospitalizations due to severe di sease) using a Bayesian inference framework. This feature allows the pipeline to be utilized for short-term forecasting or longer-term scenario projections for ongoing epidemics, since it can simultaneously be fit to data for dates in the past and then use best-fit parameters to make projections into the future.&#x20;

{% hint style="warning" %}
* The current COVIDScenarioPipeline branch for run is: `main`
* The current docker version is: `latest-dev`
* The current COVID19\_USA branch is: `main`
{% endhint %}

### General description of the COVID Scenario Modeling Pipeline

Features:

* No-code configuration for any metapopulation compartmental model with time varying parameters and complex observation model
* Distributed inference engine ready for high-performance cluster or batch cloud
* Portable using docker or anaconda
* Open-source software

<figure><img src=".gitbook/assets/CSP Overview (1).png" alt=""><figcaption><p>Overview of the pipeline organization</p></figcaption></figure>

The mathematical model within the pipeline is a _compartmental epidemic model_ embedded within a _well-mixed metapopulation_. A compartmental epidemic model is a model that divides all individuals in a population into a discrete set of states (e.g. “infected”, “recovered”) and tracks - over time - the number of individuals in each state and the rates at which individuals transition between these states. The well-known SIR model is a classic example of such a model, and much more complex versions of this model type have been simulated with this framework (for example, an SEIR-style model in which individuals are further subdivided into multiple age groups and vaccination statuses).&#x20;

The structure of the desired model, as well as the parameter values and initial conditions, can be specified flexibly by the user in a nocode fashion. The pipeline allows for parameter values to change over time at discrete intervals, which can be used to specify time-dependent aspects of disease transmission and control (such as seasonality or vaccination campaigns).

The model is embedded within a meta-population structure, which consists of a series of distinct subpopulations (e.g. states, provinces, or other communities) in which the model structure is repeated, albeit with potentially different parameter values. The subpopulations can interact, either through the movement of individuals or the influence of individuals in one subpopulation on the transition rate of individuals in another.&#x20;

Within each subpopulation, the population is assumed to be well-mixed, meaning that interactions are assumed to be equally likely between any pair of individuals (since unique identities of individuals are not explicitly tracked). The same model structure can be simulated in a continuous-time deterministic or discrete-time stochastic manner.&#x20;

In addition to the variables described by the compartmental model, the model can track other observable variables (“outcomes”) that are functions of the basic model variables but do not themselves influence the dynamics (i.e. some portion of infections are reported as cases, depending on a testing rate). The model can be run iteratively to tune the values of certain parameters so that these outcome variables best match timeseries data provided by the user for a certain time period.&#x20;

Fitting is done using a Bayesian-like framework, where the user can specify the likelihood of observed outcomes in data given modeled outcomes, and the priors on any parameters to be fit. Multiple datastreams (e.g. cases and deaths) can be fit simultaneously. A custom Markov Chain Monte Carlo method is used to sequentially propose and accept or reject parameter values based on the model fit to data, in a way that balances fit quality within each individual subpopulation with that of the total aggregate population, and that takes advantage of parallel computing environments.

The code is written in a combination of [R](https://www.r-project.org/) and [Python](https://www.python.org/), and the vast majority of users only need to interact with the pipeline via the components written in R. It is structured in a modular fashion, such that individual components - such as the epidemic model, the observable variables, the population structure, or the parameters - can be edited or completely replaced without any handling of other parts of the code.&#x20;

When model simulation is combined with fitting to data, the code is designed to run most efficiently on a supercomputing cluster with many cores. We most commonly run the code on [Amazon Web Services](https://aws.amazon.com/) or on high-performance computers using SLURM. However, even relatively large models can be run efficiently on most personal computers. Typically, the memory of the machine will limit the number of compartments (i.e., variables) that can be included in the epidemic model, while the machine’s CPU will determine the speed at which each model run is completed and the number of iterations of the model that can be run during parameter searches when fitting the model to data. While the pipeline can be installed on any computer, it is sometime easier to use an Anaconda environment or the provided a [Docker](https://www.docker.com/) container, where all the software dependencies (e.g. standardized R and Python versions along with required packages) are included, independent of the user’s local machine. All the code is maintained on our [GitHub](https://github.com/HopkinsIDD/COVIDScenarioPipeline) and shared with the GNU General Public License v3.0 license. It is build on top of a fully open-source software stack.

This documentation is organized as follows. The [Model Description](model-of-disease-transmission-+-observation/model-description.md) section describes the mathematical framework for the compartmental epidemic models that can be simulated forward in time by the pipeline. The [Model Inference](model-inference/inference-description.md) section describes the statistical framework for fitting the model to data. The [Data and Parameter](broken-reference) section describes the inputs the user must provide to the pipeline, in terms of the model structure and parameters, the population characteristics, the initial conditions, time-varying interventions, data to be fit, and more. The [How to Run](how-to/how-to-run/) section provides concrete guidance on setting up and running the model and analyzing the output. The [Tutorials](how-to/tutorials.md) section provides some example model setups, from very basic to more complex. The [Advanced](broken-reference) section goes into more detail on specific features of the model and the code that are likely to only be of interest to users who want to run more complex models or data fitting routines or substantially edit the code. It includes the a subsection describing each file and package used in the pipeline and their interactions during a model run.

Users who wish to jump to running the model themselves can see [Quick Start Guide](broken-reference).

For questions about the pipeline or to report a bug, please use the “Issues” or "Discussions" feature on [our GitHub](https://github.com/HopkinsIDD/COVIDScenarioPipeline).
