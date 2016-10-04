---
layout: post
title: How to solve intelligence
excerpt_separator: <!--more-->
autoNumber: '"all"'
---

<p class="message">
  This is still work in progress. If you have any feedback please send it to {{ site.author.email }}
</p>

In this article I try to explain my vision on the fundamental properties of intelligence and control and propose research paths to replicate them. Remarkably none of these properties are out of the scope of what can be mathematically represented and implemented. Although i don't claim this list is exhaustive, i do believe that the most important properties are in it.
<!--more-->

Neural networks have shown an incredible ability in approximation and generalization. Yet, it is not wildly accepted that they can - at least on their own - be enough to result in AI. As [[1]] puts it:

> the claim that a mind is a collection of general purpose neural networks with few initial constraints is rather extreme in contemporary cognitive science.

Here I argue, that for all tasks relevant to AI, any apparent shortcomings neural networks might have are more the result of an ill defined problem than a fundamental limitation. I believe that the way forward is to choose the right problem and make sure that the network has enough computational flexibility to perform the necessary cognitive tasks.

I also explore how a _"general purpose neural networks with few initial constraints"_ does not have to disregard consensus in early developmental psychology and linguistics. How, for example, core concepts of objects, agents, and numbers might appear early on.

If the ideas i present don't have any fatal flaws, they open a very exiting research path mainly because they are tractable. Truly, any of the main points described could be quickly implemented. And judging by the current speed of research, a fully integrated agent could be developed in under a year - although it would probably live in a low dimensional space.

## 1 The right problem
If we are intending to create a human like intelligence and we are going to do it with neural nets the only place to start is with RL. It offers a framework general enough that might seem sufficient to achieve intelligence. One apparent shortfall is apparent in DQN [[5]]. For each game a new DQN has to be trained, and there is no simple way of transfering knowledge them. çsomething fundamental to increase learning speed. Several approaches have been proposed to solve this problem [[]], [[]], [[]] but they are unsatisfactory because they all require a teacher that already knows how to play. There is, though, a canonical way to get an agent that plays multiple tasks: string the tasks together. The only constrain is that the states spaces of each task can't intersect. Something similar was done by [[]] to be used as a baseline. Some of the results where promising but not much information was provided. An analogy with supervised learning would be: learning a separate classifier for each class or learning a single multi-class classifier. Sharing representations should significantly increase the learning speed, especially when multiple tasks have commonalities. Another vantage is that the number of tasks can be expanded at any point in the training.

There is a bigger problem though, the policy depends only on the state, and that carries many limitations.

The main characteristic of intelligence is the ability to transfer knowledge from one task to another. For example: an intelligent agent that as learned the extract a useful representation of the game of pong should be able to use it to learn to play the game breakout much faster. The tow games share many underlying principles which can and should be mapped in a common  representation. The concept of an visual object, bouncing physics, the concept of objects under its control, the idea of a goal, these are all shared among both games – and indeed many others. Therefore the ability to share and use a common representation should result in a vast increase in performance – especially learning speed.

Isn't it what using the weights leaned on a task for another does? No, copying the weights is nothing more than sweeping the issue under the rug, as it is made obvious when you move from the canonical example. Consider this: The agent first has to learn to play pong, then switch to breakout and finally it's presented with a modified version of pong that simply adds a few breakout style blocks in the middle of the field. It is to be expected that an truly intelligent agent which successfully played pong and breakout should master the 3rd game in no time. But it won't. If the agent copied the weights from the 1st game to the 2nd it might learn it faster, but when it moves to the 3rd task the flaw becomes obvious, the weights learned in the 2nd game have been learned with no regard to keep the 1st game policy intact, they will, therefore, inevitably perform  poorly – likely no better than random – in the 3rd task. Copying weights does not do true knowledge transfer, it nothing else than a weight initialization method. And true knowledge transfer is essential for real intelligence.

