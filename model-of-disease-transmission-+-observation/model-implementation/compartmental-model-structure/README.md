# Specifying compartmental model structure

We want to allow users to work with a wide variety of infectious diseases or, in our case, one infectious disease under a wide variety of modeling assumptions. To facilitate this, we allow the user to specify their compartmental differential equations model via the configuration file.

We originally considered asking users to specify each compartment and transition manually. However, we quickly found that created long confusing configuration files, and created a shorthand to more tersely specify both compartments and transitions between them.

### Specifying model compartments (`compartments`)

The first stage of specifying the model is to define the infection states (variables) that the model will track. These "compartments" are defined first in the `compartments` section of the config file, before describing the processes that lead to transitions between them. &#x20;

For simple disease models, the compartments can simply be listed with whatever notation the user chooses. For example, for a simple SIR model, the compartments could be `["S", "I", "R"]`. The config also requires that there be a variable name for the property of the individual that these compartments describe, which for example in this case could be `infection_state`

```
compartments:
  infection_state: ["S", "I", "R"]
```

Our syntax allows for more complex models to be specified without much additional notation. For example, consider a model of a disease that followed SIR dynamics but for which individuals could receive vaccination, which might change how they experience infection.&#x20;

In this case we can specify compartments as the cross product of multiple states of interest. For example:

```
 compartments:
   infection_state: ["S", "I", "R"]
   vaccination_status: ["unvaccinated", "vaccinated"]
```

Corresponds to 6 compartments, which the code internally converts to this data frame

```
infection_state, vaccination_status, compartment_name
S,               unvaccinated,       S_unvaccinated
I,               unvaccinated,       I_unvaccinated
R,               unvaccinated,       R_unvaccinated
S,               vaccinated,         S_vaccinated
I,               vaccinated,         I_vaccinated
R,               vaccinated,         R_vaccinated
```

In order to more easily describe transitions, we want to be able to refer to a compartment by its components, but then use it by its compartment name.

If the user wants to specify a model in which some compartments are repeated across states but others are not, there will be pros and cons of how the model is specified. Specifying it using the cross product notation is simpler, less error prone, and makes config files easier to read, and there is no issue with having compartments that have zero individuals in them throughout the model. However, for very large models, extra compartments increase the memory required to conduct the simulation, and so having unnecessary compartments tracked may not be desired.&#x20;

For example, consider a model of a disease that follows SI dynamics in two separate age groups (children and adults), but for which only adults receive vaccination, with one or two doses of one of two vaccine types. With the simplified notation, this model could be specified as  ..

```
 compartments:
   infection_state: ["S", "I"]
   age_group: ["child","adult"]
   vaccination_status: ["unvaccinated", "1dose", "2dose"]
   vaccine_type: ["brand1", "brand2"]
```

corresponding to 12 compartments, 4 of which are unnecessary to the model

```
infection_state, age_group, vaccination_status, compartment_name
S,		 child,	    unvaccinated,	S_child_unvaccinated	
I,		 child,	    unvaccinated,	I_child_unvaccinated
S,		 adult,	    unvaccinated,	S_adult_unvaccinated
I,		 adult,	    unvaccinated,	I_adult_unvaccinated
S,		 child,	    1dose,		S_child_1dose
I,		 child,	    1dose,		I_child_1dose
S,		 adult,     1dose,		S_adult_1dose
I,		 adult,     1dose,		I_adult_1dose
S,		 child,     2dose,		S_child_2dose	
I,		 child,     2dose,		I_child_2dose
S,		 adult,	    2dose,		S_adult_2dose
I,		 adult,	    2dose,		I_adult_2dose
```

Or, it could be specified with the less concise notation

```
compartments:
   overall_date: ["S_child","I_child","S_adult_unvaccinated","I_adult_unvaccinated","S_adult_1dose","I_adult_1dose","S_adult_2dose","I_adult_2dose"]
```

which does not result in any unnecessary compartments being included.&#x20;

