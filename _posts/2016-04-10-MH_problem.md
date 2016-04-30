---
layout: post
title:  "Python Simulation Practice -- The Monty Hall Problem"
date:   2016-04-10
---

<span class="dropcap">R</span>ecently I'm following the Harvard [CS109](http://cs109.github.io/2015/) online course, which definitely is an awesome one in the data science topic. I came across the very interesting statistics probelm,  [Monty Hall Probelm](https://en.wikipedia.org/wiki/Monty_Hall_problem), in hw0 where we were trying to solve the problem via simulations. Let's review the problem at first:

"In a gameshow, contestants try to guess which of 3 closed doors contain a cash prize (goats are behind the other two doors). Of course, the odds of choosing the correct door are 1 in 3. As a twist, the host of the show occasionally opens a door after a contestant makes his or her choice. This door is always one of the two the contestant did not pick, and is also always one of the goat doors (note that it is always possible to do this, since there are two goat doors). At this point, the contestant has the option of keeping his or her original choice, or swtiching to the other unopened door. The question is: is there any benefit to switching doors? The answer surprises many people who haven't heard the question before." 

The conclusion is that, yes, switching doors will increase the winning probability A LOT (nearly 2 times)! Does it sound counter-intuitive to you? Well, not anymore if you generalize into n (n is large) doors and the host shows you the remaining n-2 goat doors. If you still have doubts about it, let's see what the simulation results tell you. Also, I will consider it as a very good Python practice for myself.

<script src="https://gist.github.com/HongleiXie/e470fd620eb035265e9df14937c34e03.js"></script>

The experiment results are pretty clear now, I tested on 3 doors, which is the original case, the resulting winning probability increased from 31.8% to 69.2% after sitching. When there are more doors, for say, 10 doors, the switching door benefits become more significant, winning probability increases from 10.3% to 89.3%. If we set the doors to be 100, keeping original guess only has 1% chance to get the prize. However, after switching the figure can reach as high as 98.9%.