The lessons about physics that you learn when performing a task influence your understanding of physics in every other task you might perform. When you learn a new word it does not matter if you where having a conversation or reading a book, you will be able to use that word in any context.

One might then think that by makeing a “composite task” by stringing together multiple tasks the problem of knowledge transfer might be resolved.

When a real AI tackles a new problem it brings in a complex prior over the reward functions and transitions. It generates multi step exploration trajectories to evaluate those hypothesis.

For example, a real AI can read a description before starting a task and have a good policy from moment one.

When the value of the knowledge gained from exploration is represented explicitly, a greedy policy can be followed.  This is contrary to exploration techniques such as e-greedy and softmax.

Lets now look at the problem from another angle, and pay attention on a fundamental difference between the general reinforcement learning setting and real intelligence. A RL agent learns to maximize performance over a certain MDP. It doesn't bring prior knowledge to task. Instead a real tintelligence finds itself it a very different setting. It is not supposed to master a single task, it instead prioritizes learning fast. One of the main ways it does it if by inferring the reward system, and dynamics of the task. Lets look at a very simple task and see how a true AI should handle it.

We are at a stateand we know – either from a description or previous tasks – that each time an action is taken state transitions toor if task ends. With action 1 reward is 1.  Action 2, reward is either 10 or 0. And here comes the interesting part. This is not an MDP, this is a set of two MDPs. One that returns 10 on action two and another one that returns 0. Therefore the reward from action two is the same regardless of the state. There is a 0.05 chance that the MDP is the first one. And the initial state is random – the distribution does not matter.

Obviously trying to learn a value function over multiple MDPs does not make sense and does not lead to the optimal policy. We can, though, look at the optimal policy and see on what it depends.

If we have previously taken action 2 the optimal policy is simple: If the reward was 10 always take action 2, if reward was 0 take action 1.
If we have not taken action 2, taking it and following the optimal policy has cost . Taking instead action 2 has cost . Therefore taking action 2 is optimal when .

To find the optimal policy we used priors – that the rewards of action 1 and 2 where constant and that with probability 0.05 the reward of action 2 would be 10 – and previous transitions.

It is fundamental to Intelligence that it learn to act under uncertainty.

There is a field called Bayes-adaptive RL that tackles a apparently similar task. But it assumes that we know how to update the value given new transitions. This might be possible in the example, but it is completely unknown for more general tasks.

But lets not loose focus on what matters. The real AI has have strong priors and depend on the history (I.e have memory, work on the information POMDP etc.). And it turns out that there is a simple method that does that, a RNN.

We can just train a RNN to predict the value of information extended state, over a set of MDPs.

Information extended learning is enough to do true knowledge transfer. It can learn exploration strategies that will generalize. One thing to worry is that in the real world no 2 states are equal, so it is not clear if the policy will generalize over the uncertainty or it will fix on meaningless details of the state.

The question is then, how to do true knowledge transfer? For the human kind of intelligence I think that there is only one answer, learn a single composite task or in other words learn to learn tasks. This is the moment we have to leave the slippery world of English and use math.

But ability to transfer knowledge must go further than this canonical example.

One important thing to note is that in metal earnign the policy depends on the previous rewards. That means that if the agent is left to play without the supervision of the reward the policy could be completely different. Something that did not happen with non augmented RL. This might seem worrying for applying it to real world tasks, but is something that I will deal lather with.

Generality: the same system should be able to work on a wide variety of problems.

Curriculum learning will be critical for general AI.

## Hierarchical

For most tasks that we perform it is easy to break them down into subtasks. To fetch something from the frig you have to walk and then grasp. These are two different and reusable subtasks, under what you might call central control.
In a more general setting we might have a top level control, reading the instructions of the task and giving orders to a walking module when displacement is necessary and grasping orders to a grasping module.

The idea of a module is that it gets a order, a description of what it should do, and then a reward, a measure of how well it did. This of course is the description of a RL problem. Therefore a module must solve the RL problem posed by its parent, by taking actions or giving orders and rewards to child module/s.

