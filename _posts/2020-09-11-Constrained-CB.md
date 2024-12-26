---
layout: post
title: "Constrained Contextual Bandits for Personalized Recommendation"
date: 2020-09-11
---
<span class="dropcap">C</span>ontextual bandits (CB), unlike the full Reinforcement Learning problem in which there are environmental states, new states depend on previous actions, and rewards can be delayed over time,  _"contexts”_  under the CB context do not mean the environment that the agent continuously interacts with, but the tuple of *(user features, item features, environment)*. Additionally, the rewards will not be delayed –- immediately we would know the user’s response such as purchasing or clicks.
<figure>
    <img src="{{ '/assets/img/20200911_CB.png' | prepend: site.baseurl }}" alt="">
</figure>

### Unconstrained Contextual Bandits
Let's start with the unconstrained case where the resource and time budget is infinity. It’s generally believed that imposing exploration to some degree produces a greater total reward in the long run. **`\(\epsilon\)`-greedy** is one of exploration approaches. The idea of `\(\epsilon\)`-greedy is quite simple: every time it selects the arm which produces the maximum reward as of now with probability 1-`\(\epsilon\)` while leaving the probability of `\(\epsilon\)` to explore, i.e. selecting an arm at random. However it has no preference for those that are nearly greedy or particularly uncertain.
*"It would be better to select among the non-greedy actions according to the potential for actually being optimal, taking into account both how close their estimates are to being maximal and the uncertainties in those estimates."* [1]

By the way, the `\(\epsilon\)`-greedy approach should technically not be categorized into "contextual bandits" but the "multi-arm bandits" because no contextual data is utilized to make the decision.
Besides `\(\epsilon\)`-greedy there are many other unconstrained contextual bandits algorithms such as the **LinUCB** [2], which usually serves as the baseline method in many CB papers. Also, it's worth to mention the **Thompson Sampling** [3] based CB. Both of them assumes the payoffs are linear.

### Constrained Contextual Bandits
Although many bandit algorithms mentioned above do not consider constraints in the original papers, adding the budget constraints (total or proportional) is not hard.
- One naive way is to always choose the best arm in each round until the remaining budget is zero. However, the claimed asymptotic regret bound might not be achieved anymore if we choose to add the constraints in this way.
- Another strategy is to choose the best arm in each round with the probability `\(\rho = (b_\tau / \tau)\)`.
- There is a more advanced resource allocation method such as the **UCB-ALP** [4], an adaptive dynamic linear programming (LP) method to solve for the constrained contextual bandit problems and use the UCB methods to estimate the expected rewards which are needed in the LP solution. We are interested in the asymptotic regime where the time-horizon `\(T\)` and the budget `\(B\)` grow to infinity proportionally, i.e., with a fixed ratio. Also it assumes finite discrete contexts (i.e. finite and fixed number of states in our language). Since they consider the fluctuations of the remaining budget, therefore, it’s named as *adaptive* linear programming. Another variant which extends the **UCB-ALP** approach to an infinite time horizon is the **HATCH** [5].

#### Reference
- [1] Sutton, Richard S., and Andrew G. Barto. _Reinforcement learning: An introduction_. MIT press, 2018.
- [2] Li, Lihong, et al. "A contextual-bandit approach to personalized news article recommendation." _Proceedings of the 19th international conference on World wide web_. 2010.
- [3] Agrawal, Shipra, and Navin Goyal. "Thompson sampling for contextual bandits with linear payoffs." _International Conference on Machine Learning_. 2013.
- [4] Huasen Wu, Rayadurgam Srikant, Xin Liu, and Chong Jiang. 2015. Algorithms with logarithmic or sublinear regret for constrained contextual bandits. In _Advances in Neural Information Processing Systems._ 433–441.
- [5] Yang, Mengyue, et al. "Hierarchical Adaptive Contextual Bandits for Resource Constraint-based Recommendation." _Proceedings of The Web Conference 2020_. 2020.
