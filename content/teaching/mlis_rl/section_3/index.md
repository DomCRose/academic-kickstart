---
title: "Solving a decision problem: dynamic programming"
linktitle: Dynamic programming
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: "det_rl"
    weight: 4

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 4
---

## Policy evaluation
In order to get an idea for how to improve a policy, we first need an efficient approach for evaluating the current policy, in other words, estimating the state and/or state-action values in order to guide our update.
While we could calculate all of the values using the definition in terms of returns, considering the full future of each state, this would be very time consuming.
Clearly we should take advantage of the Bellman equations, however we need a generic procedure.
Previously, in our example and the exercises, we took advantage of knowing the terminal or end states, and that we knew which states lead to these: we could then use the fact that their value must be zero to propagate the exact values of each state back in a single pass over the state space.
For a generic problem it may not be obvious which states lead to a particular state, depending on the complexity of the environment, and thus this backward-view of the updates may not be possible.
Instead, we adopt for an iterative procedure based on the Bellman equations. 

For example, to update the state value function, we will start from some arbitrary value function $V_0(s)$, up to the fact that the value of end states, such as those with $t=T$ in the walker examples, have zero value.
We then update the values of all states according to
$$\begin{align}\label{policy_evaluation_equation}\tag{3.1}
V_{k+1}(s)&=r\left(f[s,\pi(s)],\pi(s),s\right)+V_{k}(f[s,\pi(s)]).
\end{align}$$
Clearly the value function $V_\pi(s)$ for $\pi$ is a fixed point for this equation.
For problems we consider, the sequence $V_0,V_1,V_2...$ can be proven to converge to $V_\pi$ in a finite number of steps.
Intuitively, this can be understood to result from accurate information about the value of later states propagating backwards over time: the value of states which lead to an end state under the current policy is given purely by the reward gained from the transition, and are thus accurate after one update.
As the value function is iterated, the values which are accurate propagate back one step at a time.
Since we are considering problems where the trajectories end at some finite time, the maximum number of updates needed to converge is then simply the largest end time for a given initial state.

Practically, a naive implementation of the above algorithm would involve two arrays, one containing the current values, and an empty one filled with the results of Eq. \eqref{policy_evaluation_equation}.
A more memory efficient approach would be to instead use a single array, updating each element according to Eq. \eqref{policy_evaluation_equation}: however, this would result in some values being updated with other values which have already been iterated, depending on the order in which they are updated.
So long as this order includes every state at least once, these so-called ``in-line'' updates can be shown to still converge to $V_\pi$, often far faster in practice since they use data which is more up to date.
Each time every state is updated at least once is then referred to as a sweep, and the order of this sweep can have a significant affect on convergence rates.

## Example: policy evaluation for the walker
As an example, consider the initial walker policy as specified in Fig. \ref{example_policy_and_values}(a), with state values given by Fig. \ref{example_policy_and_values}(b).
Taking our initial values to be $V_0(s)=0$ for all $s$, lets consider taking sweeps across the state space, starting from $t=0$ then working through to $t=2$, going up from $x=-2$ to $x=1$ for each time.
While we perform these updates in-place, due to the structure of the state space given by the environment dynamics $f$, this sweep order is equivalent to taking two vectors and iterating Eq. \eqref{policy_evaluation_equation}.