This is intuitive, but there also are good arguments on why this might be useful. First of all credit assignment.

We have a general agent, it has learned to read the description of a task, spacial planning and motor control to achieve it.  Let's say that the agent is tasked with moving boxes from A to B I a certain time. It is pretty smart so it reads the description picks up a box and gets on its way, but since this agent has never walked wile holding a box before it does so clumsily.

In the case the the agent is not hierarchical. If the agent still manages to deliver in time no negative reward will be given and the agent will not learn to walk better with a box. If it misses the delivery time, the weights of all parts of the network will be penalized, even the parts that did the reading and planning.

Now lets consider a hierarchical agent that has learned a top reading module, a middle planning module, and a bottom motor module. Event if for this task walking at the exact speed ordered by the planning module is not critical, a general agent – and by extension a general planning module – will have learned that it is. Hereby the planning module will be giving bad rewards to the motor module and the agent will learn to be agile holding a box.

Different modules of a hierarchy can have different \\( \lambda \\), it is reasonable to assume that a bad reward to the motor module is due to a recent action. That can not be said of the planning module for example. The gammas for each module could be set as hyper-parameters or maybe could be continually passed by the parent module.

Hierarchies might also be useful as priors. In other words, the real world tasks might lend themselves very well to being represented this way. For example it might be much easier to learn to plan a trajectory under uncertainty if there is only the essential in the representation.

There will probably be a big drawback to hierarchies, they might be much harder to train and require more data.

Hierarchical modules can easily work in different timescales.


## Imagination or planning

The ability to “imagine” what would happen if an action is taken  seems a fundamental property of intelligence. Considering possible situations …

What I am proposing here is more analogous to model based planning that imagination. I think that pursuing a more general, less constrained imagination is worthwhile and necessary for certain cognitive tasks.

I am not going to argue if and why is that learning a transition function is easier than learning a policy of action-value function. –---- Should I?

The main component necessary for planning is the transition probability matrix or, if the MDP is deterministic, a transition function . Either of these can be approximated and then be used to open trees in a plethora of ways. I think that the ones with more potential are done in conjunction with a policy or action-value function.

I don't want to discuss all different ways of opening the tree, but I want to pay attention  

When using neural networks either a policy or a action value function have inner layers that could be interpreted as simpler representations of the state. Then we have an equivalent and .  If we recognize that imagining the future – learning the transition function – is a useful cognitive ability

## Representation stability
This is about the normal RL, regardless of generality, hirearchy or imagination. How to learn a good policy over all visited states – and the ones it generalizes – without having to keep all observations in memory.

One idea is that once the agent is already pretty smart the representations should try to be stable over the previously seen observations. This would force the new updates to change the representations only over the new observation and “not mess up” the representation over the previous observation space.

An intuition is that if a previously seen vector has a meaning, the new update should not try to change it's meaning.

A possible approach would consist of continually computing a term that tells how insensitive is the representation to the parameters. Something like:  

$$ u = \mathop{\mathbb{E}} [ \frac{\partial\pi(\hat{s} )}{\partial\theta} ^{\frac{-1} 2}] $$

The expectation being over the previous observations. And then updating with

Where is the element wise product and is what normally would have been the update (e.g. momentum, rmsprop etc).

Another intuitively similar option would be to update a projection of quite orthogonal to all previous gradients.

But in the end I think that the solution will be a network controlling the updates in a comparable way as [].

## Learn to learn/train
In an analogous manner with how [[1]] used a neural network to drive the updates, it seems reasonable that the intelligence of the agent could be used to explicitly assign credit. For example if the motion planning module has not achieved its goal it makes scene to reason which set of actions are responsible and only update the weights for those. This is a mechanism that is made apparent when studying a played game, or a trainer is correcting your position. Credit assignment uses the intelligence of the agent.

The agent should probably also control when and what to update.

