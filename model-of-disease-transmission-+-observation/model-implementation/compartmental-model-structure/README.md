# Compartmental model structure

We want to allow users to work with a wide variety of infectious diseases or, in our case, one infectious disease under a wide variety of modeling assumptions. To facilitate this, we allow the user to specify their compartmental differential equations model via configuration file.

We originally considered asking users to specify each compartment and transition manually. However, we quickly found that created long confusing configuration files, and created a shorthand to more tersely specify both compartments and transitions between them.

### Compartments

We specify compartments as the cross product of states of interest. For example:

```
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

### Transitions

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

This one is a bit complicated, so let's unpack what it says.  First, let's write compartment names as strings instead of as lists:

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
proportion_exponent: [0,0.9]
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

For transition globs, any time you could specify multiple arguments as a list, you may instead specify one argument as a non-list, which will be used for every broadcast.  So \[1,1,1] is equivalent to 1 if the dimension of that broadcast is 3.

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

Since there are two changes to rate, we'll do them one at a time. The first change we make, is to allow specifying a rate for each component, which are multiplied together. So,

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

Again, let's unpack what it says.  Since the broadcast is over groups, let's split the config back up&#x20;

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

