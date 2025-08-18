![[Pasted image 20250818132922.png]]![[Pasted image 20250818132939.png]]
![[Pasted image 20250818132946.png]]
To determine the best action for Messi, we must calculate the expected utility for both "pass" and "shoot" using the value function U from iteration 2, with a discount factor γ = 1.0. The values from the table are: U(Messi) = -2, U(Suarez) = -0.6, and U(Scored) = 2.

The expected utility of an action is calculated as:  
Q(state, action) = Σ \[Probability(next_state) * (Reward + γ * U(next_state))]

**Action: Pass**
- The ball goes to Suarez with a probability of 1.0.
- The immediate reward for a pass is assumed to be 0, as it's not specified.
- Expected Utility (Pass) = 1.0 * (0 + 1.0 * U(Suarez))
- Expected Utility (Pass) = 1.0 * (0 + (-0.6)) = **-0.6**

**Action: Shoot**
- There's a 0.2 probability of scoring (next state: Scored) with a reward of 3.
- There's a 0.8 probability of the ball going to Suarez (next state: Suarez) with a reward of -2.
- Expected Utility (Shoot) =\[0.2 * (3 + 1.0 * U(Scored))] + \[0.8 * (-2 + 1.0 * U(Suarez))]
- Expected Utility (Shoot) = \[0.2 * (3 + 2)] +\[0.8 * (-2 + (-0.6))]
- Expected Utility (Shoot) =\[0.2 * 5] + \[0.8 * -2.6]
- Expected Utility (Shoot) = 1 - 2.08 = **-1.08**

Comparing the two outcomes, Q(Messi, Pass) = -0.6 is greater than Q(Messi, Shoot) = -1.08. Therefore, to maximize the reward, **Messi should choose to pass the ball.**

---

![[Pasted image 20250818132955.png]]
We will now calculate the utility values for iteration 3 (U₃) using the values from iteration 2 (U₂) and a discount factor γ = 1.0.
**U₃(Messi)**  
The utility of a state is the maximum of the expected utilities of all possible actions from that state. We calculated these in the previous question.
- U₃(Messi) = max(Q(Messi, Pass), Q(Messi, Shoot))
- U₃(Messi) = max(-0.6, -1.08) = **-0.6**

**U₃(Suarez)**  
We need to calculate the expected utility for Suarez's possible actions (Pass or Shoot)
- **Action: Pass** (to Messi, probability 1.0, reward 0)
    - Q(Suarez, Pass) = 1.0 * (0 + 1.0 * U₂(Messi)) = 1.0 * (0 + (-2)) = -2
- **Action: Shoot** (0.6 to Scored, reward 3; 0.4 to Messi, reward -1)
    - Q(Suarez, Shoot) = \[0.6 * (3 + 1.0 * U₂(Scored))] + \[0.4 * (-1 + 1.0 * U₂(Messi))]
    - Q(Suarez, Shoot) =\[0.6 * (3 + 2)] + \[0.4 * (-1 + (-2))]
    - Q(Suarez, Shoot) = \[0.6 * 5] + \[0.4 * -3] = 3 - 1.2 = 1.8
- **Conclusion:** U₃(Suarez) = max(-2, 1.8) = **1.8**
- 
**U₃(Scored)**  
There is only one action: return the ball to Messi (probability 1.0, reward -1).
- U₃(Scored) = 1.0 * (-1 + 1.0 * U₂(Messi))
- U₃(Scored) = 1.0 * (-1 + (-2)) = **-3**

| Iteration | 0 | 1 | 2 | **3** |  
| :--- | :--- | :--- | :--- | :--- |  
| **U(Messi)** | 0.0 | -1 | -2 | **-0.6** |  
| **U(Suarez)**| 0.0 | -2 | -0.6 | **1.8** |  
| **U(Scored)** | 0.0 | 3 | 2 | **-3** |

---


