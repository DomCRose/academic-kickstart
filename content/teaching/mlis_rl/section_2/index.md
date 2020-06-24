---
title: What is a state or decision worth?
linktitle: Value functions
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: Deterministic Reinforcement Learning
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 100
---

## Value functions
When making decisions it is natural to ask, to the best of your knowledge, how good it is to be in each state, or how good decisions made in each state are.
To be useful, this concept of quality or value must relate to how well the current goal is achieved under the decisions made in the future and the changes in the environment they induce: in other words, it must be given by the return that will result following that state or state-action pair, under the current policy and the environment dynamics.

More precisely, we define the return for a subset of the trajectory 
$$
\omega_t^T=\\{(s_{t'},a_{t'},r_{t'})\\}_{t'=t}^T,
$$
as we define the return for the whole trajectory
$$
R\left(\omega_t^T\right)=\sum_{t'=t+1}^{T}r_{t'},
$$
where the reward at time $t$ is not included as it was given in the transition to the time, not the transition following that time.
The **value** of a state is then given by the return of the trajectory following that state under the current policy
$$\tag{7}\label{state_value_return}
V_\pi(s)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,\pi,f},
$$
where the ``evaluated at'' notation (the line with a subscript) is used to indicate that states and actions in the return after $s$ are generated according to the current policy $\pi$ and environment $f$: that is, given $s_t=s$, subsequent actions and states are produced according to Eq. <a href=https://dominic-c-rose.netlify.app/teaching/mlis_rl/section_1/#mjx-eqn-policy>(3)</a> and <a href=https://dominic-c-rose.netlify.app/teaching/mlis_rl/section_1/#mjx-eqn-dynamics>(4)</a>, with the corresponding rewards in the return following from Eq. <a href=https://dominic-c-rose.netlify.app/teaching/mlis_rl/section_1/#mjx-eqn-rewards>(5)</a>.
The value of an action in a particular state is defined similarly, where
$$\tag{8}\label{state-action_value_return}
Q_\pi(s,a)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,a_{t}=a,\pi,f},
$$
where both the first state and its following action are now fixed, with the rest following from the policy and dynamics as indicated by the evaluation.
The functions $V_\pi$ and $Q_\pi$ are referred to as **state value** and **state-action value** functions respectively.

