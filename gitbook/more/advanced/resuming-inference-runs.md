# Resuming inference runs

## Communication Between Iterations

The pipeline uses files to communicate between different iterations. Currently, the following file types exist:

* seed
* ~~<mark style="background-color:red;">cont</mark>~~
* snpi
* spar
* seir
* hpar
* hnpi
* hosp
* llik

During each iteration, inference uses these files to communicate with the compartmental model and outcomes. The intent is that inference should only need to read and write these files, and that the compartmental model can handle everything else. In addition to the `global` versions of these files actually passed to the compartmental/reporting model, there exist `chimeric` versions used internally by inference and stored in memory. These copies are what inference interacts with when it needs to perturb values. While this design was chosen primarily to support modularity (a fixed communication boundary makes it easy to swap out the compartmental model), it has had a number of additional benefits.

### Bootstrapping

The first iteration of an MCMC algorithm is a special case, because we need to pull initial conditions for our parameters. We originally developed the model without inference in mind, so the compartmental model is already set up to read parameter distributions from the configuration file, and to draw values from those distributions, and record those parameters to file. We take advantage of this to bootstrap our MCMC parameters by running the model one time, and reading the parameters it generated from file.

#### Resume from previous run

We can, instead of bootstrapping our first iteration, read in final values of a previous iteration. This allows us to&#x20;
