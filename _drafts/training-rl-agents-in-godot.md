---
layout: post
title: "Training RL agents in Godot"
excerpt: "This blog is not specifically about rl, even though it's the center of it, this blog is an implementation of an rl environment called CartPole inside godot. It's more suitable to say this blog covers the general process of training an agent inside godot."
date: 2026-08-13 09:00:00 +0000
---


Recently I have been reading about reinforcement learning at my leisure time. I don't particularly remember the point I got
interested, but somehow I ended up writing a blog about it, or around it to be precise. This blog is not specifically about rl,
even though it's the center of it, this blog is an implementation of rl environments, namely the CartPole rl problem inside godot. This blog will cover the two flavors of the problem, one where the agent learns to balance it from an unstable position where the pole initially starts from an upright orientation, and the other where the agent learns to swing up to balance the pole.	
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
Currently godot supports five 3D joints. For the CartPole environment we only need two, namely hinge joint and slider joint.
Even though I chose to make it in 3D, their movement is still constrained to two dimensions. Meaning the slider joint constraints the cart to only move sideways for a limited distance, while the hinge joint constraints the pole to rotate around the z-axis making it a 2 dimensional rotation. 
Since both the cart and pole are rigidbodies and connected through the hinge joint, movement of the pole results in movement of the cart, so to stabalize the cart whenever the pole moves, I adjusted their mass. One other thing to look out for is sleep being enabled by default. When
movement is very subtle in order to save compute resource the physics engine completely stops the rigidbody, that should be disabled.  
![Slider and Hinge joints]({{ '/assets/images/rl-and-joints/joint-config.gif'}}){: width="%" .image-center }	

