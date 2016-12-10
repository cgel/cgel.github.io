---
layout: post
title: On positive back transfer
excerpt_separator: <!--more-->
autoNumber: '"all"'
---

Curriculum learning can help learning because representations learned in previous played, easier tasks will be close to the representation that the agent is trying to learn now. There is, though, a apparently related concept that is significantly more challenging to achieve, positive back transfer. It is the ability to learn to perform better in previous tasks after learning a related task. The most clear case of this can bee seen in learning motor skills, where improvements immediately generalize to every task ever played. But when trying to bring positive back transfer to RL we face an apparently unsolvable problem.

<!--more-->

For the sake of simplicity lets consider a simplified version of the positive back transfer problem. There are two related tasks, t1 and t2, played in order. Will assume that there are two respective policies pi1 and pi2 that depend on a common inner representation h. We want for the performance on t1 to have improved after playing t2. The problem is that the the change in pi1 after changing h is completely unpredictable. There is no reason why a change in h wile performing t2 would impact positively pi1.

It seems to me that the only way to achieve positive back transfer is for the representation to have a measure of performance independent to the particular task. For example, if the inner representation is a "simulator", to predict the future, and pi1 uses it to solve t1, the simulator can be improved in t2 and pi1 will be able to use it more effectively. Again, the reason pi1 can improve after training in t2 is that a simulator can be objectively better, that it has a measure of performance. All policies dependent on a simulator will improve when the simulator improves. Of course, to improve a simulator both tasks must be in a similar domain.

Going back to the initial example of motor control we see how this idea could explain it. In this case we require another policy, a motor policy m_pi(v), where v is a vector indicating the desired direction and speed the agent should have. m_pi's objective is independent of the particular task, it does not matter if v will lead to high reward. It only cares about how well it tan match v. If pi1 depends on m_pi, and m_pi has improved in another task, pi1 will improve. It would be important to ensure that pi1 does not learn to work around the shortcoming of m_pi, because this shortcomings will hopefully disappear in future tasks.

I propose that the key to positive back transfer is to build policies that depend on objectively improvable representations or sub-policies.
