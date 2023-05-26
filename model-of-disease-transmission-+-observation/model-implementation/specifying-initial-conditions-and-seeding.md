---
description: >-
  This section describes how to specify the values of each model state at the
  time the simulation starts, and how to make instantaneous changes to state
  values at other times (e.g., due to importations)
---

# Specifying initial conditions and seeding

## Overview

In order for the models specified previously to be dynamically simulated, the user must provide _initial conditions,_ in addition to the model structure and parameter values. Initial conditions describe the value of each variable in the model at the time point that the simulation is to start. For example, on day zero of an outbreak, we may assume that the entire population is susceptible except for one single infected individual. Alternatively, we could assume that some portion of the population already has prior immunity due to vaccination or previous infection. Different initial conditions lead to different model trajectories.&#x20;

FlepiMoP also allows users to specify instantaneous changes in values of model variables, at later times in the simulation. We call this "seeding". For example, some individuals in the population may travel or otherwise acquired infection from outside the population throughout the epidemic, and this important of infection could be specified with the seeding option. As another example, new genetic variants of the pathogen may arise due to mutation and selection that occurs within infected individuals, and this generation of new strains can also be modeled with seeding. Seeding allows individuals to change state at specified times in ways that do not depend on the model equations. In the first example, the individuals would be "seeded" into the infected compartment from the susceptible compartment, and in the second example, individuals would be seeded into the "infected with new variant" compartment from the "infected with wild type" compartment.

The seeding option can also be used as a convenient alternative way to specify initial conditions. By default, FlepiMoP initiates models by putting the entire population size (specified in the geodata file) in the first model compartment. If the desired initial condition is only slightly different than the default state, it may be more convenient to specify it with a few "seedings" that occur on the first day of the simulation. For example, for a simple SIR model where the desired initial condition is just a small number of infected individuals, this could be specified by a single seeding into the infected compartment from the susceptible compartment at time zero, instead of specifying the initial values of three separate compartments. For larger models, the difference becomes more relevant.&#x20;

The `seeding` and `initial_conditions` section of the configuration file are detailed below.&#x20;

This table is useful for a quick comparison of these sections

|                                       | initial\_conditions                                                                                                                              | seeding                                                                                                                                                                                                          |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Config section optional or required?  | Optional                                                                                                                                         | Optional                                                                                                                                                                                                         |
| Function of section                   | Specify number of individuals in each compartment at time zero                                                                                   | Allow for instantaneous changes in individuals' states                                                                                                                                                           |
| Default                               | Entire population in first compartment, zero in all other compartments                                                                           | No seeding events. Entire population in first compartment, zero in all other compartments                                                                                                                        |
| Requires input file?                  | Yes, .csv                                                                                                                                        | Yes, .csv                                                                                                                                                                                                        |
| Input description                     | Input is a list of compartment names, location names, and amounts of individuals in that compartment-location. All compartments must be listed.  | Input is list of seeding events defined by source compartment, destination compartment, number of individuals transitioning, and date of movement. Compartments without seeding events don't need to be listed.  |
| Specifies an incidence or prevalence? | Amounts specified are prevalence values                                                                                                          | Amounts specified are instantaneous incidence values                                                                                                                                                             |
| Useful for?                           | Specifying initial conditions, especially if simulation does not start with a single infection introduced into a naive population.               | Modeling importations, evolution of new strains, and specifying initial conditions                                                                                                                               |



what happens if both config sections missing?

what happens if all initial conditions dont add up to population size?

IC - prevalence or incidence??



## Specifying model seeding





## Specifying model initial conditions

