---
layout: post
title: Meta-RL is the key to intelligence
excerpt_separator: <!--more-->
future: true
autoNumber: '"all"  '
---

There is a long list of cognitive skills that an AI should muster to be truly intelligent: language, bayesian reasoning and inference, one-shot learning (for example when observing the dynamics of an object), logical reasoning, different types of memory, learning from imitation, learning by instructions (like a written task description or the verbal feedback provided by a teacher), transfer learning, etc.

The common research approach is to select one skill that you want to imitate, simplify it to the point where you can model it mathematically and find an appropriate loss to minimize. Although the simplifications might work for simple problems, it is always easy to see how it might break down with harder tasks. I do not believe this approach will scale.

There is a simpler and more powerful way to tackle this lit of challenges, meta-RL. In meta-RL we learn a policy to learn policies. In other words, we train the agent to quickly use the information it gathers in a task to maximize its reward in it (and we train it over many different tasks). It can also be interpreted as learning to act under uncertainty or learning to explore (but more on that in a later post). Very importantly, meta-RL only requires minor modifications to a standard RL algorithm - it should be recurrent and it should have the previous reward as an input.

In meta-RL we find a unified strategy to "force" the agents to acquire any necessary skills, as I will now demonstrate with the example of imitation learning.

<!--more-->

When people are provided with an example of how to perform a task, they can immediately infer what the hidden objective was, they can learn about the dynamics of environment and they can generalize all this information to many other situations and tasks. This is what we call imitation learning.

In the meta-RL context, the natural way to learn to perform imitation learning is to create a problem (a set of MDPs) in the following way: At the beginning of each task (each MDP) the agent is given an example as a sequence of images and actions, then the agent is left to act on the MDP and is trained with a standard meta-RL algorithm. If the information provided in the example is fundamental to achieving high rewards, the agent will be forced to find a policy that can perform imitation learning. Of course, this "imitation policy" will only to generalize if we train it on a large enough variety of MDPs.

Note that it is irrelevant whether the examples are provided from a first person view or from third person view. They could even be given by a radically different medium, like text or sound. We could also not feed the action sequence of the example and only feed the images. This approach is extremely flexible. More importantly, we have broadly extended the capabilities of the agent without adding any complexity to the RL algorithm, we only had to create the right task for the agent to learn.

In there lies the true power of meta-RL, if the tasks are designed properly, we can teach the agents any skills. We could, for example, use this principle to force the agent to learn language by giving the agent tasks that require true language understanding. Similarly, if playing a first task gave useful information to solve a second task, a meta-RL agent would learn to use such information, thus learning to perform knowledge transfer. To me it seems clear that this is the way to solve AI.

You might then ask: If we train a meta-RL agent into tasks that force it to learn all the necessary skills, will we have a truly intelligent agent? The answer is that the reinforcement agents that we have today would not be able to solve the necessary tasks. Many innovations are needed (like the ones I proposed in [here]({% post_url 2016-10-3-Solve_inteligence %}) and [here]({% post_url 2017-1-17-scaling_up_deep_rl %}) ). But once we have agents that are capable of solving those tasks the answer will be __YES__.