![[Pasted image 20250818133002.png]]
We will perform two iterations of policy evaluation for the given initial policy, with a discount factor γ = 0.8.  
The initial policy is: π(Messi) = Pass, π(Suarez) = Pass, π(Scored) = Return.  
The initial value function is: U₀(Messi) = 0, U₀(Suarez) = 0, U₀(Scored) = 0.
The update rule for policy evaluation is:  
Uₖ₊₁(s) = Σ \[P(s'|s, π(s)) * (R(s, π(s), s') + γ * Uₖ(s'))]
**Iteration 1 (calculating U₁ from U₀):**
- **U₁(Messi):**
    - The action is "Pass", the next state is Suarez with probability 1.0. The immediate reward for a pass is 0.
    - U₁(Messi) = 1.0 * \[0 + 0.8 * U₀(Suarez)] = 1.0 * \[0 + 0.8 * 0] = 0
- **U₁(Suarez):**
    - The action is "Pass", the next state is Messi with probability 1.0. The immediate reward for a pass is 0.
    - U₁(Suarez) = 1.0 * \[0 + 0.8 * U₀(Messi)] = 1.0 * \[0 + 0.8 * 0] = 0
- **U₁(Scored):**
    - The action is "Return", the next state is Messi with probability 1.0. The reward for reaching Messi is -1.
    - U₁(Scored) = 1.0 * \[-1 + 0.8 * U₀(Messi)] = 1.0 *\[-1 + 0] = -1
**Values after Iteration 1: U₁(Messi) = 0, U₁(Suarez) = 0, U₁(Scored) = -1.**

**Iteration 2 (calculating U₂ from U₁):**
- **U₂(Messi):**
    - The action is "Pass", next state is Suarez.
    - U₂(Messi) = 1.0 * \[0 + 0.8 * U₁(Suarez)] = 1.0 *\[0 + 0.8 * 0] = 0
- **U₂(Suarez):**
    - The action is "Pass", next state is Messi.
    - U₂(Suarez) = 1.0 * [0 + 0.8 * U₁(Messi)] = 1.0 * [0 + 0.8 * 0] = 0
- **U₂(Scored):**
    - The action is "Return", next state is Messi.
    - U₂(Scored) = 1.0 * [-1 + 0.8 * U₁(Messi)] = 1.0 * [-1 + 0] = -1

**Values after Iteration 2: U₂(Messi) = 0, U₂(Suarez) = 0, U₂(Scored) = -1.**  
The value function has converged after two iterations.

| Iteration | π(Messi) | π(Suarez) | π(Scored) |
| :-------- | :------- | :-------- | :-------- |
| 0         | Pass     | Pass      | Return    |
| 1         | Pass     | Pass      | Return    |
| 2         | Pass     | Pass      | Return    |

![[Pasted image 20250818133012.png]]
The core update rules differ: Q-learning is an off-policy algorithm. Its update rule is: Q(s, a) ← Q(s, a) + α * \[r + γ * maxₐ'(Q(s', a')) - Q(s, a)]. When updating the Q-value for the current state-action pair (s, a), it considers the action that yields the maximum Q-value among all possible actions a' in the next state s', i.e., maxₐ'(Q(s', a')). It learns the value of the optimal policy (greedy policy), regardless of what exploratory action the agent actually takes next. Sarsa is an on-policy algorithm. Its name comes from the data sequence (s, a, r, s', a') used in the update. Its update rule is: Q(s, a) ← Q(s, a) + α * \[r + γ * Q(s', a') - Q(s, a)]. Sarsa uses the Q-value Q(s', a') corresponding to the action a' actually executed in the next state s' for the update. It evaluates and improves the policy the agent is currently following (including exploration). Policy learning approach: Q-learning is more "aggressive" or "idealistic," directly learning the optimal path while ignoring potential risks encountered during exploration. Sarsa is relatively "conservative" because it incorporates the consequences of exploratory steps (e.g., a random, non-optimal action) into its value estimation. If an exploratory action could lead to poor outcomes, Sarsa learns that the state values near that region are lower, thus favoring safer paths.


![[Pasted image 20250818133017.png]]
1. **Find the maximum Q-value for the next state, Messi**:
    - Q(Messi, Pass) = -0.4
    - Q(Messi, Shoot) = -0.8
    - maxₐ'(Q(Messi, a')) = max(-0.4, -0.8) = -0.4
2. **Apply the update rule**:
    - Q(Suarez, Pass) ← -0.7 + 0.4 * [0 + 0.9 * (-0.4) - (-0.7)]
    - Q(Suarez, Pass) ← -0.7 + 0.4 * [-0.36 + 0.7]
    - Q(Suarez, Pass) ← -0.7 + 0.4 * [0.34]
    - Q(Suarez, Pass) ← -0.7 + 0.136
    - **New Q(Suarez, Pass) = -0.564**

![[Pasted image 20250818133021.png]]

The SARSA update formula is:  
Q(s, a) ← Q(s, a) + α * \[r + γ * Q(s', a') - Q(s, a)]
1. **Find the Q-value for the next state-action pair**:
    - We need Q(s', a'), which is Q(Messi, Shoot). From the Q-table provided in the problem, this value is **-0.8**.
2. **Apply the update rule**:
    - Q(Suarez, Pass) ← -0.7 + 0.4 * \[0 + 0.9 * (-0.8) - (-0.7)]
    - Q(Suarez, Pass) ← -0.7 + 0.4 * \[-0.72 + 0.7]
    - Q(Suarez, Pass) ← -0.7 + 0.4 * \[-0.02]
    - Q(Suarez, Pass) ← -0.7 - 0.008
    - **New Q(Suarez, Pass) = -0.708**

**Comparison to the Q-learning Update**
- **SARSA Update Result**: -0.708
- **Q-learning Update Result (from Question 5)**: -0.564

The fundamental difference lies in the **future value estimate** they use to update the Q-value. This stems from one being an on-policy algorithm and the other being an off-policy algorithm.
1. **Q-learning (Off-policy)**: When updating, it always assumes the optimal action will be taken in the next state. It therefore uses the value maxₐ' Q(Messi, a'), which is max(-0.4, -0.8) = -0.4, for its future value. It learns the value of the optimal policy, regardless of what action the agent actually takes next.
2. **SARSA (On-policy)**: When updating, it uses the Q-value corresponding to the action a' that the agent **actually takes** in the next state. In this problem, that action a' is "Shoot", so it uses Q(Messi, Shoot) = -0.8 for its future value. It learns the value of the policy that is currently being followed (including exploratory moves).
In this specific example, because the actual next action taken (Shoot, with a value of -0.8) was worse than the theoretically optimal action (Pass, with a value of -0.4), the SARSA update is more "pessimistic," causing the value of Q(Suarez, Pass) to decrease. In contrast, Q-learning ignores the suboptimal action that was actually executed and provides a more "optimistic" update, causing the value of Q(Suarez, Pass) to increase.

