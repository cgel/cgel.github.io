---
layout: post
title: Learning to search
excerpt_separator: <!--more-->
future: true
autoNumber: '"all"  '
---
{% assign ref_names = "" | split:"|"  %}
{% assign ref_urls = "" | split:"|"  %}
<!-- 1 -->
{% assign ref_names = ref_names | push:
  "Mastering the game of Go with deep neural networks and tree search"%}
{% assign ref_urls = ref_urls | push:
  "http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html" %}
<!-- 2 -->
{% assign ref_names = ref_names | push:
  "Bandit based monte-carlo planning" %}
{% assign ref_urls = ref_urls | push:
  "http://ggp.stanford.edu/readings/uct.pdf" %}
<!-- 3 -->
{% assign ref_names = ref_names | push:
  "Finite-time Analysis of the Multiarmed Bandit Problem" %}
{% assign ref_urls = ref_urls | push:
  "https://homes.di.unimi.it/~cesabian/Pubblicazioni/ml-02.pdf" %}
<!-- 4 -->
{% assign ref_names = ref_names | push:
  "The grand challenge of computer Go: Monte Carlo tree search and extensions" %}
{% assign ref_urls = ref_urls | push:
  "http://www0.cs.ucl.ac.uk/staff/d.silver/web/Publications_files/grand-challenge.pdf" %}

{% for i in (1..ref_names.size) %}
  {%assign j = i | minus: 1 %}
  [{{i}}]: {{ ref_urls[j] }} "{{ref_names[j] }}"
{% endfor %}

Undoubtedly, one of the greatest achievements of the field of AI has been AlphaGo [[1]]. Similarly to how people play the game, AlphaGo used a value network to evaluate the game position instead of relying on large numbers of rollouts. Since there is a lot of structure in Go, it is possible to learn patterns. This patterns can then be exploited to make more efficient algorithms.

Yet, there is still a stark difference between how AlphaGo and humans play: The way they search. People use their intelligence and experience to decide what combinations are worth evaluating, AlphaGo, instead, relies on a hard-coded <sup>1</sup> search strategy.

Here I propose a new way to learn to search trees based on UCT. I also prove that the learned algorithm is consistent.
<!--more-->
$$\newcommand{\Var}{\mathrm{Var}}$$
$$\newcommand{\E}{\mathrm{\mathop{\mathbb{E}}}}$$

## Monte Carlo Tree Search
Monte Carlo Tree Search (MCTS) [[2]] [[4]] is a way to use Monte Carlo rollouts to estimate the value of each state. It keeps a tree of action-values estimates $$Q(s,a)$$ and visit counts $$N(s,a)$$ and $$N(s)$$, and uses these to further expand the tree more efficiently. Every step, the tree is traversed until a leaf state $$s_L$$ is found. Then, a value estimate $$V(s_L)$$ is computed (usually with a Monte Carlo rollout), and all traversed $$Q(s,a)$$ and counts are updated.

AlphaGo introduced a new way to evaluate the leaf states. Instead of relying only on rollouts, it computed $$V(s_L)$$ as a combination of a rollout and the output of a function approximation, trained to predict the probability of winning from $$s_L$$.

UCB1, studied in depth by [[3]], is a strategy for the bandits problem. Briefly, a bandits problem is a stateless MDP. So, instead of estimating $$Q(s,a)$$, we can just estimate $$Q(a)$$. UCB1 follows the principle of optimism in the face of uncertainty. Intuitively, we want to search the actions that might lead to the highest reward, even if the current $$Q(a)$$ is low. At each step, UCB1 selects the action that has the highest score $$Z(a)$$.

$$Z(a) = Q_{N(a)}(a) + C_{N(a),N}(a)$$

$$Q_n(a)=\frac{1}{n}\sum_{i=1}^{n}r_i(a)$$

$$C_{n,N}(a) = \sqrt{\frac{2\ln{n}}{N}}$$

Where $$r_i(a)$$ is reward obtained the $$i$$'th time action $$a$$ was taken, and $$N$$ is the total number of actions selected. $$Q_n(a)$$ is the estimate of the value of action $$a$$ after selecting it $$n$$ times.

By Hoeffding’s tail inequality, it can be proven that $$C_{n,N}(a)$$ is an upper confidence bound.

$$P(Q_n(a) \geq Q(a) + C_{n,N}(a)) \leq N^{-4}$$

Where $$Q(a)=\E{[Q_1(a)]}$$ is the true action value.

[[3]] shows that UCB1 quickly converges to the best action. It efficiently solves the exploration-explotation dilemma. It balances between searching over all actions so that it doesn't miss the best one, and quickly narrowing the search over the actions that are proving to be good. The more an action is selected, the more its bound $$C(a)$$ tightens, converging to the greedy policy.

The most commonly used MCTS method is UCT. Proposed by [[2]], it treats the action selection on the tree as a bandits problem, and uses UCB1 to solve it. Similarly to UCB1, the action with the highest $$Z(s,a)$$ is selected.


$$Z(s,a) = Q_{N(s,a)}(s,a) + C_{N(s,a),N(s)}(s,a)$$

$$C_{n,N}(s,a) = C_p\sqrt{2\frac{\ln{n}}{N}}$$

$$Q_n(s,a)=\frac{1}{n}\sum_{i=1}^{n}V_i(s,a)$$

$$P(Q_{n}(s,a) \geq \mathop{\mathbb{E}}{[Q(s,a)]} + C_{n,N}(s,a)) \leq N^{-4}$$