### Balancing an upright pole
My implementation of this version of the cartpole rl environment is based on the paper Balancing a CartPole System with Reinforcement Learning by Swagat Kumar. This paper demonstrates how different kinds of rl algorithms affect the learning performance of the cartpole problem. They finally settled with DQN(Deep Q Network) with PER(Prioritized Experience Replay), stating the possibility of the problem being too simple for the more advanced algorithms, and the addition of PER to DQN had a significant positive impact on the learning performance.    
Previously I have mentioned MDP (Markov's Decision Process), and MDP needs state, action, and reward specified explicitly. Keep in mind that different resources might use state interchangably to refer to the observable part or both the unobserved and observed. In my case state will always refer to the observable part.     

Action = 2 numbers [ 0 and 1] [left and right]  
State = 4 numbers [ cart's x position, cart's x velocity, pole's angle with respect to y-axis, pole's angular velocity]     
Reward = 1 number [ +1 or -1]   

The episode termination conditions are as follows. If the angle of the pole aginst the vertical axis is greater than 30 degrees, if the cart is at a distance greater than 1 unit from the center, and finally if the episode is 500 actions long. The problem is considered solved when the average reward of the last 100 consecutive episodes is equal or greater than 195.

There is a difference between the reward system between my implementation and the theirs. On the paper the agent learns by choosing the action that will result in the highest future reward. So the agent is always positively rewarded even at failure, however it's followed by epsiode termination. This means if it had chosen a better action it could have had recieved more rewards, so the next time it will choose an action which brings more rewards. That is the logic behind their reward system.   
My agent failed to learn with that reward system. I don't know why, but it might be important to know that I'm using a different algorithm to that of the paper's. I'm using what's known as Policy Proximal Optimization (PPO).    
As a result after tweaking the reward system to punish the failure by giving a negative reward, it started to learn.   

```{python}
	func termination_conditions() -> bool:
		var pole_angle: float = ai_controller.pole_angle
		var cart_pos_x: float = ai_controller.cart_pos_x
		
		var pole_failed: bool = abs(pole_angle) > MAX_POLE_ANGLE
		var cart_failed: bool = abs(cart_pos_x) > MAX_CART_DIST
		var time_limit: bool = ai_controller.n_steps >= MAX_NUM_STEPS
		
		
		if pole_failed or cart_failed:
			ai_controller.reward = -1.0
		else:
			ai_controller.reward = 1.0
		
		current_episode_return += ai_controller.reward
		
		if pole_failed or cart_failed or time_limit:
			check_if_solved()
			ai_controller.reset()
			reset_values()
			return true
		return false
```

I used multi-agent training instead of training with one environment instance. Training multiple agents at once helps in diversifying training data, stability, and reducing the wall-clock time of the training.  

![cp agent training]({{ '/assets/images/rl-and-joints/cp-agent-training.gif'}}){: width="%" .image-center } 	

A suprising thing I noticed is that the agent can learn even when your reward system is slightly off. For instance I had accidentally made it so that it punishes the agent everytime it survived to perform a certain number of actions. Now it is indeed not significant compared to the number of rewards it gets before that, however since the number sqeezed out of the neural network satisfies the evaluation, it learns.  

![cp agent trained]({{ '/assets/images/rl-and-joints/cp-trained-agent.gif'}}){: width="%" .image-center } 	

### Swing up and balance
This part of the solution is based on the article Reinforcement learning approach to control an inverted pendulum. The article addresses the problem on both virtual and physical space, it refers to them as simulation and experiment respectively. They used both Q-learning and DQN to train the agent, with DQN coming on top as expected. They covered several aspects of the problem starting from the physical model of the system to the influence of parameters on the simulation. In general it talks about the timestep it took for the algorithms to learn, and how the parameters affected it. It also has a good introduction to rl with descriptions and explanations, making it a good introductory material as well.  
The episode termination conditions are as follows:
1. Cart moving beyond the specified distance from the center. The agent recieves a massive -400 reward deduction.
2. Pole rotating more than 200 rad/sec.
3. Episode is 800 timesteps long.	

The reward system uses a normalized return value. Meaning the summation of the rewards for a single episode is divided by 800(the specified episode length). The agent is doing great when the return value is close to 1.
This is the reward function used in the article  
$r(\theta,x) = (1/2)(1-\cos(\theta))-(x/x_0)^2$  
The function equals 1 when $\theta = \pi$ and $x = 0$. $\theta$ is relative to -y-axis unlike the first version of the problem. The problem is considered solved if the average return of 100 consecutive episodes is greater than 0.85.  

In the first version we used $(\theta, \dot{\theta}, x, \dot{x})$ as the state. Now we're using $(\sin(\theta), \cos(\theta), \dot{\theta}, x, \dot{x})$. This is because in the first version the angle range was limited, it was +/- 30 with respect to the y-axis. In this one the pole can rotate freely, so since angles have a behavior of wrapping around, we can't use the raw angle as state component. Using $\sin$ and $cos$ instead, gives us the correct and unambiguous input for our training.
```{python}
	func reward_func(angle: float, pos_x: float) -> float:
		var n: float = 0.5 * (1.0 - cos(angle))
		var d: float = pow(pos_x / MAX_CART_DIST, 2.0)
		return n - d
	
	func termination_conditions() -> bool:
		var pole_angular_velocity: float = ai_controller.pole_angular_velocity
		var cart_pos_x: float = ai_controller.cart_pos_x
		var pole_angle: float = ai_controller.pole_angle
		
		var pole_failed: bool = abs(pole_angular_velocity) > MAX_POLE_ANGULAR_VELOCITY
		var cart_failed: bool = abs(cart_pos_x) > MAX_CART_DIST
		var time_limit: bool = ai_controller.n_steps >= MAX_NUM_STEPS
		
		
		if cart_failed:
			ai_controller.reward = -400
		else:
			ai_controller.reward = reward_func(pole_angle, cart_pos_x)
		
		current_episode_return += ai_controller.reward
		
		if pole_failed or cart_failed or time_limit:
			current_episode_return /= MAX_NUM_STEPS
			check_if_solved()
			ai_controller.reset()
			reset_values()
			return true
		return false
```
Before getting started with training I have found it better to start with attaching a manual controller script to the object that's going to be controlled by the agent, and try to have some notion of the level of difficulty the task in hand can be or if it is even possible with the physical attributes set for the environment. Because in virtual environments our assumption of how the environment should behave might not be correct because of how different platforms are set up, hence it may result in waste of training compute and time.  
As you can imagine this took more time to train than the prior, it took a total of more than half a million timesteps for the agent to be able to swing up and balance the pole.

### How the godot-rl-agents plugin work
