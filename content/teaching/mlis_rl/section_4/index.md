---
title: "Learning by doing: sampling with Monte Carlo"
linktitle: Monte Carlo
toc: true
type: docs
draft: false
menu:
  mlis_rl:
    parent: "det_rl"
    weight: 5

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 5
---

## Relevant states
So far, the algorithm we have considered treated each state in the state space equally.
In practice, this is often largely unnecessary, with the vast majority of a problems state space being irrelevant for its solution: our current method can therefore waste a substantial amount of computational resources.
There are two main reasons for which this occurs.

Firstly, the set of possible states can be restricted by the initial conditions, combined with the evolution prescribed by the environments dynamics.
For example, suppose in our walker example we only ever start in a small set of states near zero, say $x=-2,0,2$, and the target is at an end time of $T$.
Since the particle can only jump up or down $1$ place at each time step, there is a large number of states that can never be reached: the particles position $x$ can not move further than $t$ steps away from where it starts by time $t$, so for a state $s=(x,t)$ we must have $|x|\leq t+2$, with the precise bounds of each episode depending on the initial position.
This cone of states is sketched in Fig. \ref{relevant_states}.
Due to the initial condition, any states outside of this cone are **inaccessible** to the control problem, and thus calculating their values is a waste of resources.
While for this particular simple problem we could adapt our sweeps to account for this restriction, in many more complex problems the true space of **possible** states can not be found by hand.

