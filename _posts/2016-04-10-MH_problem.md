---
layout: post
title:  "Python Simulation Practice -- The Monty Hall Problem"
date:   2016-04-10
---

<span class="dropcap">R</span>ecently I'm following the Harvard [CS109](http://cs109.github.io/2015/) online course, which definitely is an awesome one among many data science MOOCs. I came across the very interesting statistics problem,  [Monty Hall Probelm](https://en.wikipedia.org/wiki/Monty_Hall_problem), in hw0 where we were trying to solve the problem via simulations. Let's review the problem at first:

"In a gameshow, contestants try to guess which of 3 closed doors contain a cash prize (goats are behind the other two doors). Of course, the odds of choosing the correct door are 1 in 3. As a twist, the host of the show occasionally opens a door after a contestant makes his or her choice. This door is always one of the two the contestant did not pick, and is also always one of the goat doors (note that it is always possible to do this, since there are two goat doors). At this point, the contestant has the option of keeping his or her original choice, or swtiching to the other unopened door. The question is: is there any benefit to switching doors? The answer surprises many people who haven't heard the question before."

The conclusion is that, yes, switching doors will increase the winning probability A LOT (nearly 2 times)! Does it sound counter-intuitive to you? Well, not anymore if you generalize into n (n is large) doors and the host shows you the remaining n-2 goat doors. If you still have doubts about it, let's see what the simulation results tell you. Also, I will consider it as a very good Python practice for myself. See the implementation codes below.

- In the first step, let's import some packages and set up parameters. For example, I'm simulating 10 doors, 1000 trials.

```python
# -*- coding: utf-8 -*-

import numpy as np # imports a fast numerical programming library
import scipy as sp #imports stats functions, amongst other things
import matplotlib as mpl # this actually imports matplotlib
import matplotlib.cm as cm #allows us easy access to colormaps
import matplotlib.pyplot as plt #sets up plotting under plt
import pandas as pd #lets us handle data as dataframes
#sets up pandas table display
pd.set_option('display.width', 500)
pd.set_option('display.max_columns', 100)
pd.set_option('display.notebook_repr_html', True)
import seaborn as sns #sets up styles and gives us more plotting options

#Set Parameters
nsim = 1000 # simulation times
doors = 10 # number of dooors, must be >2 integers
```
- Next, write a few functions. `simulate_prizedoor` and `simulate_guess` are used to randomly assign a prize door and randomly guess a door respectively. `goat_door` simulates the opening of a "goat door" that doesn't contain the prize, and is different from the contestants guess. `switch_guess` applies the strategy that always switches a guess after the goat door is opened.


```python
def simulate_prizedoor(nsim):
    return np.random.randint(0, doors, (nsim))

def simulate_guess(nsim):
    return np.random.randint(0, doors, (nsim))

def goat_door(prizedoors, guesses):
   out = pd.DataFrame();
   while out.shape[1] != nsim:
    for t in range(0, nsim):
        if prizedoors[t] == guesses[t]:
            same_thing = prizedoors[t]
            out[t] = np.random.choice([x for x in range(0,doors) if x != same_thing], doors-2, replace = False)
        else:
            out[t] = [x for x in range(0,doors) if x != prizedoors[t] and x != guesses[t]]
    return out    

def switch_guess(guesses, goatdoors):
    result = pd.DataFrame()
    for t in range(0, nsim):
        result[t] = list(set(range(0,doors)) - set([guesses[t]]) - set(goatdoors[t]))
    return result
```   

  - Finally, compute the winning rates under the two strategies.  

```python
def win_percentage(guesses, prizedoors):
    return 100 * (guesses == prizedoors).mean()

#keep guesses
print "When there are", doors, "doors, win percentage when keeping original door is"
print  win_percentage(simulate_guess(nsim), simulate_prizedoor(nsim))

#switch
prize = simulate_prizedoor(nsim)
guess = simulate_guess(nsim)
goats = goat_door(prize, guess)

guess = switch_guess(guess, goats)
print "On the other hand, if switching the win percentage is"
print win_percentage(prize, guess).mean()
```

The experiment results are pretty clear now, I tested on 3 doors, which is the original case, the resulting winning probability increased from 31.8% to 69.2% after sitching. When there are more doors, for say, 10 doors, the switching door benefits become more significant, winning probability increases from 10.3% to 89.3%. If we set the doors to be 100, keeping original guess only has 1% chance to get the prize. However, after switching the figure can reach as high as 98.9%.
