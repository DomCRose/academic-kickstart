---
title: Introduction
linktitle: Introduction
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: Deterministic Reinforcement Learning
    weight: 1

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

## The aims and set up of reinforcement learning
The task which reinforcement learning (RL) aims to solve is that of making the best decision in order to achieve some objective; of controlling something optimally to achieve some goal.
To tackle such tasks, we start with the general set up of these problems, broken down into the main ingredients.

Firstly, we need the decision maker, referred to as the **agent**: for example, this could consist of a human, a robot, a dog, or any other entity we consider to be acting according to information it receives to achieve some goal.
Secondly, we need the situation in which the agent must achieve its objective, referred to as the **environment**: for example, for the human this could be a game and their opponents; for the robot it could be a factory and parts for a machine it is making; for the dog it could be a park and a stick it is trying to fetch.

To achieve its objective, the agent has the ability to exert some form of control over its environment through its **actions**: for the human playing a game, this could be to move a piece in a board game or press a button on a controller; for the robot in the factory, this could be to move its arms or adjust a gripper; for the dog in the park, this could be to run or to grab the stick in its mouth. 
For the agent to take effective actions, it must have some knowledge about the environment with which it makes its decisions according to, called the environments **state**: for the human playing a game; this could be the current set up of pieces on a board game, or a short-term memory of recent images on a computer screen; for the robot in the factory, this could be the location of objects it can interact with; for the dog in the park, this could be the location of the stick, itself and any obstacles.

Inspired by simple models of animal learning in behavioural psychology, the idea of RL is to encode the objective in **rewards** or punishments, telling the agent whether its actions in a given state are good or bad.
These rewards are viewed as signals sent to the agent by the environment: for example, the dogs owner gives it treats for doing the right thing, or punishments for misbehaving; the human receives intrinsic satisfaction for success in games, via endorphins; the robot is instructed computationally as to whether it has made the right choices.
With this encoding, the task of RL then becomes to maximise the rewards, a less abstract concept than ``achieving the objective'', which can be considered generically in order to cover multiple problems with the same algorithms.

