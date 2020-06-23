---
title: Basics of reinforcement learning
linktitle: Basics of reinforcement learning
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: Deterministic Reinforcement Learning
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---

### The aims and set up of reinforcement learning
The task which reinforcement learning (RL) aims to solve is that of making the best decision in order to achieve some objective; of controlling something optimally to achieve some goal.
To tackle such tasks, we start with the general set up of these problems, broken down into the main ingredients.

Firstly, we need the decision maker, referred to as the **agent**: for example, this could consist of a human, a robot, a dog, or any other entity we consider to be acting according to information it receives to achieve some goal.
Secondly, we need the situation in which the agent must achieve its objective, referred to as the **environment**: for example, for the human this could be a game and their opponents; for the robot it could be a factory and parts for a machine it is making; for the dog it could be a park and a stick it is trying to fetch.

To achieve its objective, the agent has the ability to exert some form of control over its environment through its **actions**: for the human playing a game, this could be to move a piece in a board game or press a button on a controller; for the robot in the factory, this could be to move its arms or adjust a gripper; for the dog in the park, this could be to run or to grab the stick in its mouth. 
For the agent to take effective actions, it must have some knowledge about the environment with which it makes its decisions according to, called the environments **state**: for the human playing a game; this could be the current set up of pieces on a board game, or a short-term memory of recent images on a computer screen; for the robot in the factory, this could be the location of objects it can interact with; for the dog in the park, this could be the location of the stick, itself and any obstacles.

{{< 
figure src="agent_environment_diagram.png" 
title="A diagram representing the decomposition of control problems in to an agent which makes the decision to take particular actions, and an environment influenced by those actions. These actions are taken according to information received by the agent from the environment in the form of states, while ideal actions in particular states are reinforced through rewards." 
lightbox="true" 
>}}

Inspired by simple models of animal learning in behavioural psychology, the idea of RL is to encode the objective in **rewards** or punishments, telling the agent whether its actions in a given state are good or bad.
These rewards are viewed as signals sent to the agent by the environment: for example, the dogs owner gives it treats for doing the right thing, or punishments for misbehaving; the human receives intrinsic satisfaction for success in games, via endorphins; the robot is instructed computationally as to whether it has made the right choices.
With this encoding, the task of RL then becomes to maximise the rewards, a less abstract concept than ``achieving the objective'', which can be considered generically in order to cover multiple problems with the same algorithms.

Once the agent takes an action, this clearly influences a change in the environments state: in general, multiple actions taken in a sequence of resulting states will be required to achieve the objective.
This feedback loop of actions and states, along with rewards, completes the general set up of RL problems, also referred to as control problems, and is outlined diagrammatically in Fig. \ref{agent_environment_diagram}.

### Mathematical formalism
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
$$\label{concise_trajectory}
\omega_0^T=\{(s_t,a_t,r_t)\}_{t=0}^T,
$$
where $\omega_0^T$ is used to represent the set of states, actions and rewards between and including times $t=0$ and $t=T$, indicated by the right side of Eq. \eqref{concise_trajectory}.
Note the first reward $r_0$ and last action $a_T$ in this notation are simply a side effect of the concise notation, since they do not happen in the actual trajectory, as seen in Eq. \eqref{trajectory}.

The dependence of these quantities -- the states, actions and rewards -- on the previous ones in the episode follows intuitively from the set up discussed above.
The action taken only depends on the current state of the system, according to some decision made by the agent.
When the **agent** makes the same decision every time it is in the same state, this decision can be encoded by a function $\pi$, such that 
$$\label{policy}
a_t=\pi(s_t),
$$
where $\pi$ is called the **policy**.
If we assume the environment is deterministic, such that it changes to the same state every time a particular action is taken in a particular state, transitions in the state of the **environment** can also be written as a function $f$ of the current state and action
$$\label{dynamics}
s_{t+1}=f(s_t,a_t),
$$
where we will refer to $f$ as the environments **dynamics**.
Finally, the reward at each time must depend only on the events around that time step: the states of the environment on either side of the transition, and the action taken to trigger that transition.
The most general **reward** is thus simply a function $r$ such that
$$\label{rewards}
r_{t+1}=r\left(s_{t+1},a_t,s_t\right),
$$
where punishments are simply negative values, and rewards positive.

