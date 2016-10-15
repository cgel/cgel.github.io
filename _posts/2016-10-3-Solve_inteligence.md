---
layout: post
title: Solve intelligence in 5 easy steps
excerpt_separator: <!--more-->
autoNumber: '"all"'
---

<p class="message">
  This is still work in progress. Some parts are not finished and many references are missing. If you have any feedback please send it to {{ site.author.email }}
</p>

In this article I present several research ideas that are both, fundamental to intelligence and tractable. I also tackle the control problem, and present a solution to introduce ethics into a intelligent RL agent.

<!--more-->


Neural networks have shown an incredible ability in approximation and generalization. Yet, it is not wildly accepted that they are - at least on their own - enough to result in true AI. As [[1]] puts it:

> the claim that a mind is a collection of general purpose neural networks with few initial constraints is rather extreme in contemporary cognitive science.

Here, I argue that for all tasks relevant to AI, any apparent shortcomings neural networks might have are more the result of an ill defined problem than a fundamental limitation. I believe that the way forward is to choose the right problem and make sure that the network has enough computational flexibility to develop the necessary cognitive skills.

I also explore how _"general purpose neural networks with few initial constraints"_ could learn in a similar manner to what the consensus in early developmental psychology and linguistics expect. How, for example, core concepts of objects and language might appear early on.


## 1 The right problem
Whether RL is the way to achieve intelligence is a subject of debate. Its detractors have exposed many shortcomings. I believe, tough, that RL is the only setting general enough to achieve intelligence and that these shortcomings should inspire instead of disparage research.

One shortfall of RL is the difficulty to do knowledge transfer. DQN [[5]] has to train a new networck for each game. And although several approaches have been proposed to solve this problem [[7]], [[8]], [[9]]  they are ultimately unsatisfactory because they require a teacher that already knows how to play. There is, though, a canonical way to do knowledge transfer: string the individual tasks together. Something similar to this was done by [[7]] to be used as a baseline with some promising results - though not much information was provided. An analogy with supervised learning would be: learning a separate classifier for each class vs learning a single multi-class classifier. When multiple tasks have commonalities, sharing representations should significantly increase the learning speed . And the only constrain is that the states spaces of each task should not intersect - and in atari2600 they don't.

There is a bigger problem though, making the policy depend only on the state is very limiting. Just consider tasks that require memory like imitation or that include a description of the objective. Note that a RL agent could solve any of this tasks if their state spaces didn't intersect. The limitation is that the "true" policy cannot be learned, therefore, whichever policy it is leaned will not generalize well.

This is related to another fundamental difference between RL and real intelligence, the way in which they explore. Exploration for a real intelligence is complex. It uses rapid model building and concepts like intuitive physics and causality to decide which action will provide the necessary information for the current task. In short, to learn quickly intelligence is necessary. In contrast, RL exploration techniques like $$\epsilon$$-greedy are simple. And any method that tries to do something more intelligent like [[10]] or [[11]], does it orthogonally to the policy. I believe that the root problem is that RL is fixed on learning to play as opposed to be learning to learn how to play.

I think that there is a simple solution: Make the policy depend on the extended information state $$ \hat s $$ -  which includes all previously visited states, the actions taken and the rewards received - using a RNN. To see how this could solve the previously mentioned shortcomings it is easier to consider the case where the agent trains on each task only once. In this case, if there is uncertainty in the task and exploration is necessary the agent would have to explore and use the observations. If the tasks required language it could only solve them by learning the language. This is analogous of how [[15]] was able to learn to classify with very few examples - by also using $$ \hat s $$.

If, for example, there was uncertainty over the properties of an object, the agent would learn to infer them after the minimum number of observations possible, using visual queues and ideas of compositionality to drive the inference - learning something analogous to the core object concept. Of course, this type of cognitive tasks will require powerful recurrent models like [[12]], [[13]], [[14]].

I believe that by learning to act under information extended space and a lots of tasks it is possible to achieve true intelligence.

There would be two main versions of this method, the one where state $$ \hat{s} $$ depends on the episode, and the one where it depends an all previous episodes. Since the second method would have a much bigger state space it might be more challenging to train, but it would also be more powerful, and it clearly is also more resemblant to human intelligence.

Note that the agent could not learn good exploration policies without having previously explored. Therefore, in the beginning the agent would still depend on a simple exploration technique like $$ \epsilon$$-greedy. Another important thing to note is that since the policies now would depend on the rewards received, the behavior under no rewards would be unpredictable. This might seem highly constraining for real world applications but i will later propose a method to remove this constraint.


## 2 Imagination
The ability to “imagine” what would happen if an action is taken  seems a fundamental property of intelligence. Considering different situations, and reasoning about them is a useful cognitive tool. Giving neural networks these tools is therefore a interesting research problem.

Here, I am only going to discuss a constrained form of imagination, model based planning.

In highly structured environments learning to plan could be much easier than directly learning a policy. Planning could then be used to take better actions or even update the policy.

