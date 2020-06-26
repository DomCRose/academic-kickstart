---
title: Exercises 1
linktitle: Exercises 1
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: "det_rl"
    weight: 3

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 3
---

## 1. Learning to walk
Consider the walks of length $T=6$ shown in Fig. \ref{example_walks},
{{< 
figure src="example_walks.png" 
title="Figure 1" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Examples of particle walks on a chain. The red walk, initiated from $x=-4$, has a return of $R=1-4p$, due to being in the punishing red region at 4 time steps (excluding the initial time), and ending at the target.
</p>
with the reward function
$$\label{walker_excursion_reward}\tag{1}
r\left(s',a,s\right)=
\left\{\begin{array}{l@{\qquad}l}
1 & s'=(0,T) \\
-p & s'=(x,t)\quad\mathrm{with}\quad x\leq0 \\
0 & \mathrm{otherwise}
\end{array}\right..
$$

1.  Calculate the return of the remaining three walks shown.
2.  Some of the walks shown are produced by the policy
    $$
    \pi(s)=
    \left\\{\begin{array}{l@{\qquad}l}
    1 & s=(x,t)\quad\mathrm{with}\quad x\leq-1 \\\\
    -1 & s=(x,t)\quad\mathrm{with}\quad x>-1
    \end{array}\right..
    $$
    Calculate the values of states which can be reached starting from $x=2,0,-2,-4$ under this policy in the $7\times7$ grid shown, either by considering the trajectories following each state or using the Bellman equations.
3.  Calculate the value of actions in states where the position $x=0$.
4.  How do these state-action values suggest the policy could be changed to 
    improve  the returns?
5.  Is this an optimal policy?

## 2. Describing a robot
A robot has three limbs, each with two joints.
For each limb, one joint can move in a plane like an elbow or knee, i.e. its position is described by a single angle $\theta_i$, where $i=1,2,3$ indexes the limb; the other joint can move spherically, like a shoulder or hip, i.e. its position is described by two angles $\phi_i,\psi_i$.
At each time step, it can choose to move all its joints a small amount
$$
\\{(\theta_i,\phi_i,\psi_i)\\}_{i=1}^3\rightarrow\\{(\theta_i+\Delta\theta_i,\phi_i+\Delta\phi_i,\psi_i+\Delta\psi_i)\\}_{i=1}^3.
$$
It is located in a closed room with a stairs; its position in space can be described by the position of its centre of mass $(x,y,z)$.
1.  What are the states of the environment and the actions of the agent?
2.  Without specific mathematical equations, what physics would be relevant to the change
    in environmental state at each time step.
3.  Suppose the objective is for the robot to reach the top of the stairs. 
    What structure could the reward function have in order for its maximization to encode this goal?
    How could different reward functions effect the learning process?

## 3. More Bellman equations
The Bellman equations considered previously are two of many possible ways of relating values.

1.  Using equations 
    $$\begin{align}
    V_\pi(s)&=r\left(f[s,\pi(s)],\pi(s),s\right)+V_\pi(f[s,\pi(s)]),\label  {state-state_bellman}\tag{2}\\\\
    Q_\pi(s,a)&=r\left(f[s,a],a,s\right)+V_\pi(f[s,a]),\label{state-action-state_bellman}\tag{3}
    \end{align}$$
    write an equation for the value of a state in terms only the state-action value.
2.  Using the answer to a) and Eq. \eqref{state-action-state_bellman} write an 
    equation for the value of a state-action pair in terms of a single reward and the  state-action value.
3.  Using the answer to b), write down a Bellman optimality equation for the  
    state-action value function in terms of itself, analogous to the equation
    $$\label{bellman_optimal_value}\tag{4}
    V_*(s)=\max_a\left\\{r\left(f[s,a],a,s\right)+V_*(f[s,a])\right\\}.
    $$ 
    for the state value function.

## 4. Deterministic policy improvement theorem
**(Hard)** Lets prove the deterministic version of the policy improvement theorem.
That is, lets prove that given two policies $\pi$ and $\pi'$ satisfying 
$$\label{policy_improvement}\tag{5}
Q_\pi(s,\pi'(s))\geq V_\pi(s)\quad\mathrm{for}\ \mathrm{all}\ s,
$$
then 
$$\label{value_policy_ordering}\tag{6}
V_{\pi'}(s)\geq V_{\pi}(s)\quad\mathrm{for}\ \mathrm{all}\ s,
$$
holds, and thus $\pi'\geq\pi$.
1.  Using a Bellman equation, expand Eq. \eqref{policy_improvement} to write an 
    inequality for the value of a state under $\pi$, in terms of the value of another state under $\pi$ where the state is the result of actions taken by the policy $\pi'$, and the reward in between time steps.
2.  Substitute Eq. \eqref{policy_improvement} into the result of part a) to arrive at a 
    new inequality for the value of a state under $\pi$ in terms of the state-action value under $\pi$ with action chosen according to $\pi'$, and the reward in between time steps.
3.  Repeating this process, show Eq. \eqref{value_policy_ordering} by induction.