---
layout: post
title: "RL and Joints"
excerpt: "This blog is not specifically about rl, even though it's the center of it, this blog is an implementation of an rl environment called CartPole inside godot. It's more suitable to say this blog covers the general process of training an agent inside godot."
date: 2026-08-13 09:00:00 +0000
---


Recently I have been reading about reinforcement learning at my leisure time. I don't particularly remember the point I got
interested, but somehow I ended up writing a blog about it, or around it to be precise. This blog is not specifically about rl,
even though it's the center of it, this blog is an implementation of rl environments, namely CartPole and Pendulum inside godot.
It's more suitable to say this blog covers the general process of training an agent inside godot. Why godot? you may ask,
because I like it, and after discovering a plugin called godot-rl-agents which let's you use state of the art 
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
Currently as of this date godot supports five 3D joints. For the CartPole environment we only need two, namely hinge joint and slider joint.
Even though I chose to make it in 3D, their movement is still constrained to two dimensions. Meaning the slider joint constraints the cart to only move sideways for a limited distance, while the hinge joint constraints the pole to rotate around the z-axis making it a 2 dimensional rotation. 
Since both the cart and pole are rigidbodies and connected with the hinge joint, movement of the pole results in movement of the cart, so to
stabalize the cart whenever the pole moves, I adjusted their mass. One other thing to look out for is sleep being enabled by default. When
movement is very subtle in order to save compute resource the physics engine completely stops the rigidbody, that should be disabled.  
### CartPole rl environment
My implementation of the cartpole rl environment is based on the paper Balancing a CartPole System with Reinforcement Learning by Swagat Kumar. This paper demonstrates how different kinds of rl algorithms affect the learning performance of the cartpole problem. They finally settled with DQN(Deep Q Network) with PER(Prioritized Experience Replay), stating the possibility of the problem being too simple for the more advanced algorithms, and the addition of PER to DQN had a significant positive impact on the learning performance.    
Previously I have mentioned MDP (Markov's Decision Process), and MDP needs state, action, and reward specified explicitly. Keep in mind that different resources might use state interchangably to refer to the observable part or both the unobserved and observed. In my case state will always refer to the observable part.     

Action = 2 numbers [ 0 and 1] [left and right]  
State = 4 numbers [ cart's x position, cart's x velocity, pole's angle with respect to y-axis, pole's angular velocity]     
Reward = 1 number [ +1 or -1]   

The episode termination conditions are as follows. If the angle of the pole aginst the vertical axis is greater than 12 degrees (I made it 30, no reason), if the cart is at a distance greater than 2.5cm (2.5 units long in godot. By convention 1unit = 1meter) from the center, and finally if the episode is 200 actions long (I made it 500, no reason again).

There is a difference between the reward system between my implementation and the theirs. On the paper the agent learns by choosing the action that will result in the highest future reward. So the agent is always positively rewarded even at failure, however it's followed by epsiode termination. This means if it had chosen a better action it could have had recieved more rewards, so the next time it will choose an action which brings more rewards. That is the logic behind their reward system.   
My agent failed to learn with that reward system. I don't know why, but it might be important to know that I'm using a different algorithm to that of the paper's. I'm using what's known as Policy Proximal Optimization (PPO).    
As a result after tweaking the reward system to punish the failure by giving a negative reward, it started to learn.   

```{python}
	var pole_angle: float = ai_controller.pole_angle
	var cart_pos_x: float = ai_controller.cart_pos_x
	
	var pole_failed: bool = abs(pole_angle) > MAX_POLE_ANGLE
	var cart_failed: bool = abs(cart_pos_x) > MAX_CART_DIST
	var time_limit: bool = ai_controller.n_steps >= MAX_NUM_STEPS
	
	
	if pole_failed or cart_failed:
		ai_controller.reward = -1.0
	else:
		ai_controller.reward = 1.0
	
	if pole_failed or cart_failed or time_limit:
		ai_controller.reset()
		reset_values()
		return true
	return false
```

I used multi-agent training instead of training with one environment instance. Training multiple agents at once helps in diversifying training data, stability, and reducing the wall-clock time of the training.   
A suprising thing I noticed is that the agent can learn even when your reward system is slightly off. For instance I had accidentally made it so that it punishes the agent everytime it survived to perform a certain amounts of actions. Now it is indeed not significant compared to the number of rewards it gets before that, however since the number sqeezed out of the neural network satisfies the evaluation, it learns.  