Where $$V_i(s,a)$$ is value estimate computed from a leaf state found when traversing $$(s,a)$$ the $$i$$'th time. Since we are now estimating the values of a tree, the estimates might drift. That is why [[3]] introduced the constant $$C_p \geq 1$$. They prove that if $$C_p$$ is large enough, $$Q_{N(s,a)}(s,a)$$ will converge. Similarly to UCB1, UCT solves the exploration-explotation problem for MDPs. In fact, for the general MDP we can't do much better.

But sometimes this isn't enough. When the state space is too big, it is often not feasible to use UCT. It might take to long before the bounds tighten and the search can proceed efficiently. This is the reason AlphaGo uses a heuristic to drive the search, to get better, earlier value estimates.


## Learning the bounds
Consider the case where we can discart $$\frac{1}{2}$$ of the actions of every state. If before the computational costs where proportional to $$N_a^D$$, where $$N_a$$ is the average number of actions per state and $$D$$ is the depth the search is performed, after such reduction the computational costs would be $$\frac{N_a}{2}^D$$. Therefore we would have a reduction in computation of:

$$\frac{\frac{N_a}{2}^D}{N_a^D} = \frac{1}{2}^D$$

In a game like Go that has approximately 300 steps, this could mean a reduction on the order of 1e-91! (if the search was performed until the end of the game)

The basic idea I am introducing here is that better bounds, conditional on the state, can be learned. By learning to accurately predict the uncertainty, we could use optimism in the face of uncertainty to search over the actions that truly might to lead to high rewards, and ignore the ones that wont. If it is possible to accurately estimate the value of a state why not its uncertainty? Further, the value estimates and bound estimates could share features. There would barely be any extra computation requirements, and the efficiency of MCTS would be greatly improved.

To put it in mathematical terms, we want to learn new bounds $$C_{n,N}(s,a)$$ for which $$(8)$$ still holds (so that the convergence guaranties of UCT still apply). The tightest the bounds we find, the quicker the search will converge.

The simplest and most obvious way to do it is by estimating the variance of $$Q_1(s,a)$$. <sup>2</sup>

$$P(Q_n(s,a) \geq \E{[Q(s,a)]} + C_{n,N}(s,a)) \leq \frac{\Var(Q_1(s,a))}{n C_{n,N}(s,a)^2} $$

So, if $$C_{n,N}(s,a) = N^{-2} \sqrt{\frac{\Var(Q_n(a,s)}{n})}$$ the bound $$(8)$$ will hold.

We can now use any function approximator to estimate the variance and search with these better bounds.

There is, though, a more appealing idea. What if instead of learning a tighter upper bound, we directly try to estimate the exact confidence bound

$$P(Q_{n}(s,a) \geq \mathop{\mathbb{E}}{[Q(s,a)]} + C_{n,N}(s,a)) \approx N^{-4}$$

With this goal in mind, we can construct a loss. Estimating the bound $$(8)$$ is equivalent to finding the point where only $$N^{-4}$$ of the samples fall to the right. Mathematically, it can be constructed this way.

$$L = \sum_{s,a,n,N}\E{[l_{n,N}(s,a)]]} $$

$$ l_{n,N}(s,a)= d_{n,N}(s,a)
\begin{cases}
    N^{-4}, & \text{if } d(s,a)\geq 0 \\
    (1-N^{-4}), & \text{otherwise}
\end{cases}
$$

$$d_{n,N}(s,a) = Q_n(s,a)-\E[Q(s,a)]-C_{n,N}(s,a)$$

By learning the actual confidence bound we would be learning the optimal search strategy of optimism in the face of uncertainty.

But what sort of parametrization of bound can we use? It would be tempting to the form $$f_\theta(s,a) \sqrt{\frac{\ln{n}}{N}}$$, but it is not clear that this is the right form. Does the optimal bound really tighten at this rate for all states? And if we only train using $$n$$ up to a certain depth, we cannot be sure that the bound will hold after that.

Ideally we would like to find a bound for which we can prove that if $$(11)$$ is true then this is also true.

$$P(Q_{n+1}(s,a) \geq \mathop{\mathbb{E}}{[Q(s,a)]} + C_{n+1,N}(s,a)) \approx N^{-4}$$

What form of bound exhibits this behavior I don't know.

Another option would be to directly learn the bound and pass $$n$$ and $$N$$ as inputs and make sure it generalizes up to the depth we will search during a game.

## Does it converge?
You might be thinking that, although learning better bounds could lead to large improvements, it would break the convergence guaranties of UCT. After all, we cannot prove that bounds dependent on a function approximator will hold over all states and actions. For example, we might underestimate the variance of some actions, not exploring them as much as we should. Luckily for us, UCT already accounts for cases where the bounds don't hold. Therefore, for a large enough constant $$C_p$$, UCT with a learned bound will converge. Of course, having a large $$C_p$$ defeats the whole purpose of learning tight bounds. So it would be necessary to find the right balance.

## Beyond Go
Although I have only been talking about the game of Go, learning to search is a fundamental problem of intelligence with many different applications.

In particular, the frontier that I find the most interesting is that of training RL agents that use generative models of the world to decide which actions to take (my current research project is exactly on this topic). In complex worlds with large action spaces, learning to search will be crucial to use the learned generative models efficiently.

---------------------

### Notes
1. AlphaGo uses the network trained to imitate human moves to perform search. Although this network isn't hard-coded, its usage as a search policy is. There is no principle to it. It was simply empirically found to work better than all other search strategies tested.
2. I am sure there must be a better bound for the case where $$x\in[0,1]$$. If you know it please reach out to me.

---------------------


### References

{% for i in (1..ref_names.size) %} {%assign j = i | minus: 1 %}
 {{i}}. [{{ref_names[j] }}] [{{i}}] {% endfor %}
