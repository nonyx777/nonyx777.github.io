---
layout: post
title: "RL and Joints"
excerpt: "This blog is not specifically about rl, even though it's the center of it, this blog is an implementation of an rl environment called CartPole inside godot. It's more suitable to say this blog covers the general process of training an agent inside godot."
date: 2026-08-13 09:00:00 +0000
---


Recently I have been reading about reinforcement learning at my leisure time. I don't particularly remember the point I got
interested, but somehow I ended up writing a blog about it, or around it to be precise. This blog is not specifically about rl,
even though it's the center of it, this blog is an implementation of an rl environment called CartPole inside godot.
It's more suitable to say this blog covers the general process of training an agent inside godot. Why godot? you may ask,
I wanted the open endedness of it, and after discovering a plugin called godot-rl-agents which let's you use state of the art 
python libraries such as stablebaseline3, I was reassured that it was the right decision to make.
### High-level introduction to rl
Reinforcement learning as you certainy have heard before, is all about rewarding and punishing the agent based on its interaction
with its environment. Almost all current rl algorithms are based on something called Markov's decision process (MDP). MDP is a mathematical framework for sequential decision making under uncertainty. It uses a single digit as a reward signal and the future state depends only on
the current state and action. Those two things are the main reasons behind why MDP falls short at framing a complicated problem.    
One of the components of MDP is known as policy, it's concerned with choosing an action in a given state which maximizes reward, specifically
the cumulative future reward. The cumulative future reward is known as return. Which makes the ultimate goal
of MDP to find a policy that maximizes return.  
RL is categorized into two, model-free and model-based. I currently only have knowledge of model-free algorithms and that is what's going to be
discussed. Model-free rl algorithms build a model of the enviornment by trial and error, the experiences gained are stored inside a table for tabular based algorithms such as Q-learning, or in a form of weights for those based on deep neural networks such as DQN. As you have guessed,
those stored numerical representations of experiences get adjusted as the agent learns. 
Initially every action has the same probability of being picked, but in time as the policy gets better that changes.    

### Joints in Godot
Currently as of this date godot supports 5 3d joints, for the CartPole environment we only need two, namely hinge joint and slider joint.
Even though I chose to make it in 3d, their movement is still constrained to two dimensions. Meaning the slider joint constraints the cart to only move sideways for a limited distance, while the hinge joint constraints the pole to rotate around the z-axis making it a 2 dimensional rotation. 
Since both the cart and pole are rigidbodies and connected with the hinge joint, movement of the pole results in movement of the cart, so to
stabalize the cart whenever the pole moves, I adjusted their mass. One thing that's enabled by default in rigidbodies is sleep. When
movement is very subtle in order to save compute resource the physics engine halts the rigidbody, that should be disabled.  
### CartPole rl environment
My implementation of the cartpole rl environment is based on the paper Balancing a CartPole System with Reinforcement Learning by Swagat Kumar.
#### The joints I used
#### The algorithm I used to train the agent
#### The result