{{< 
figure src="example_policy_and_values.png" 
title="Figure 1" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
An example policy (a) and its corresponding state value (b) and state-action value (c) functions. 
Only odd states at odd times and even states at even times are considered, as these are the only states which can reach the target x=0 at t=3.
For the state-action values in (c), each group describes the values of actions in the state in the corresponding position, with states at the final time excluded since no action can be taken; white squares encode the value of moving down, while grey squares encode the value of moving up.
</p>

Only tracking the value of states which can reach the target, the sequence of values given by subsequent sweeps is given by $V_1$, $V_2$ $V_3$ in Fig. \ref{example_policy_evaluation}.
In this case, convergence to the correct values given by Fig. \ref{example_policy_and_values}(b) is reached after only three iterations, the maximum number of steps needed for initial states at $t=0$ to reach an end state.
Despite this, we can converge even faster by considering an alternative sweep order: suppose we instead work backwards, starting from states with $t=2$ and decreasing to $t=0$, again going from $x=-2$ to $x=1$ for each time.
Under this order, due to the use of in-place updating rather than two separate vectors, we immediately jump from $V_0$ to $V_3$ after a single sweep, achieve far faster convergence.

{{< 
figure src="example_policy_evaluation.png" 
title="Figure 2" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Evaluation of the policy in Fig. \ref{example_policy_and_values}(a).
Under the first sweep described in the text, we have three iterations indicated by the solid arrows, from $V_0$ through to $V_3$, where the values have converged.
The alternative sweep instead follows the dashed line from $V_0$ straight to $V_3$, due to the use of in-place updates.
</p>

## Policy iteration
Once we have the values for the current policy $\pi$, we then use these values to construct a new policy.
The policy improvement theorem discussed before implies that as long as we construct a policy $\pi'$ from the value functions of $\pi$ such that 
$$\begin{align}\label{policy_improvement}\tag{3.2}
Q_\pi(s,\pi'(s))\geq V_\pi(s)\quad\mathrm{for}\ \mathrm{all}\ s,
\end{align}$$
holds, the resulting policy will achieve an equal or higher value for all states.
More precisely, given estimates $V(s)$ for the state values achieved after a sufficient number of sweeps, we construct $\pi'$ such that
$$\begin{align}\label{policy_iteration_state_values}\tag{3.3}
r\left(f[s,\pi'(s)],\pi'(s),s\right)+V(f[s,\pi'(s)])\geq V(s),
\end{align}$$
where the left side results from the Bellman equation for state-action values in terms of state values
$$\begin{align}
Q_\pi(s,a)=r\left(f[s,a],a,s\right)+V_\pi(f[s,a]).\label{state-action-state_bellman}\tag{3.4}
\end{align}$$

The simplest policy satisfying this condition is the \textbf{greedy} policy, the one which takes the action of maximum value.
This is given by the action which maximizes the left side of Eq. \eqref{policy_iteration_state_values}, i.e.
$$\begin{align}\label{greedy_policy}\tag{3.5}
\pi'(s)=\underset{a}{\operatorname{argmax}} \left\\{r\left(f[s,a],a,s\right)+V_K(f[s,a])\right\\}.
\end{align}$$
If multiple actions possess the same value, any is equally justified, and any strategy for selecting one for the policy can be used: for example, if the current action has maximum value, we could keep that, otherwise choosing a new maximum value action at random.

<style type="text/css">
    ol { list-style-type: lower-alpha; }
</style>

As discussed before, updating the policy will in turn change the values of states and actions.
These new values could in turn show that the policy can be further improved.
In order to reach optimality, we then apply the two steps described above, policy evaluation and policy improvement, repeatedly until the policy ceases to change.
At each stage the policy may only be slightly modified, in which case the best values to start from are those of the previous policy, since the value function likely changes little.
We thus have a simple algorithmic procedure for finding the optimal policy as follows:
1.  Initialize the policy $\pi$ and value function $V$.
2.  Choose a sweep order for this evaluation:
	1.  Perform a sweep over the state space, calculating
		$$\begin{align}
		V(s)&=r\left(f[s,\pi(s)],\pi(s),s\right)+V(f[s,\pi(s)]),
		\end{align}$$
		for each state in the order prescribed.
	2.  Repeat until the value function remains the same after a sweep.
3.  Construct the greedy policy as a described, based on Eq. \eqref{greedy_policy}.
4.  If $\pi'=\pi$, terminate. Else, repeat steps 2 and 3 with $\pi=\pi'$, 
	beginning evaluation from the final values of the previous evaluation.
In the case of our running example, having evaluated the policy in Fig. \ref{example_policy_and_values}(a) as demonstrated in Fig. \ref{example_policy_evaluation}, applying step 3 of this algorithm immediately leads to the optimal policy in Fig. \ref{example_policy_improvement}(a).
Repeating steps 2 and 3 as required will thus show that $\pi=\pi'$ on the second pass, and thus terminates the program.