Given some initial state $s_0$, the functions $\pi$, $f$ and $r$ completely determine the following trajectory $\omega_0^T$.
The goal of reinforcement learning then becomes finding a policy which maximizes the total reward, or **return**, from any given initial state
$$
R\left(\omega_0^T\right)=\sum_{t=1}^{T}r_t.
$$

### Example: walks on a chain
As a simple example, consider a particle hopping up and down on a chain running from positive to negative infinity, i.e. its position $x$ is in $\{...,-2,-1,0,1,2,...\}$.
At each time step, the agent is given the choice of moving the particle either up or down by one site: stated mathematically the actions and the change induced in the position are written
$$
a_t&=\pm1,\\
x_{t+1}&=x_t+a_t.
$$
The sequence of moves forming a trajectory is referred to as a walk of the particle.

Suppose we initiate an episode with the particle in some position $x_0$, and our goal is for the particle to be at $0$ at a precise time $T$, i.e. $x_T=0$, spending minimal time below $0$, i.e. a preference for $x_t\geq0$ at all times $t$.
Clearly, achieving this objective will require knowledge of both the position and the current time, so the environmental state the agent must be aware of consists of both, $s_t=(x_t,t)$.
The environment dynamics can then be written
$$
s_{t+1}=f(s_t,a_t)=(x_t+a_t,t+1).
$$

