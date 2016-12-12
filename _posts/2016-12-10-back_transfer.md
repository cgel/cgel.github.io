---
layout: post
title: On positive back transfer
excerpt_separator: <!--more-->
autoNumber: '"all"'
---

Curriculum learning can help if the representations learned in previously-played, easier tasks are close to the representation that the agent is trying to learn now. It can be used by the standard RL algorithms, and it has proven to work in a variety of occasions. There is, though, a apparently related concept that is significantly more challenging to achieve, positive back transfer. It is the ability to improve your performance in a task after training in another. A clear case of this is how improvements of motor skills immediately generalize to all previously learned tasks. How can positive back transfer be brought to RL? Here are my thoughts on it.
<!--more-->

For the sake of simplicity lets consider a reduced version of the positive back transfer problem. There are two related tasks, $$t_1$$ and $$t_2$$, played in order. Taking some inspiration from progressive nets, we have two policies $$\pi_1$$ and $$\pi_2$$ for their respective tasks, both depending on a common inner representation $$h$$. We want the performance of $$\pi_1$$ to have improved after training on $$t_2$$.

The reason positive back transfer does not occur in naive RL, is that there is no reason why the changes in $$h$$ caused by training in $$t_2$$, should affect positively $$\pi_1$$. The effects these changes have $$\pi_1$$ are completely unpredictable.

I only see one way to achieve positive back transfer. And that is to carefully choose an $$h$$ with the specific property of having a measure of performance independent of the particular task. Consider the case where $$h$$ is a generative model of the world and the policies use it to do planing. A generative model can be objectively better by predicting the future more accurately. If the world is the same in $$t_1$$ and $$t_2$$, training in $$t_2$$ could help improve $$h$$ and, by extension, improve $$\pi_1$$. Again, the reason positive back transfer could occur in this case is that the optimal $$h$$ for one task is also optimal for all the others.

We can now go back to the initial example of motor control. Here, to achieve positive back transfer, we could try to find a action representation optimal for all tasks. The policies $$\pi_1$$ and $$\pi_2$$ would output a velocity vector $$v$$ instead of an action, and a motor policy $$\pi_m(v)$$ could give the actual action, with the goal of having the agent match $$v$$. Since the optimal $$\pi_m$$ is independent of the particular task, any improvements will generalize to all other tasks.

Of course, this idea presents new problems and questions.

* If the generative model or $$\pi_m$$ are imperfect, $$\pi_1$$ and $$\pi_2$$ could learn to depend on such imperfections. Consequently, when this inaccuracies are latter solved, the policies might not benefit. How big is this problem? Are there mathematical grounds to tackle it?
* Generative models seem like a very limiting set of state representations. What other forms of $$h$$ can benefit from positive back transfer?
* It seems that, to train $$\pi_m$$, we need a hand made loss. Could this be learned end-to-end? This is, of course, related to the problem of learning hierarchies.