The main component necessary for planning is the transition probability matrix $$P_{s,s^\prime}^a$$ or, if the MDP is deterministic, a transition function $$T: (s,a) \to (s^\prime, r )$$. In the case of a large, stochastic state space, it would probably be better to learn a stochastic transition function than the probability transition matrix.

Approximating $$ T(s,a) $$ could be inefficient - like in the case where the state is an image. An alternative approach would be to take a lower dimension, inner representation $$h(s)$$ of the policy, and approximate an equivalent transition function $$ T(h(\hat s),a)$$. We could then add $$(h^\prime-h)^2$$ to the loss or, if we are using $$ Q $$ functions, we could alternatively add $$(r + \lambda \max_{a^\prime}{Q(h^\prime,a^\prime)} - Q(h, a) )^2$$.

It is also easy to envision a multi step version where the transition function function is trained use "synthetic" hidden states.


## 3 End-to-end learning
A clear trend in the field of AI is the tendency towards end-to-end. Problems are so difficult that a solution can only be learned. [[2]] took it a step further by not only learning the problems, but by learning the learning algorithm. I think that this is a very promising direction.

### 3.1 Credit assignment
Credit assignment is one of the fundamental problems of RL, and is one of the biggest sources of the variance of the updates. Could use the intelligence of the agent to do explicit credit assignment?

When a human go player studies a game he is looking for the good and the bad moves and using this information to play better. Instead, in [[18]] the updates to all actions depend on the final reward. Learning to do credit assignment would likely increase the performance or at least increase the learning speed.

### 3.2 Updates
The canonical RL algorithms sample from the stationary distribution of states. But since this is intractable in the real world alternative update methods have been proposed. [[5]] used a replay memory that contained the last 1 million transitions and sampled the updates uniformly from it. [[17]] also used a 1 million sample replay memory and improved on it by biasing the sampling towards transitions with higher TD error. One problem to this approach is that many "useless" transitions are maintained in memory. Another, is that the policy for any state that has not been seen in the last 1 million transitions will degrade very quickly.

I believe that a much better update process can be learned. The update network would decide which transitions to store - potentially only storing inner states - and how to sample them.

The network could also use information on the stability of the representations, knowing how the current update would change the policy on previously sampled transitions. It could then find an alternative update that improves the policy on this state but does not change the it in the previously seen ones.


## 4 Learning hierarchies
Many real world tasks can be easily broken down into subtasks and organized in hierarchies. In these cases, hierarchical RL can alleviate conceptual and temporal abstraction difficulties and help generalization. It can also ease the credit assignment problem, speeding up learning.

Consider the case where a part of the brain (a module) is responsible for motor control and its reward is the "top level" reward. Here, learning is limited by a possibly very sparse reward. In addition, other independent parts of the brain can affect the updates to this module - because they affect the reward.

In contrast, if a person tasked with moving from A to B trips, he will learn to walk better whether he succeeded or not. If instead he failed to complete the task due to a planning error, he will not change his the walking policy.

Work like [[16]] has explored hierarchies . It consists of two $$Q$$ functions, a controller $$Q_1(s,a,g)$$ under a "intrinsic reward" $$r_\prime(s,a,g)$$ and a meta-controller $$Q_2(s,g)$$ under the real reward. Here $$g$$ is a goal, or command, from the meta-controller to the controller, an order from a high level to a low level. The meta controller selects goals for the controller, and the controller tries to achieve them - tries to maximize the intrinsic reward that is conditional on the goal. While the method has shown significant performance gains, it hasn't tackled the fundamental challenge of hierarchical RL: How to learn the reward signal for each module - the intrinsic reward? Instead, it has relied on a hand crafted intrinsic reward function.

Then, the challenge is not only to learn the subtasks policies, but to also learn a reward signal $$r(s,s^\prime,a,g)$$ for each module. This is, of course, a very complex problem, and presents the biggest drawback to hierarchies.

The problem of learning reward functions could be approached in a RL fashion, interpreting the individual rewards as actions. First, we would wait until the policies have converged given the current reward function. Second, we would update the reward function in a similar manner that we would update a policy. And then we would repeat.

I believe that learning hierarchies will improve performance whenever learning a general reward signal is easier than learning a general policy. Like for the subtask of walking, where a velocity vector is passed to the module as a target. In this case, learning to robustly evaluate the walking speed - necessary to compute the correct reward of the subtask - is clearly easier than learning a walking policy for all possible circumstances - like multiple terrains or when carrying heavy objects.

Another advantage of hierarchies is that the representations could be much smaller because they only need to contain the information relevant to the particular subtask. Therefore, generalization will be much faster and reliable.

And since hierarchical RL is ultimately a collection of RL problems it can be easily combined with the methods previously described. This is especially interesting in the case of planning over different temporal and conceptual representations.


## 5 The ethics signal
If the previous propositions are successful we might end up with a truly intelligent agent. It might be able to solve a wide spectrum of tasks, read and understand task descriptions, use imagination to learn faster and take better actions, and use hierarchies to find better temporal and conceptual representations. Yet, in the real world, where there is no reward signal, its usage would sill be highly limited. Not only it could not learn anything new without rewards. It could not even perform a task that it has already learned - because, as i mentioned earlier, the policy depends on the received rewards.