The natural way of approaching this task if from a RL perspective. Figure out what to assign credit to by trial and error. But it also seems like it could be integrated with the previously learned planning to avoid having to learn anything.

This of course raises problems about stability.

## The ethics signal
We might have an agent that is truly intelligent, it might be able to read and understand task descriptions. It might be able to plan trajectories and complete them. But what use does it have if to learn a new task it always needs a reward signal. And, as we discussed previously, the policy depends on the reward, so that a non existent reward means an unpredictable policy.

What we wont is to be able to introduce ethics – in a very broad sense – to the AI such as: do what I want you to do – Knowing what I want is a matter of intelligence, something that we are assuming by this point. The ethics signal should be constructed in a way that you wold want an agent to maximize it. It could encompass things like do what I want, protect people, don't run way to avoid commands, be smooth in your movements, etc. And an agent with a higher accumulated ethics signal should always be preferred to another agent with a lower one.

The idea is simple, use the intelligence of the machine of the machine to learn to predict the ethics signal and then use this signal as the reinforcement learning signal. To do this we will need to extend the model of the agent. Adding a scalar output, function of a diverse set of representations. And then train this output as a normal regression task.

A large set of “ethics augmented tasks” – normal tasks with an ethics signal – must be defined. For which you can be sure that if the agent is performing well in this tasks it has generalized the ethics. It is important to note that the agent does not act according to the ethics in any way. It acts as RL agent as it always has. It is simply also predicting the ethics signal. The RL reward signal might be or might not be the ethics signal. It is especially important to have a good prediction of the ethics signal in situations with a low value, thus, if the agent is trying to avoid them, it will be harder to be sure the ethics prediction has generalized.

Once the agent is correctly predicting the ethics signal it can use it as the RL reward. It will use its general intelligence to learn to maximize it, and we will have a intelligent agent acting according our predefined ethics without the need to supervise with the signal!

A nice property of this method is that the agent will continue learning after it has been “set free” with its ethics. It will be interesting to study how learning new things affects the ethics prediction, e.g. After learning a new word will the agent predict the correct ethics signal if it has received a command using this word? Intuitively it seems that it should.

## Final thoughts
Among the sections in this article we can separate the two most important ones. Section 1, that strives to make the agent intelligent. And section X that intends to "control" the agent. It is surprising how well these two fit together.##


1: Train a net into a set of low dimensional increasingly challenging tasks. The network will have to  have memory to be able to learn basic to learn. Solve this current problems:
    1: Making sure that it does not “forget” how to act in states that it hasn't visited for a while.
    2: Use goal hierarchies. They will be very useful to learn to explore.
    3: Use the hierarchies to do planning.
2: Get the AI to work on high dimensional tasks like modern games and speech recognition etc.
3: Define a set of situations and questions with a reinforcement signal for which high signal means to be ethic. In other words, define a measurable ethic, one in which you can be confident that an agent with a high score will act accordingly with this ethic in all the relevant domains – will have generalized well. Then give the AI the task of predicting the ethics signal as opposed to the task of maximizing it.
4: Set the AI free in the world with the task of maximizing the ethics signal that is predicting itself.

RULE THE WORLD!!!!!!

---

References
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
  "" %}
{% assign ref_urls = ref_urls | push:
  "" %}
<!-- 8 -->
{% assign ref_names = ref_names | push:
  "" %}
{% assign ref_urls = ref_urls | push:
  "" %}
<!-- 8 -->
{% assign ref_names = ref_names | push:
  "" %}
{% assign ref_urls = ref_urls | push:
  "" %}  

{% for i in (1..ref_names.size) %} {%assign j = i | minus: 1 %}
 {{i}}. [{{ref_names[j] }}] [{{i}}] {% endfor %}

{% for i in (1..ref_names.size) %}
  {%assign j = i | minus: 1 %}
  [{{i}}]: {{ ref_urls[j] }} "{{ref_names[j] }}"
{% endfor %}