![[Pasted image 20250818133027.png]]
First, we deconstruct the game trace into a sequence of (state, action, reward). We will assume the discount factor γ = 0.9 and the learning rate α = 0.4, consistent with the previous questions.
1. **t=0**: State s₀=Suarez, Action a₀=Pass.
2. **t=1**: Receive reward r₁=0, arrive in state s₁=Messi. Messi takes action a₁=Shoot.
3. **t=2**: Receive reward r₂=3 (for scoring the goal), arrive in state s₂=Scored. Take action a₂=Return.
4. **t=3**: Receive reward r₃=-1 (for returning the ball), arrive in state s₃=Messi. Take action a₃=Pass.
We need to perform a 3-step SARSA update for the first state-action pair in this sequence, (s₀, a₀), which is (Suarez, Pass).
The formula for the n-step return G is:  
G_{t:t+n} = R_{t+1} + γ*R_{t+2} + ... + γⁿ⁻¹*R_{t+n} + γⁿ*Q(S_{t+n}, A_{t+n})
For our case, t=0 and n=3:  
G_{0:3} = r₁ + γ*r₂ + γ²*r₃ + γ³*Q(s₃, a₃)
5. **Calculate the value of G_{0:3}**:
    - r₁ = 0
    - r₂ = 3
    - r₃ = -1
    - Q(s₃, a₃) is Q(Messi, Pass), which from the Q-table is **-0.4**.
    - G = 1.6084
6. **Update Q(Suarez, Pass)**:
    - Q(s₀, a₀) ← Q(s₀, a₀) + α *\[G - Q(s₀, a₀)]
    - Q_new(Suarez, Pass) ← -0.7 + 0.4 * \[1.6084 - (-0.7)]
    - Q_new(Suarez, Pass) ← -0.7 + 0.4 * \[2.3084]
    - Q_new(Suarez, Pass) ← -0.7 + 0.92336
    - **New Q(Suarez, Pass) = 0.22336**

    In this scenario, the **3-step update is likely more accurate**. A 1-step update relies heavily on the estimated value of the next state (e.g., Q(Messi, Shoot) which was -0.8). This estimate can be highly inaccurate, especially early in the learning process. The 3-step update, however, incorporates more **real rewards** from the future (critically, the +3 for the goal) and reduces the dependency on inaccurate Q-value estimates. Because it integrates more actual feedback from the environment, its update to the value of Q(Suarez, Pass) (a significant jump from a negative to a positive value) is likely closer to the true long-term value of that action.
    The choice of the number of steps (n) is a classic **bias-variance tradeoff**.
    - **Short steps (e.g., n=1)**: High bias (because the update target is heavily based on a potentially wrong estimate), but low variance (the update is only affected by a few random events, making it relatively stable).
    - **Long steps (large n)**: Low bias (because the update target is based on many real rewards, making it a better sample of the true return), but high variance (the return depends on a long sequence of random actions and outcomes, so its value can fluctuate wildly between episodes, leading to unstable learning)    
    Therefore, more steps is not always better. An intermediate value for n often performs best by finding a good balance between reducing bias and controlling variance, leading to the most effective learning.