There is also the control problem. Any desirable reward signal for the real world cannot be dependent on simple goals that can be achieved at any cost. Instead, it must be an ethic reward signal (an ethics signal) that relies on high level concepts like language, physics and psychology among others.

The idea i am proposing is simple: use the learned representations to predict an ethics signal in a supervised fashion and then use this signal as the RL reward signal that the agent tries to maximize.

To do this we would need to extend the neural network with a scalar output and define a large set of “ethics augmented tasks” – normal RL tasks with an ethics signal target. Defining such tasks would be a big challenge on it own, since  we need to make sure their ethics signal is one that we want to be maximized.

Then another challenge will be to confirm that the learned ethics signal has generalized over all states that the agent might find itself into. The difficulty of this will depend on the agent having learned the appropriate representations. If, for example, the agent has learned a good language representation, the parts of the ethics signal that depend on language will be quickly learned.

Very importantly, this method does not force the agent to stop learning after it has been “set free” in the real world.  A RL algorithm can still be used to maximize the ethic signal rewards - although the updates should not change the ethics signal.

---

### References
{% assign ref_names = "" | split:"|"  %}
{% assign ref_urls = "" | split:"|"  %}
<!-- 1 -->
{% assign ref_names = ref_names | push:
  "Building Machines That Learn and Think Like People"%}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1604.00289" %}
<!-- 2 -->
{% assign ref_names = ref_names | push:
  "Learning to learn by gradient descent by gradient descent" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1606.04474v1" %}
<!-- 3 -->
{% assign ref_names = ref_names | push:
  "A Roadmap towards Machine Intelligence" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1511.08130" %}
<!-- 4 -->
{% assign ref_names = ref_names | push:
  "Neural Turing Machines" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1410.5401v2" %}
<!-- 5 -->
{% assign ref_names = ref_names | push:
  "Human-level control through deep reinforcement learning" %}
{% assign ref_urls = ref_urls | push:
  "http://www.nature.com/nature/journal/v518/n7540/full/nature14236.html" %}
<!-- 6 -->
{% assign ref_names = ref_names | push:
  "Policy Gradient Methods for Reinforcement Learning with Function Approximation" %}
{% assign ref_urls = ref_urls | push:
  "https://webdocs.cs.ualberta.ca/~sutton/papers/SMSM-NIPS99.pdf" %}
<!-- 7 -->
{% assign ref_names = ref_names | push:
  "Policy Distillation" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1511.06295" %}
<!-- 8 -->
{% assign ref_names = ref_names | push:
  "Actor-Mimic: Deep Multitask and Transfer Reinforcement Learning" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1511.06342" %}
<!-- 9 -->
{% assign ref_names = ref_names | push:
  "Deep Learning for Real-Time Atari Game Play Using Offline Monte-Carlo Tree Search Planning" %}
{% assign ref_urls = ref_urls | push:
  "https://papers.nips.cc/paper/5421-deep-learning-for-real-time-atari-game-play-using-offline-monte-carlo-tree-search-planning" %}  
<!-- 10 -->
{% assign ref_names = ref_names | push:
  "Unifying Count-Based Exploration and Intrinsic Motivation" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1606.01868" %}
<!-- 11 -->
{% assign ref_names = ref_names | push:
  "Incentivizing Exploration In Reinforcement Learning With Deep Predictive Models" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1507.00814" %}
<!-- 12 -->
{% assign ref_names = ref_names | push:
  "Reinforcement Learning Neural Turing Machines - Revised" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1505.00521" %}
<!-- 13 -->
{% assign ref_names = ref_names | push:
  "Neural Turing Machines" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1410.5401" %}
<!-- 14 -->
{% assign ref_names = ref_names | push:
  "Neural Programmer-Interpreters" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1511.06279" %}
<!-- 15 -->
{% assign ref_names = ref_names | push:
  "One-shot Learning with Memory-Augmented Neural Networks" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1605.06065" %}
<!-- 16 -->
{% assign ref_names = ref_names | push:
  "Hierarchical Deep Reinforcement Learning: Integrating Temporal Abstraction and Intrinsic Motivation" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1604.06057" %}
<!-- 17 -->
{% assign ref_names = ref_names | push:
  "Prioritized Experience Replay" %}
{% assign ref_urls = ref_urls | push:
  "https://arxiv.org/abs/1511.05952" %}
<!-- 18 -->
{% assign ref_names = ref_names | push:
  "Mastering the game of Go with deep neural networks and tree search" %}
{% assign ref_urls = ref_urls | push:
  "http://www.nature.com/nature/journal/v529/n7587/full/nature16961.html" %}



{% for i in (1..ref_names.size) %} {%assign j = i | minus: 1 %}
 {{i}}. [{{ref_names[j] }}] [{{i}}] {% endfor %}

{% for i in (1..ref_names.size) %}
  {%assign j = i | minus: 1 %}
  [{{i}}]: {{ ref_urls[j] }} "{{ref_names[j] }}"
{% endfor %}
