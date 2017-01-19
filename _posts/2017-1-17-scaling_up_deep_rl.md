---
layout: post
title: Scaling up deep RL
excerpt_separator: <!--more-->
future: true
autoNumber: '"all"  '
---
The simplest and most intuitive way to perform multi-task learning in the RL setting is to create a "supertask" by selecting a random task each time an episode begins. Unfortunately, current RL methods do not perform well in this setting, which also means that they wont be able to scale to bigger, more challenging tasks. Although sharing representations among many tasks (and training over a lot more data) should benefit generalization, doing so augments the variance of the gradients. If each one of $$k$$ tasks requires $$n$$ samples to have a good enough gradient at the current learning rate, computing a good gradient of the supertask would require $$kn$$ samples or a learning rate $$k$$ times smaller. Clearly, this does not scale.

<!--more-->

## Scaling up to multiple tasks

The problem we are facing can be interpreted as updates to the policy messing up the policy at other places. One idea would be to construct a policy with the property that updates from states of one task do not change the policy over the states of a different task. How can we construct such policy?

From this description it would seem reasonable to construct the policy combining several sub-policies dependent on a common representation $$\pi(s,a) = \sum_{i} T_i(s)\pi^i_{\theta_i}(\psi_w(s), a)$$, where $$ \psi_w(s) $$ is the common representation over all tasks and  $$ T_i(s)$$ is $$1$$ if the state pertains to task $$i$$ and $$0$$ otherwise. If we denote $$S_i$$ as all the states for which $$T_i(s)=1$$, we can derive from the policy gradient theorem that:

$$ \nabla_{\theta_i} J = \sum_{s \in S_i} d(s) \sum_{a} \frac{\partial \pi^i_{\theta_i}(\psi_w(s), a)}{\partial \theta_i} Q(s,a) $$

$$ \nabla_{w} J = \sum_{i} \sum_{s \in S_i} d(s) \sum_{a} \frac{\partial \pi^i_{\theta_i}(\psi_w(s), a)}{\partial w} Q(s,a) $$

And intuitive explanation of this method is: Since the gradient of $$\theta_i$$ depend on $$n$$ times fewer states than the gradient of $$w$$, a $$n$$ times larger learning rate or $$n$$ times fewer samples can be used.

At the beginning, learning $$ \psi_w(s) $$ will require a lot bigger samples - and therefore more computation - or a smaller learning rate - slowing down learning. This difficulties, though, will probably be outweighed by the benefits. However harder learning $$\psi_w(s)$$ is, it will not be $$n$$ time slower or require $$n$$ times more computation. Also, all policies $$\pi_i$$ will benefit from representations learned over a lot more data that will generalize much better. These benefits will be especially important if after training on a set of games, a new game is added.

## Scaling up to more difficult tasks
I previously mentioned that if an agent cannot learn to act over multiple tasks it also wont scale to more challenging ones. This is made clear if you consider that most difficult tasks can be broken down into sub problems. If an agent needs to have conversations and has to perform motor control in a game, it would make sense that the gradients computed from states of conversation do not affect the part of the network that performs motor control and vice versa.

A policy could be constructed that depends on parts that are activated and deactivated depending on the sate this way.

$$\pi(s,a) = g_w(\sigma^1_{\omega_1}(h) f^1_{\theta_1}(h),..., \sigma^k_{\omega_k}(h) f^k_{\theta_k}(h))$$

Where, for the sake of clarity, I have used $$h$$ to represent the hidden state $$ \psi_w(s) $$ and where $$\sigma^i_{\omega_i}(h)$$ defines how much each $$f_i$$ should be activated - and, by extension, how much it should be updated.

Now we can make an equivalent derivation from the policy gradient theorem:

$$ \nabla_{\theta_i} J = \sum_s d(s) \sigma^i_{\omega_i}(h) \sum_{a} \frac{\partial \pi(s,a)}{\partial \theta_i} Q(s,a) $$

$$ \nabla_{w} J = \sum_{i} \sum_s d(s) \sigma^i_{\omega_i}(h) \sum_{a} \frac{\partial \pi(s,a)}{\partial f^i_{\theta_i}(h)} \frac{\partial f^i_{\theta_i}(h)}{\partial w}Q(s,a) $$

$$ \nabla_{\omega_i} J = \sum_s d(s) \sum_{a} \frac{\partial \pi(s,a)}{\partial \omega_i} Q(s,a) $$

We know that the learning rate of $$w$$ should be smaller than that of $$\theta_i$$. But by how much? Since the activations of $$f_i$$ depend on $$\sigma^i_{\omega_i}$$ - a learned function - the right way to balance the learning rates will be variable. Is there a principled way to find this balance, them?

We could maybe make the (very reasonable) assumption that the learning rate of a policy that acts over a state-space twice as big as another policy, should be two times smaller. In our case we could keep and estimate of $$\hat{\sigma}^i_{\omega_i} = \mathrm{\mathop{\mathbb{E}}}[\sigma^i_{\omega_i}(h)]$$ and set the learning rate of $$\theta_i$$ to be the base learning rate times $$(\hat{\sigma}^i_{\omega_i})^{-1}$$.

When training in this manner, the functions $$f_i$$ that help take better actions will get activated more. It is not clear, though, that the functions that do not contribute will get deactivated. We could add a new loss that minimizes $$\hat{\sigma}^i_{\omega_i}$$, allowing the updates to $$\theta_i$$ to be bigger and, by extension, increasing learning speed.

Another desirable property of $$\sigma^i_{\omega_i}$$ would be temporal consistency. It seems reasonable that if a part of the network is useful at time $$t$$ it will also be helpful at time $$t+1$$. By also adding $$(\sigma^i_{\omega_i}(h_t)-\sigma^i_{\omega_i}(h_{t+1})^2$$ to the loss we could drive $$\sigma^i_{\omega_i}$$ in that direction.

## Conclusions
Separating the network into parts that can be activated and deactivated differently is a promising approach to scaling up deep RL to the multi-task setting and to more challenging tasks. It could possibly be interpreted as specializing parts of the network and giving them different levels of plasticity.

The principle can be applied to many different network architectures, from networks with different policy heads acting on different tasks, to cases where different low level perception networks are activated depending on the task and anywhere in between.

---

I am focused on other projects and I have very little compute power, that is why I haven't implemented the idea. If anyone is interested, though, I would be happy to help and really interested in seeing the results. It should not be hard to implement - especially in multi-task version, where you already know a good activation function.