To set up this goal as a reward maximization problem, we could simply give some negative reward (i.e. a punishment) for going below zero, and a positive reward for reaching the target at the end, i.e.
$$\label{walker_excursion_reward}
r\left(s',a,s\right)=
\left\{\begin{array}{l@{\qquad}l}
1 & s'=(0,T) \\\\
-p & s'=(x,t)\quad\mathrm{with}\quad x\leq0 \\\\
0 & \mathrm{otherwise}
\end{array}\right.,
$$
where $p$ is some positive number.
Example walks are given in Fig. \ref{example_walks}, with the region of punishments shaded in red and the target marked with a cross; the return for one of these examples is given.

{{< 
figure src="example_walks.png" 
title="Examples of particle walks on a chain. The red walk, initiated from $x=-4$, has a return of $R=1-4p$, due to being in the punishing red region at 4 time steps (excluding the initial time), and ending at the target." 
lightbox="true" 
>}}

In this simple problem, the policies which maximize the total reward over a trajectory are immediately clear: a walk from such a policy is given by the upper example in Fig. \ref{example_walks}, initiated from position $x=2$.
If the particle is below zero, it must move up at every time step to minimize the punishment received; if it is at zero, it must move up in order to avoid punishment; while above zero, it can move either up or down as long as it stays low enough to leave sufficient time for it to move back to zero at time $T$ in order to receive the reward.
Notably, the value of $p$ will not change the policies which maximize the total reward, since these will all behave the same in or near the punishing region regardless of $p$.
However, the magnitude of this punishment relative to the reward for the target can have significant effects on what the agent learns to prioritize.
If the agent receives more substantial punishments when learning from experience, it could initially focus on avoiding negative positions more strongly, causing it to take longer to discover and learn about the reward at $s=(0,T)$.
In contrast, if the punishment is too weak it may take a long time for the agent to start avoiding negative positions.

Given this simple problem with well understood solutions, we next turn to the issue of how to distinguish between the quality different choices, in order to improve decision making.


## What is a state or decision worth?
### Value functions
When making decisions it is natural to ask, to the best of your knowledge, how good it is to be in each state, or how good decisions made in each state are.
To be useful, this concept of quality or value must relate to how well the current goal is achieved under the decisions made in the future and the changes in the environment they induce: in other words, it must be given by the return that will result following that state or state-action pair, under the current policy and the environment dynamics.

More precisely, we define the return for a subset of the trajectory 
$$
\omega_t^T=\{(s_{t'},a_{t'},r_{t'})\}_{t'=t}^T,
$$
as we define the return for the whole trajectory
$$
R\left(\omega_t^T\right)=\sum_{t'=t+1}^{T}r_{t'},
$$
where the reward at time $t$ is not included as it was given in the transition to the time, not the transition following that time.
The **value** of a state is then given by the return of the trajectory following that state under the current policy
$$\label{state_value_return}
V_\pi(s)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,\pi,f},
$$
where the ``evaluated at'' notation (the line with a subscript) is used to indicate that states and actions in the return after $s$ are generated according to the current policy $\pi$ and environment $f$: that is, given $s_t=s$, subsequent actions and states are produced according to Eq. \eqref{policy} and \eqref{dynamics}, with the corresponding rewards in the return following from Eq. \eqref{rewards}.
The value of an action in a particular state is defined similarly, where
$$\label{state-action_value_return}
Q_\pi(s,a)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,a_{t}=a,\pi,f},
$$
where both the first state and its following action are now fixed, with the rest following from the policy and dynamics as indicated by the evaluation.
The functions $V_\pi$ and $Q_\pi$ are referred to as **state value** and **state-action value** functions respectively.

For example, consider the policy for particle walks defined by the arrows in Fig. \ref{example_policy_and_values}(a), where we only consider states that can reach the target.
The values given to states by this policy are shown in Fig. \ref{example_policy_and_values}(b), and the values for actions taken in each state, grey for up and white for down, are given in Fig. \ref{example_policy_and_values}(c).

{{< 
figure src="example_policy_and_values.png" 
title="An example policy (a) and its corresponding state value (b) and state-action value (c) functions. Only odd states at odd times and even states at even times are considered, as these are the only states which can reach the target x=0 at t=3. For the state-action values in (c), each group describes the values of actions in the state in the corresponding position, with states at the final time excluded since no action can be taken; white squares encode the value of moving down, while grey squares encode the value of moving up." 
lightbox="true" 
>}}

The interpretation of the state-action value is worth considering: in it, depending on what action we consider, we calculate the value of taking an action in a state which could be different from that which would be taken under the current policy; however, actions after this step are then taken according to the current policy.
This is useful as it allows us to estimate the effect of changing the policy in the current state: if we calculate that an action is more valuable than the one which would be taken under the current policy, then it indicates that we could improve the policy by changing it such that this is the action we take.
Changing the policy in turn changes the values of states and state-action pairs, which may cause further differences between the current policies action and the most valuable action in each state.

### Relating values: Bellman equations
The discussion at the end of the previous section is suggestive of a simple iterative procedure in which we calculate the values of the actions in each state for the current policy, update the policy according to those values, and repeat the process until the policy ceases to change.
While this would work, it can be inefficient, as the current statement of the values would indicate that we need to consider the entire trajectory following each state or state-action pair for each update, a substantial undertaking in complex problems.

This can be improved by noting that the values of each state and state-action pair are closely related: clearly, the value of a state is equal to the value of the following state under the current policy and environment dynamics, plus the reward received in between; the value of an action in a state is the value of the state resulting from that action under the environment dynamics, plus the reward received in between.
Mathematically, this follows from the definitions of values in Eq. \eqref{state_value_return} and \eqref{state-action_value_return} and is written as
$$
V_\pi(s)&=r\left(f[s,\pi(s)],\pi(s),s\right)+V_\pi(f[s,\pi(s)]),\label{state-state_bellman}\\
Q_\pi(s,a)&=r\left(f[s,a],a,s\right)+V_\pi(f[s,a]),\label{state-action-state_bellman}
$$
where subsequent states and actions have been written as the actions of the policy and environment on the inputs.
Equations such as these are referred to as **Bellman equations**, and encode how state and state-action values are related.

### Optimality and improvement
In order to better understand how we can use these value functions to optimize the policy, it is worth considering the value functions for an optimal policy, one which maximizes the return following all initial states.
Such a policy maximizes the value of all initial states, since these are simply the possible returns of each episode under the current policy.
It must therefore maximize the value of all states: if it did not, then there is a state for which there exists a policy that assigns it a higher value, and therefore by the Bellman equation achieves higher values for all states which lead to this state.
This implies that the new policy also improves the value of at least one initializing state, and thus increases the return the policy achieves on an initial state, contradicting the assumption that the policy maximized returns.

Value functions thus provide a way of ordering policies.
A policy is said to be better than or equal to another, $\pi'\geq\pi$, if it provides an equal or greater value for every state, 
$$\label{value_policy_ordering}
V_{\pi'}(s)\geq V_{\pi}(s)\quad\mathrm{for}\ \mathrm{all}\ s.
$$
Specifically, this is referred to as a partial ordering, as it is possible for two policies to each be better than the other in a subset of states, in which case they are not ordered by according to Eq. \eqref{value_policy_ordering}.
Despite this, it is a sufficient ordering to say that there must exist at least one policy for which this holds true for all other policies, which we refer to as an **optimal policy** $\pi_*$, that is
$$
\pi_*\geq\pi\quad\mathrm{for}\ \mathrm{all}\ \pi.
$$
Even if there are multiple optimal policies, they share the same optimal state value function $V_*(s)$, given by
$$
V_*(s)=\max_\pi V_\pi(s)\quad\mathrm{for}\ \mathrm{all}\ s,
$$
along with the same optimal state-action value function $Q_*(s,a)$.

As any other value functions, those of optimal policies must satisfy the Bellman equations discussed earlier.
In the case of optimal policies, these take a particularly simple form which make no explicit reference to the policy, for example
$$\label{bellman_optimal_value}
V_*(s)=\max_a\left\{r\left(f[s,a],a,s\right)+V_*(f[s,a])\right\}.
$$
For finite problems the coupled set of equations for the values given by Eq. \ref{bellman_optimal_value}, referred to as a **Bellman optimality equation**, has a unique solution for any reward function $r$ and environmental dynamics $f$: solving this equation thus solves the problem of maximizing returns.
In reverse, given the values of some policy, by checking whether they satisfy Eq. \eqref{bellman_optimal_value} we can confirm whether the policy is optimal or not.

Phrasing the goal of maximizing returns in terms of the value functions as done in this section is suggestive of a simple approach for improving the policy: assuming accurate knowledge of the current policies value functions, we simply choose a new policy in which the value of every state is equal or greater than the current one.
As eluded to earlier, a straightforward strategy which achieves this is given by modifying the policy $\pi$ to a new policy $\pi'$ according to the state-action value function for $\pi$, such that
$$\label{policy_improvement}
Q_\pi(s,\pi'(s))\geq V_\pi(s)\quad\mathrm{for}\ \mathrm{all}\ s.
$$
That this leads to a definitively equal or better policy, i.e. that Eq. \eqref{value_policy_ordering} holds for $\pi$ and $\pi'$, is a result of the **policy improvement theorem**, which forms the conceptual basis for many of the algorithms of reinforcement learning.

For a simple example of its application, consider improving the policy in Fig. \ref{example_policy_and_values}(a) using the state-action values in Fig. \ref{example_policy_and_values}(c): choosing the better action for each state according to this information provides the policy in Fig. \ref{example_policy_improvement}(a), with the new state values in Fig. \ref{example_policy_improvement}(b).
This policy turns out to be optimal after only one attempt to improve it, however more generally we will find we need to improve our policy multiple times, especially if our values are estimates, rather than exactly correct.

{{< 
figure src="example_policy_improvement.png" 
title="Example of improving a policy, where the policy in Fig. \ref{example_policy_and_values}(a) has been improved by changing the policy to take the best action according to Fig. \ref{example_policy_and_values}(c). The resulting policy (a) has values given by (b). Note how the value of the state $s=(x=-1,t=0)$ has not gone to $1-p$ as suggested by the value of taking the the up action in this state according to Fig. \ref{example_policy_and_values}(c), but has in fact gained the better value of $1$: this is a side effect of changing the policy in both $s=(x=-1,t=0)$ and $s=(x=0,t=1)$ at the same time, resulting in a greater improvement to the value of $s=(x=-1,t=0)$ than expected." 
lightbox="true" 
>}}