These compartments are referenced in multiple different subsequent sections of the config. In the `seeding (LINK TBA)` section the user can specify how the initial (or later imported) infections are distributed across compartments; in the [`seir`](./#transitions-seir-transitions) section the user can specify the form and rate of the transitions between these compartments encoded by the model; in the [`outcomes`](../specifying-observational-model/outcomes-module/outcomes-for-compartments.md) section the user can specify how the observed variables are generated from the underlying model states.

Notation must be consistent between these sections.&#x20;

### Specifying compartmental model transitions (`seir::transitions`)



The way we specify transitions is a bit more complicated than compartments. We specify one or more transition globs, each of which corresponds to one or more transitions. Since transition globs are shorthand for collections of transitions, we describe a single transition before discussing transition globs. A transition has 5 pieces associated of information

* source
* destination
* rate
* proportional\_to
* proportion\_exponent

#### Source

The compartment the transition moves people out of as an array

```
[S,unvaccinated]
```

#### Destination

The compartment the transition moves people into as an array

```
[I,unvaccinated]
```

#### Rate

[Joseph Lemaitre](https://app.gitbook.com/u/vgUi3iqQciVpkdEPuV1EGMpOcIr1 "mention")I'm bad at explaining what rate is

```
0.3
```

#### Proportional To

A vector of groups of compartments (each of which is an array) corresponding to which compartments the number of people moving is proportional to. Each group of compartments are combined before multiplying to obtain the proportion.

```
[[[S,unvaccinated]], [[I,unvaccinated],[I, vaccinated]]]
```

This one is a bit complicated, so let's unpack what it says. First, let's write compartment names as strings instead of as lists:

```
[[S_unvaccinated], [I_unvaccinated, I_vaccinated]]
```

From here, we can say that the transition we are describing is proportional to `S_unvaccinated` and `I_unvaccinated + I_vaccinated`.

#### Proportion Exponent

An exponent for each group of compartments in the proportional to section.

```
[1, 0.9]
```

#### Summary

Putting it all together, the transition

```
source: [S, unvaccinated]
destination: [I, unvaccinated]
proportional_to: [[[S,unvaccinated]], [[I,unvaccinated],[I,vaccinated]]]
rate: [0.3]
proportion_exponent: [1,0.9]
```

corresponds to the following math

$$
\frac{\delta (\text{S},\text{unvaccinated})}{\delta t} = - 0.3 \times(\text{S},\text{unvaccinated})^1 \times ((\text{I},\text{unvaccinated}) +(\text{I},\text{vaccinated}))^{0.9}
$$

$$
\frac{\delta (\text{I},\text{unvaccinated})}{\delta t} = +0.3 \times(\text{S},\text{unvaccinated})^1 \times ((\text{I},\text{unvaccinated}) +(\text{I},\text{vaccinated}))^{0.9}
$$

### Transition Globs

The basic idea for a transition is to take each compartment component, and allow one or more components to be specified instead. It creates transitions by broadcasting across components. To assist in this, we allow multiple rate values to be specified across different components, and then reduce to a single rate by multiplication. Again, let's go through an example.

For transition globs, any time you could specify multiple arguments as a list, you may instead specify one argument as a non-list, which will be used for every broadcast. So \[1,1,1] is equivalent to 1 if the dimension of that broadcast is 3.

#### Source

We allow one or more arguments to be specified for each compartment. So to specify both susceptible compartments we would use

```
[[S],[unvaccinated,vaccinated]]
```

#### Destination

The destination should be the same shape as the source, and in the same relative order. So to specify a transition from (S,unvaccinated) to (I,unvaccinated) and (S,vaccinated) to (I,vaccinated), we would write

```
[[I],[unvaccinated,vaccinated]]
```

If instead we wrote:

```
[[I],[vaccinated,unvaccinated]]
```

we would have a transition from (S,unvaccinated) to (I,vaccinated) and (S,vaccinated) to (I,unvaccinated).

#### Rate

Since there are two changes to x a time. The first change we make, is to allow specifying a rate for each component, which are multiplied together. So,

```
[0.1, 3]
```

would correspond to an rate of

```
[0.1 * 3]
```

By itself, this is useless, but we also allow broadcasting of each component:

```
[[0.1], [3,0.5]]
```

This would mean our transition from (S,unvaccinated) to (I,unvaccinated) would have a rate of `0.1 * 3` while our transition from (S,vaccinated) to (I,vaccinated) would have a rate of `0.1 * 0.5`.

#### Proportional To

The broadcasting here is a bit more complicated. In other cases, each broadcast is over a single component. However, in this case, we have a broadcast over a group of components. We allow a different group to be chosen for each broadcast.

```
[
  [[S,unvaccinated], [S,vaccinated]],
  [[I,unvaccinated],[I, vaccinated]], [[I,unvaccinated],[I, vaccinated]]
]
```

Again, let's unpack what it says. Since the broadcast is over groups, let's split the config back up

into those groups

```
[
  [S,unvaccinated],
  [[I,unvaccinated],[I, vaccinated]]
]
[
  [S,vaccinated],
  [[I,unvaccinated],[I, vaccinated]]
]
```

From here, we can say that we are describing two transitions. Both are proportional to the same compartments (S,unvaccinated) and all of I.

If, for example, we would not expect contact between unvaccinated infected and vaccinated susceptibles, we would instead write:

```
[
  [[S,unvaccinated], [S,vaccinated]],
  [[I,unvaccinated],[I, vaccinated]], [[I, vaccinated]]
]
```

#### Proportion Exponent

Similarly to rate and proportional to, we provide an exponent for component and every group across the broadcast. So we could for example use:

```
[[1,1], [0.9,0.8]]
```

#### Summary

Putting it all together, the transition glob

```
source: [[S],[unvaccinated,vaccinated]]
destination: [[I],[unvaccinated,vaccinated]]
proportional_to: [
                   [[S,unvaccinated], [S,vaccinated]],
                   [[I,unvaccinated],[I, vaccinated]], [[I, vaccinated]]
                 ]
rate: [[0.1], [3,0.5]]
proportion_exponent: [[1,1], [0.9,0.8]]
```

corresponds to the following transitions

```
- source: [S,unvaccinated]
  destination: [I,unvaccinated]
  proportional_to: [[[S,unvaccinated]], [[I,unvaccinated],[I, vaccinated]]]
  proportion_exponent: [1 * 0.9]
  rate: [0.1*3]
- source: [S,vaccinated]
  destination: [I,vaccinated]
  proportional_to: [[[S,vaccinated]], [[I, vaccinated]]]
  proportion_exponent: [1 * 0.8]
  rate: [0.1*0.5]
```

#### Warning

We warn the user that it is possible to specify quite large models with not that many lines of config. The more compartments and transitions you specify, the longer the model will take to run, and the more memory it will require.