Once the agent takes an action, this clearly influences a change in the environments state: in general, multiple actions taken in a sequence of resulting states will be required to achieve the objective.
This feedback loop of actions and states, along with rewards, completes the general set up of RL problems, also referred to as control problems, and is outlined diagrammatically in Fig. [1](#figure-figure-1).

{{< 
figure src="agent_environment_diagram.png" 
title="Figure 1" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
A diagram representing the decomposition of control problems in to an agent which makes the decision to take particular actions, and an environment influenced by those actions. These actions are taken according to information received by the agent from the environment in the form of states, while ideal actions in particular states are reinforced through rewards.
</p>

## Mathematical formalism
With this repeating sequence of actions, states and rewards, we clearly require some concept of time with which we can order events.
For simplicity, we will focus on problems which can be viewed in **discrete time**, which we label $t$: while the everyday world runs in continuous time, most problems can in practice be effectively solved by discretizing them with small enough time steps.

We label the **state** of the environment at time $t$ as $s_t$, and the **action** taken by the agent at this time $a_t$.
The **reward** received by the agent at this time is written as $r_t$.
The problems we consider are assumed to end at some point in the future, in which case they are called **episodic** problems, with time beginning at $0$ and ending at some potentially variable **end time** $T$.
The sequence of states, actions and rewards in each run of a problem forms a **trajectory** or **episode**, with each set of values following the previous, i.e.
$$\tag{1}\label{trajectory}
s_0,a_0\rightarrow s_1,r_1,a_1\rightarrow s_2,r_2,a_2\rightarrow...\rightarrow s_T,r_T,
$$
where the reward for taking the action $a_t$ in the state $s_t$, triggering transition to the state $s_{t+1}$, is given at the next time, $r_{t+1}$.
For clarity in later equations, we introduce a shorthand notation for a trajectory, specifically we write
$$\tag{2}\label{concise_trajectory}
\omega_0^T=\\{(s_t,a_t,r_t)\\}_{t=0}^T,
$$
where $\omega_0^T$ is used to represent the set of states, actions and rewards between and including times $t=0$ and $t=T$, indicated by the right side of Eq. \eqref{concise_trajectory}.
Note the first reward $r_0$ and last action $a_T$ in this notation are simply a side effect of the concise notation, since they do not happen in the actual trajectory, as seen in Eq. \eqref{trajectory}.

The dependence of these quantities -- the states, actions and rewards -- on the previous ones in the episode follows intuitively from the set up discussed above.
The action taken only depends on the current state of the system, according to some decision made by the agent.
When the **agent** makes the same decision every time it is in the same state, this decision can be encoded by a function $\pi$, such that 
$$\tag{3}\label{policy}
a_t=\pi(s_t),
$$
where $\pi$ is called the **policy**.
If we assume the environment is deterministic, such that it changes to the same state every time a particular action is taken in a particular state, transitions in the state of the **environment** can also be written as a function $f$ of the current state and action
$$\tag{4}\label{dynamics}
s_{t+1}=f(s_t,a_t),
$$
where we will refer to $f$ as the environments **dynamics**.
Finally, the reward at each time must depend only on the events around that time step: the states of the environment on either side of the transition, and the action taken to trigger that transition.
The most general **reward** is thus simply a function $r$ such that
$$\tag{5}\label{rewards}
r_{t+1}=r\left(s_{t+1},a_t,s_t\right),
$$
where punishments are simply negative values, and rewards positive.

Given some initial state $s_0$, the functions $\pi$, $f$ and $r$ completely determine the following trajectory $\omega_0^T$.
The goal of reinforcement learning then becomes finding a policy which maximizes the total reward, or **return**, from any given initial state
$$
R\left(\omega_0^T\right)=\sum_{t=1}^{T}r_t.
$$

## Example: walks on a chain
As a simple example, consider a particle hopping up and down on a chain running from positive to negative infinity, i.e. its position $x$ is in $\\{...,-2,-1,0,1,2,...\\}$.
At each time step, the agent is given the choice of moving the particle either up or down by one site: stated mathematically the actions and the change induced in the position are written
$$
\begin{align}
a_t&=\pm1,\\\\
x_{t+1}&=x_t+a_t.
\end{align}
$$
The sequence of moves forming a trajectory is referred to as a walk of the particle.

Suppose we initiate an episode with the particle in some position $x_0$, and our goal is for the particle to be at $0$ at a precise time $T$, i.e. $x_T=0$, spending minimal time below $0$, i.e. a preference for $x_t\geq0$ at all times $t$.
Clearly, achieving this objective will require knowledge of both the position and the current time, so the environmental state the agent must be aware of consists of both, $s_t=(x_t,t)$.
The environment dynamics can then be written
$$
s_{t+1}=f(s_t,a_t)=(x_t+a_t,t+1).
$$

To set up this goal as a reward maximization problem, we could simply give some negative reward (i.e. a punishment) for going below zero, and a positive reward for reaching the target at the end, i.e.
$$\tag{6}\label{walker_excursion_reward}
r\left(s',a,s\right)=
\begin{cases}
1 & s'=(0,T) \\\\
-p & s'=(x,t)\quad\mathrm{with}\quad x\leq0 \\\\
0 & \mathrm{otherwise}
\end{cases},
$$
where $p$ is some positive number.
Example walks are given in Fig. [2](#figure-figure-2), with the region of punishments shaded in red and the target marked with a cross; the return for one of these examples is given.

{{< 
figure src="example_walks.png" 
title="Figure 2" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Examples of particle walks on a chain. The red walk, initiated from $x=-4$, has a return of $R=1-4p$, due to being in the punishing red region at 4 time steps (excluding the initial time), and ending at the target.
</p>


In this simple problem, the policies which maximize the total reward over a trajectory are immediately clear: a walk from such a policy is given by the upper example in Fig. [2](#figure-figure-2), initiated from position $x=2$.
If the particle is below zero, it must move up at every time step to minimize the punishment received; if it is at zero, it must move up in order to avoid punishment; while above zero, it can move either up or down as long as it stays low enough to leave sufficient time for it to move back to zero at time $T$ in order to receive the reward.
Notably, the value of $p$ will not change the policies which maximize the total reward, since these will all behave the same in or near the punishing region regardless of $p$.
However, the magnitude of this punishment relative to the reward for the target can have significant effects on what the agent learns to prioritize.
If the agent receives more substantial punishments when learning from experience, it could initially focus on avoiding negative positions more strongly, causing it to take longer to discover and learn about the reward at $s=(0,T)$.
In contrast, if the punishment is too weak it may take a long time for the agent to start avoiding negative positions.

Given this simple problem with well understood solutions, we next turn to the issue of how to distinguish between the quality different choices, in order to improve decision making.