{{< 
figure src="relevant_states.png" 
title="Figure 1" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
A sketch of the inaccessible, possible and relevant states for walks initiated from zero.
</p>
	
Secondly, and even more importantly, the relevant states are substantially less than the possible states: for a given initial policy, only a small subset of the state space will be visited.
In our walker example for some initial policy, suppose that the trajectories followed from the three initial positions $x=-2,0,2$ all cover unique states, but lie close to the axis: positions at all times are such that $|x|<c<<T$, where $c$ is some positive constant much less than the target time $T$, which is roughly the maximum distance that could be reached from the origin.
If we repeatedly evaluate only the values of states in the corridor in the $x-t$ plane with $|x|<c$, followed by updating the policy to the one greedy with respect to this evaluation, the actions will change to either move above zero, be drawn towards the target, or be left the same: the resulting trajectories will remain in this corridor.
As the policy is iterated the states **relevant** to the optimization, those visited by trajectories for each iteration of the policy and others nearby, thus consists of a very small subset of the possible states, sketched in Fig. \ref{relevant_states}.
Again, while we may adapt our dynamic programming procedure for this simple problem as described here, for a generic environment we will have no idea which states are relevant and which are not.

These different degrees of importance ascribed to each state suggest that an approach which naturally favours relevant states is desirable.

## Learning value through experience
To solve the problem of evaluating irrelevant states, we take another note from nature: we learn through experiences sampled from actual interactions with the environment, i.e. purely from the sequence of events in trajectories.
Theoretically, such an approach has an additional advantage, since sampled experience does not require the agent to have a precise knowledge of the inner workings of the environment, something we previously assumed.
This makes learning from experience a **model-free** approach, in contrast to a **model-based** approach such as dynamic programming.
Despite this lack of knowledge, it clearly isn't necessary for learning: an animal doesn't know a precise model of gravity, mechanics and electrodynamics necessary to understand how it can apply a force to a surface, yet can easily learn to walk across the ground.
Even if a model is available, for example in computer simulated environments, sampling experiences can help address the relevancy of states discussed earlier, since it automatically prioritizes the states which matter.

To see why this works, consider the definitions of the values in terms of returns.
As stated in 
$$\begin{align}\label{state_value_return}\tag{4.1}
V_\pi(s)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,\pi,f},
\end{align}$$
or
$$\begin{align}\label{state-action_value_return}\tag{4.2}
Q_\pi(s,a)=\left.R\left(\omega_t^T\right)\right|_{s_{t}=s,a_{t}=a,\pi,f},
\end{align}$$
we defined the value to be the return corresponding to the sequence of states and actions generated by the current policy $\pi$ and environment $f$ when initiated from a particular state $s$ or state-action pair $(s,a)$.
However, these returns are simply composed of the rewards given at each step, a signal we assume to be sent from the environment to the agent.
Thus, no knowledge of the environment dynamics is necessary to learn the value of each state, only the ability to generate experiences via interaction with the environment, including information about the resulting rewards.
For example, we can calculate the values of states along a trajectory, an experience the agent undergoes
$$\begin{align}
s_0,a_0\rightarrow s_1,r_1,a_1\rightarrow s_2,r_2,a_2\rightarrow...\rightarrow s_T,r_T,
\end{align}$$
by calculating
$$\begin{align}
V_\pi(s_t)=\sum_{t'=t+1}^Tr_{t'},
\end{align}$$
purely from the reward signals recieved.

## Changing decision making through exploration
Since we no longer assume the agents access to a model of the environment, the value of states can not be used to judge and choose alternative actions as done in Eq. \eqref{greedy_policy}, since we can't look ahead and see what state each action leads to.
To update the policy, we instead need direct access to the state-action values for the current policy, so that we can construct a new greedy policy as
$$\begin{align}\label{monte_carlo_greedy_policy}\tag{4.3}
\pi'(s)=\argmax_a Q_\pi(s,a).
\end{align}$$

Unfortunately, by only considering trajectories generated by the current policy, the agent will clearly learn nothing of the value of alternative actions to those it currently takes, since it never makes this alternative decisions.
To achieve this, and thus facilitate genuine learning through the adjustment of decision making, we need to incorporate some form of **exploration** in our approach.
This can be done by including some small probability $\epsilon$, called the exploration rate, that the action taken is not the current greedy action, but instead a random action: that is, given $\mathcal{A}(s)$ possible actions, which may depend on the current state $s$, the probability of $a$ given $s$ is
$$\label{epsilon-greedy}\tag{4.4}
P_\epsilon(a|s)=
\left\{\begin{array}{l@{\qquad}l}
1-\epsilon+\frac{\epsilon}{\mathcal{A}(s)} & a=\pi(s) \\
\frac{\epsilon}{\mathcal{A}(s)} & a\neq\pi(s)
\end{array}\right..
$$
The distribution $P(a|s)$ is often referred to an $\epsilon$-greedy policy when the current policy $\pi$ is greedy with respect to a value function.

Whenever such an exploratory action occurs, we can use the future trajectory from that point to calculate the value of the alternative action taken when the current policy is followed after that point.
However, within each trajectory we can only evaluate the last exploratory action taken, and any state-action pairs occurring after it: the future of any states or actions occuring before the last exploratory step is not completely generated by the current policy, and therefore does not provide an accurate description of their values.

{{< 
figure src="monte_carlo_exploration.png" 
title="Figure 2" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Example trajectories with exploratory steps.
The black lines indicate steps under the current policy, while the red dashed lines indicate exploration steps, combined indicating the trajectories followed by two particles initiated from $x=0$ and $x=-2$.
The grey dashed lines indicate segments of the trajectories that would be followed if the exploratory steps did not occur, until the policy causes these paths rejoin the same path followed by the actual trajectory.
</p>

As an example, consider the two trajectories in Fig. \ref{monte_carlo}.
For the trajectory initiated from $x=0$, the single exploratory step at $s=(0,6)$ results in the action to go up, $a=1$, instead of down as the policy would prescribe: the return following this action is $1$, and thus we have learnt that $Q_\pi\left[(0,6),1\right]=1$, in contrast to the current policies action $Q_\pi\left[(0,6),-1\right]=1-p$.
We can also use this exploratory step to learn the value of an action which the policy would have taken, going down in the state $(1,7)$, since the agent would only experience this by arriving in this state from an exploratory action: we thus have $Q_\pi\left[(-1,7),-1\right]=1$.

For the trajectory initiated from $x=-2$, there are two exploratory steps, with the last occurring at $s=(-1,9)$ where the action up was taken instead of down: this action tells us that $Q_\pi\left[(-1,9),1\right]=1$, compared to the policies current action $Q_\pi\left[(0,6),-1\right]=1-2p$.
The earlier exploratory step in the second trajectory at $(-1,3)$ can not be used to calculate new values, since its future was not generated entirely by the current policy, and thus the return after this point is not representative of this actions value.

Despite painting a dim picture for having multiple exploratory steps in one trajectory, they are a necessity to guarantee a chance of finding the best policy.
This is best demonstrated by an example: in Fig. \ref{multiple_exploratory}(a), with trajectories only initiated from $x=2$, if we only consider single exploratory steps then the agent will never find the reward associated to the target under the initial policy.
Instead, it requires at least two successive exploratory steps from make it from the initial state to the target.
After this, it will then learn that going down in $(1,1)$ is better then going up, resulting in the policy shown in Fig. \ref{multiple_exploratory}(b).
After subsequent exploratory actions, it will then learn to go down twice from the start to reach the target.
Put more generally, finding the best rewards possible, and thus the best ways to improve the policy, may require taking multiple actions which differ from those of the current policy.

{{< 
figure src="multiple_exploratory.png" 
title="Figure 2" 
lightbox="true" 
>}}
<p style="text-align: center; font-size:80%">
Example of the necessity of multiple exploratory steps.
In (a), for trajectories only initiated from $x=2$ and generated by the policy indicated by the arrows, the target at $(0,2)$ will never be found by single a single exploratory step.
Instead, two subsequent exploratory steps are required, going down from both $(2,0)$ and $(1,1)$ as indicated by the red dashed lines.
After updating the value of going down in $(1,1)$ and using the new greedy policy seen in (b), only a single exploratory step is required for the correct action to be discovered from $(2,0)$.
</p>

## Monte Carlo policy iteration
We now discuss some of the technical elements of turning this discussion into an actual learning algorithm, which we will refer to as **Monte-Carlo** policy iteration.
Producing actual experiences and exploring alternatives through the use of random actions is an example of a Monte-Carlo method, inspired by the randomness of games in casinos: these methods are essential to the computational study of many areas of science, from Physics and Chemistry to Computer Science and Mathematics.

As with dynamic programming, our algorithm must consist of two alternating steps: first, we must perform some evaluation for the current policy; second, we must improve our policy to a new one according to this evaluation.
In dynamic programming, we ended the evaluation step when the values of states converged.
However, many trajectories may provide no information to update the values, and unless we track which states have been updated, adding a significant memory overhead, there is no way to tell which states have even been updated.
This makes it difficult to define a measure with which we can say enough evaluation has occured: instead, the solution to this problem in Monte-Carlo is simply to decide on some pre-defined number of trajectories/episodes between each policy update.
We will label this number of trajectories $N_T$.

After each evaluation step, the policy is simply made greedy according to Eq. \eqref{monte_carlo_greedy_policy}, analogous to improving the policy according to Eq. \eqref{greedy_policy} in our dynamic programming algorithm.
We will call each period of evaluation, followed by a policy update, an epoch.
Unlike the dynamic programming algorithm, we have no idea whether the learning has converged to an optimal policy, since its possible some sequence of exploratory steps revealing a better action has been missed, regardless of how many epochs we consider: in practice, we thus also limit the number of epochs to some value $N_E$.

In practice, the greedy policy after evaluation is implemented by storing two value functions: a $Q$, which we take greedy actions with respect to, and a $Q'$, which we update between policy updates.
After an evaluation, $Q$ is replaced by the evaluated values of $Q'$.

Finally, as with the dynamic programming algorithm, for each step of policy evaluation we begin our state-action values $Q'(s,a)$ with the result at the end of the evaluation of the previous policy: if the new policy didn't change the future of any states, these values would be the same, so this is a good place to start.
Any new values calculated which disagree with the current value are then overwritten, while values which are not updated are left the same.

The resulting algorithm is sketched out below.
\begin{enumerate}
	\item Initialize the current value function $Q$, updated value function $Q'=Q$, number of epochs $N_E$ with epoch index $n=0$, and trajectories per epoch $N_T$ with trajectory index $i=0$.
	\item For the current epoch:
	\begin{enumerate}
		\item Generate a trajectory using $\epsilon$-greedy actions generated according to Eq. \eqref{epsilon-greedy}, with the greedy policy being that given by $Q$.
		\item For each state-action pair $(s_t,a_t)$ occurring during or after the last exploratory step of that episode, calculate the return
		$$\begin{align}
		R_t=\sum_{t'=t+1}^Tr_{t'},
		\end{align}$$
		and set $Q'(s_t,a_t)=R_T$.
		\item Iterate the trajectory index by $1$, i.e. $i\rightarrow i+1$.
		If $i=N_T$, end the evaluation and set $Q=Q'$, else return to a).
	\end{enumerate}
	\item Iterate the epoch index by $1$, i.e. $n\rightarrow n+1$.
	If $n=N_E$ end the algorithm.
	Else return to step 2.
\end{enumerate}

In real world applications, where the agent is trying to achieve its goals while learning, rather than in preparation for some future occasion with no importance given to current results, exploration can be detrimental.
If the agent has achieved a somewhat decent policy, the presence of exploratory steps could lower the returns in the short term, without appreciable gains in the long term.
In practice, finding the balance of **exploration vs exploitation**, as this issue is called, is more of an art than a science, and depends on the problem, the algorithm and the time frame of the task, along with many other aspects.

## On/off-policy methods and replay buffers
The form of exploration discussed in the previous sections is a particular case of a general concept called **off-policy** learning, where the actions taken differ from the policy we are trying to learn, with the aim of aiding exploration in order to achieve better final results.
For example, here we take actions with probabilities given by the $\epsilon$-greedy policy of Eq. \eqref{epsilon-greedy}, rather than simply following the greedy policy of Eq. \eqref{monte_carlo_greedy_policy}: the consequence is that we can not use trajectories to update states occurring before exploratory steps, as their futures are off-policy.
In contrast, **on-policy** learning updates the same policy that is used to produce trajectories, and therefore must learn a probabilistic policy directly in order to incorporate exploration.
A more detailed discussion of on- and off-policy learning is beyond the scope of these lectures, as it requires a more general formulation of control problems, called **Markov Decision Processes**, which we will not have time to cover in depth: instead we will retain a deterministic policy, and learn using off-policy exploration through the special case of $\epsilon$-greedy policies.

While we previously discarded all state-action pairs occurring before the last exploratory step, as their futures were not those corresponding to the _current_ policy, we can retain a memory of them for later use in our algorithm.
For example, in the lower trajectory of Fig. \ref{monte_carlo} while the future of the first exploratory step was not generated by the policy the agent was following at the time the trajectory was generated, it is the future that would have been generated by the _new_ policy after updating with information about the second exploratory step.
We could therefore use the future of the first exploratory step to update the value function of the new policy.
This is a special case of a general idea commonly used in industry and research, where stored past experiences are referred to as a **replay buffer**: in general, this is an off-policy method, since a generic set of past experiences will usually not be equivalent to experiences generated by the current policy.
Additionally we note that replay buffers are more commonly used with the techniques of the next section, since in a Monte-Carlo setting the full future of the actions, states and rewards after each step in the buffer is required, in order to check how the future actions taken in each state compare to those of the most recent policy.
The storage and check thus require a substantial memory and computing overhead.