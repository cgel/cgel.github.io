---
layout: post
title: Is deep learning the future of UAV vision
---

**This essay was first posted on <http://www.diydrones.com>. It was inspired by other recent posts about the efforts to bring obstacle avoidance to the PX4 drone platform and the paper [End to End Learning for Self-Driving Cars](https://arxiv.org/abs/1604.07316) by NVIDIA.**

[This post](http://diydrones.com/profiles/blogs/px4-update-obstacle-avoidance-talk-at-dronecode-unconference) details the current efforts to develop a obstacle avoidance system for drones. They are based on Simultaneous Location And Mapping (building a map of the world in real time) and Path Planning -- For the sake of conciseness I will call this approach SLAMAP.

 I want to offer some thoughts on why this might not be a very promising approach, and propose an alternative that i think is more interesting.

One of the major shortfalls of SLAMAP is its inability of handling dynamic objects (objects that move)
![octomap]({{ site.url }}/assets/octomap.png)

For example, the block on the middle of this 3D map could either be a still box, or a vehicle moving at full speed towards the drone. Just take a moment to think how fundamental the perception of motion is to your ability of moving around or driving. Think about how would it be to have no information about the velocity of the objects around you.

Another problem is the binarity of the information stored in 3d maps — The cell is either empty or solid, seen or unseen.
![snow_wall]({{ site.url }}/assets/snow_wall.png)

Look at this image. To Assume that the cloud of snow powder is a solid wall is a huge constraint on the path of the drone. If the drone were following the snowboarder from behind, it would be forced to suddenly stop.

The idea of seen and unseen is also very limiting. If a building you have previously seen is now out of line of sight it’s reasonable to assume it hasn’t moved. But you cannot say the same about a person or vehicle. Similarly you can’t assume that there is nothing in every place you looked and saw nothing. — This relates to the lack of understanding of dynamic objects.

What I am arguing here is that to successfully interact with the environment, a drone needs a semantic understanding of the world (materials, physics etc) and ability to handle uncertainty. SLAMAP can’t do that.

Another difficulty with SLAMAP is that the utilization of this framework to provide the desired functionality is not trivial. Path planning solves the problem of finding a feasible route from A to B. But following a target is not the same problem. Reframing the problem to track a target, while also filming beautiful smooth footage and avoiding obstacles is very hard.

And finally there is a very empirical argument against SLAMAP. After decades of research it seems to have failed at finding applications outside academia. Most industrial applications in which SLAMAP is used, are simple and highly controlled — nothing like drone flying.

In short, the shortfalls of SLAMAP are:
Does not support dynamic objects
Does not handle uncertainty
Does not have semantic understanding of the world
Is difficult to get the desired behaviours
Empirically, it has been around for quite a while and hasn’t been very successful.


So, is there another option? Yes there is, and is called Deep Learning.

The idea behind deep learning is drop all the handmade parts of a system, and replace them all with a single neural net. This neural net can then be trained with examples of how to do a task, so that it learns the desired behaviour.

So, instead of having a stereo visión block, sparse SLAM block, dense octomap bock, path planning block, etc, now there would be a single neural net. To train it to control a drone you could use two basic methods. Give it footage of a person flying it or simply tell it if it’s doing a good job or not — or a mixture of the two.

Deep learning has proven incredibly successful in a wide variety of tasks. One of the earliest and most important is object recognition. This year deep nets outperformed humans on the ImageNet challenge — classifying an image among a 1000 classes — achieving an error rate of 3.5%. For perspective, humans are estimated to score around 5.5% while the best non deep learning approaches get arround 26%.

The state of the art speech recognition systems are also based on deep learning. See this post by google.

 Deep learning has also been used to outperform the current systems in video classification, handwriting recognition, natural language processing, human pose estimation, depth estimation from a single image and many others.

And it has also been applied to broader tasks outside classification. One of the most famous examples is the deep net that learned to play atari games, sometimes at a superhuman level.

And of course Alphago, a go player that recently beat the go master Lee Sedol 4-1. A feat that was thought by many to be decades away.

Very recently NVIDIA published a paper of an end to end steering system for a car. It showed a simple deep net — so simple that i was amazed — with only 70 hours of driving experience, running on a mobile GPU at 30 fps perform very well at driving on all kinds terrains and weathers.

But aside from off the empirical success of deep learning, the reason i believe it is more promising than SLAMAP is that it has the capacity to understand all the things SLAMAP cannot. All of the inherent limitations of SLAMAP i previously mentioned don’t exist in a deep net.

A deep net can learn to understand a dynamic world – tell the difference between a truck moving at 100 mph and at rest. And they can also learn meaningful semantics like: that snow powder is nothing to worry about, but that water is dangerous. And it can then learn how to use  this understanding


It might seem too good to be true. But would it really be that surprising that the methods that succeed were based on the only machine that can currently do this task — the brain.

I am pretty sure that with the available mobile hardware, deep learning frameworks and sea of freely accessible research, a decent team in less than a year would develop a better system than SLAMAP would ever lead to.