For example, consider the policy for particle walks defined by the arrows in Fig. [3(a)](#figure-figure-3), where we only consider states that can reach the target.
The values given to states by this policy are shown in Fig. [3(b)](#figure-figure-3), and the values for actions taken in each state, grey for up and white for down, are given in Fig. [3(c)](#figure-figure-3).

{{< 
figure src="example_policy_and_values.png" 
title="Figure 3" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
An example policy (a) and its corresponding state value (b) and state-action value (c) functions. Only odd states at odd times and even states at even times are considered, as these are the only states which can reach the target x=0 at t=3. For the state-action values in (c), each group describes the values of actions in the state in the corresponding position, with states at the final time excluded since no action can be taken; white squares encode the value of moving down, while grey squares encode the value of moving up.
</p>

The interpretation of the state-action value is worth considering: in it, depending on what action we consider, we calculate the value of taking an action in a state which could be different from that which would be taken under the current policy; however, actions after this step are then taken according to the current policy.
This is useful as it allows us to estimate the effect of changing the policy in the current state: if we calculate that an action is more valuable than the one which would be taken under the current policy, then it indicates that we could improve the policy by changing it such that this is the action we take.
Changing the policy in turn changes the values of states and state-action pairs, which may cause further differences between the current policies action and the most valuable action in each state.

## Relating values: Bellman equations
The discussion at the end of the previous section is suggestive of a simple iterative procedure in which we calculate the values of the actions in each state for the current policy, update the policy according to those values, and repeat the process until the policy ceases to change.
While this would work, it can be inefficient, as the current statement of the values would indicate that we need to consider the entire trajectory following each state or state-action pair for each update, a substantial undertaking in complex problems.

This can be improved by noting that the values of each state and state-action pair are closely related: clearly, the value of a state is equal to the value of the following state under the current policy and environment dynamics, plus the reward received in between; the value of an action in a state is the value of the state resulting from that action under the environment dynamics, plus the reward received in between.
Mathematically, this follows from the definitions of values in Eq. \eqref{state_value_return} and \eqref{state-action_value_return} and is written as
$$
\begin{align}
V_\pi(s)&=r\left(f[s,\pi(s)],\pi(s),s\right)+V_\pi(f[s,\pi(s)]),\tag{9}\label{state-state_bellman}\\\\
Q_\pi(s,a)&=r\left(f[s,a],a,s\right)+V_\pi(f[s,a]),\tag{10}\label{state-action-state_bellman}
\end{align}
$$
where subsequent states and actions have been written as the actions of the policy and environment on the inputs.
Equations such as these are referred to as **Bellman equations**, and encode how state and state-action values are related.

## Optimality and improvement
In order to better understand how we can use these value functions to optimize the policy, it is worth considering the value functions for an optimal policy, one which maximizes the return following all initial states.
Such a policy maximizes the value of all initial states, since these are simply the possible returns of each episode under the current policy.
It must therefore maximize the value of all states: if it did not, then there is a state for which there exists a policy that assigns it a higher value, and therefore by the Bellman equation achieves higher values for all states which lead to this state.
This implies that the new policy also improves the value of at least one initializing state, and thus increases the return the policy achieves on an initial state, contradicting the assumption that the policy maximized returns.

Value functions thus provide a way of ordering policies.
A policy is said to be better than or equal to another, $\pi'\geq\pi$, if it provides an equal or greater value for every state, 
$$\tag{11}\label{value_policy_ordering}
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
$$\tag{12}\label{bellman_optimal_value}
V_*(s)=\max_a\left\\{r\left(f[s,a],a,s\right)+V_*(f[s,a])\right\\}.
$$
For finite problems the coupled set of equations for the values given by Eq. \eqref{bellman_optimal_value}, referred to as a **Bellman optimality equation**, has a unique solution for any reward function $r$ and environmental dynamics $f$: solving this equation thus solves the problem of maximizing returns.
In reverse, given the values of some policy, by checking whether they satisfy Eq. \eqref{bellman_optimal_value} we can confirm whether the policy is optimal or not.

Phrasing the goal of maximizing returns in terms of the value functions as done in this section is suggestive of a simple approach for improving the policy: assuming accurate knowledge of the current policies value functions, we simply choose a new policy in which the value of every state is equal or greater than the current one.
As eluded to earlier, a straightforward strategy which achieves this is given by modifying the policy $\pi$ to a new policy $\pi'$ according to the state-action value function for $\pi$, such that
$$\tag{13}\label{policy_improvement}
Q_\pi(s,\pi'(s))\geq V_\pi(s)\quad\mathrm{for}\ \mathrm{all}\ s.
$$
That this leads to a definitively equal or better policy, i.e. that Eq. \eqref{value_policy_ordering} holds for $\pi$ and $\pi'$, is a result of the **policy improvement theorem**, which forms the conceptual basis for many of the algorithms of reinforcement learning.

For a simple example of its application, consider improving the policy in Fig. [3(a)](#figure-figure-3) using the state-action values in Fig. [3(c)](#figure-figure-3): choosing the better action for each state according to this information provides the policy in Fig. [4(a)](#figure-figure-4), with the new state values in Fig. [4(b)](#figure-figure-4).
This policy turns out to be optimal after only one attempt to improve it, however more generally we will find we need to improve our policy multiple times, especially if our values are estimates, rather than exactly correct.

{{< 
figure src="example_policy_improvement.png" 
title="Figure 4" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Example of improving a policy, where the policy in Fig. <a href=#figure-figure-3>3(a)</a> has been improved by changing the policy to take the best action according to Fig. <a href=#figure-figure-3>3(c)</a>. The resulting policy (a) has values given by (b). Note how the value of the state $s=(x=-1,t=0)$ has not gone to $1-p$ as suggested by the value of taking the the up action in this state according to Fig. <a href=#figure-figure-3>3(c)</a>, but has in fact gained the better value of $1$: this is a side effect of changing the policy in both $s=(x=-1,t=0)$ and $s=(x=0,t=1)$ at the same time, resulting in a greater improvement to the value of $s=(x=-1,t=0)$ than expected.
</p>