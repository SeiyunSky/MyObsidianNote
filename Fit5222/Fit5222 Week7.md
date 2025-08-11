![[Pasted image 20250811151400.png]]
![[Pasted image 20250811151408.png]]
**a. 
- **Problem Formulation:**
    - **States:** A state can be defined as a tuple (x, y, orientation), where:
        - (x, y) represents the robot's current coordinates in the maze grid.
        - orientation represents the robot's current facing direction (North, East, South, West).
    - **Initial State:** (center_x, center_y, North).
    - **Goal State:** Any state (x_exit, y_exit, orientation) where (x_exit, y_exit) is a coordinate just outside the maze or at a designated exit point.
    - **Actions:**
        - Turn(direction): Changes the robot's orientation to direction (North, East, South, West).
        - MoveForward(distance): Moves the robot distance units in its current orientation until it hits a wall or completes the distance. The distance could be a single unit (e.g., one grid cell) or a variable distance until it hits an obstacle. For simplicity, we can assume it moves one grid cell at a time.
    - **Path Cost:** Each action could have a cost (e.g., 1 per move/turn, or distance for moves). The goal is to find a path with minimum cost.
- **State Space Size:**  
    Let W be the width of the maze and H be the height of the maze.
    - Number of possible (x, y) positions = W * H.
    - Number of possible orientations = 4 (North, East, South, West).
    - Therefore, the total state space size = (W * H) * 4.  
        If the maze is represented as a grid, and W and H are the dimensions of that grid, the state space can be quite large, especially for complex mazes.

**b. 
- **Problem Reformulation:**  
    This observation simplifies the problem by changing the nature of actions and states. We no longer need to consider every single cell as a potential turning point.
    - **States:** A state can now be defined as a tuple (intersection_x, intersection_y, orientation_at_intersection), where (intersection_x, intersection_y) are coordinates of an intersection or a dead end.
    - **Actions:**
        - MoveToNextIntersection(): From an intersection, the robot moves straight along a corridor until it reaches the next intersection, a dead end, or the maze boundary. This action implicitly includes the distance traveled.
        - Turn(direction): Once at an intersection, the robot can turn to align itself with an outgoing corridor.
    - The robot's orientation only matters when it arrives at an intersection and needs to decide which corridor to take next
- **State Space Size:**  
    Let N_intersections be the number of distinct intersections (including dead ends and start/goal points if they are effectively "intersections" where decisions are made) in the maze.
    - Number of possible (intersection_x, intersection_y) positions = N_intersections.
    - Number of possible orientations at these intersections = 4.
    - Total state space size = N_intersections * 4.  
        Since N_intersections is typically much smaller than W * H (the total number of grid cells), this reformulation significantly reduces the state space.

**c. 
- **Problem Reformulation:**  
    This is a further simplification, where "turning point" seems to refer to intersections or dead ends, similar to part b, but with an emphasis on the action being a direct move to such a point.
    - **States:** A state can simply be (x, y), representing a specific turning point (intersection or dead end) in the maze.
    - **Actions:**
        - MoveTo(next_turning_point): From the current turning point, the robot executes a single action that takes it directly to an adjacent turning point in one of the four cardinal directions (if a path exists without hitting a wall). The action encapsulates the entire movement and implicit turns needed to align with the next segment.
 we **do not need to explicitly keep track of the robot's orientation** in the state definition anymore. The actions MoveTo(next_turning_point) implicitly handle the necessary turns and movements. When we are at a turning point A and choose to MoveTo turning point B, the robot "knows" which way to go without its orientation being part of the state. The orientation is handled by the action itself, as it defines moving along a specific corridor from one turning point to another.

**d. 
Here are three simplifications made in the initial problem description:
1. **Perfect Movement and Collision Detection:** The robot is assumed to move perfectly straight and stop precisely at a wall without any complex physics (e.g., momentum, skidding, varying friction). It "will stop before hitting a wall" implies an ideal sensor and braking system, and the ability to instantly halt. In the real world, movement might be imprecise, and collision detection could be complex, involving impact dynamics.
2. **Discrete Environment/Grid Representation:** The maze is implicitly treated as a discrete grid or a set of defined corridors and intersections. Real-world mazes have continuous spaces, and the robot's position could be any (x, y) within those continuous coordinates, not just specific grid points. This simplifies pathfinding by reducing it to a graph problem rather than a continuous control problem.
3. **No Obstacles Other Than Walls:** The problem only mentions walls as obstacles. In a real-world scenario, there could be other dynamic or static obstacles (e.g., debris, other robots, people) that the robot would need to perceive and avoid. The problem simplifies by assuming an empty maze save for its structure.

![[Pasted image 20250811151424.png]]
![[Pasted image 20250811151429.png]]

a、
A planning graph consists of alternating layers of propositions and actions. For each layer, we include all propositions that are true and all actions whose preconditions are met. In a relaxed planning graph (which is typical for these problems), we ignore negative effects (deletions).
**Initial State (P0 - Proposition Layer 0):**  
P0 = {garbage, cleanHands, quiet, ¬dinner, present}
**A0 (Action Layer 0 - Applicable actions from P0 and their effects):**
- cook(): Precondition cleanHands is in P0. Effects: dinner.
- wrap(): Precondition quiet is in P0. Effects: present.
- carry(): Precondition none. Effects: garbage, cleanHands.
- dolly(): Precondition none. Effects: garbage, quiet.

**P1 (Proposition Layer 1 - All propositions from P0 + all effects of A0):**
- Propositions from P0: garbage, cleanHands, quiet, ¬dinner, present
- New effects from A0: dinner (from cook())  
    (Note: present, garbage, cleanHands, quiet are also effects, but they were already in P0).
P1 = {garbage, cleanHands, quiet, ¬dinner, present, dinner}

**A1 (Action Layer 1 - Applicable actions from P1 and their effects):**  
All actions applicable in A0 remain applicable as their preconditions (cleanHands, quiet, none) are still met in P1. No new actions become applicable because no new preconditions are met that weren't met in P0.
- cook(): Precondition cleanHands (in P1). Effects: dinner.
- wrap(): Precondition quiet (in P1). Effects: present.
- carry(): Precondition none. Effects: garbage, cleanHands.
- dolly(): Precondition none. Effects: garbage, quiet.

**P2 (Proposition Layer 2 - All propositions from P1 + all effects of A1):**  
Since no new propositions are generated by actions in A1 that were not already in P1, P2 is identical to P1.  
P2 = {garbage, cleanHands, quiet, ¬dinner, present, dinner}
The planning graph has reached a **steady-state** at P1, as P2 is identical to P1. No new propositions will be added in subsequent layers.
**Where there's a possible solution:**
The goal state is g = {dinner, present, ¬garbage}.  
Let's check if all goal literals are present in P1 (the steady-state proposition layer):
- dinner: Is present in P1.
- present: Is present in P1.
- ¬garbage: Is **NOT** present in P1. In fact, garbage is present, and no action in the given table has ¬garbage as an effect or implies its removal. The actions carry() and dolly() actually have garbage as a positive effect.


b、
- **Literal Mutex:** Two propositions are mutex if they are contradictory (e.g., P and ¬P).
- **Action Mutex:**
    - **Inconsistent Effects:** Two actions are mutex if one action makes a proposition true and the other makes its negation true (e.g., Action A effects P, Action B effects ¬P).
    - **Interference:** An action's effect negates a precondition of another action.
    - **Competing Needs:** Two actions require preconditions that are themselves mutex.

**What can we do so we don't have them?**
- **Correct Action Definitions:** If actions are generating unintended conflicts (e.g., an action producing P when another action must produce ¬P and both are needed, but they are consistently mutex), the actions might need to be redefined. In our problem, if carry() and dolly() were intended to clean up garbage, their effects should have been ¬garbage. Correcting this (effect: ¬garbage) would resolve the unachievable goal literal issue, but would introduce mutexes with garbage if other actions (or the initial state) make garbage true.
- **Alternative Actions/Paths:** Sometimes, mutexes indicate that a direct path is impossible, and an alternative sequence of actions or a different set of parallel actions must be chosen to avoid the conflict.
- **Problem Domain Redesign:** More broadly, if a problem consistently leads to unsolvable states due to pervasive mutexes, it might indicate a flaw in the domain's logical representation of actions and their effects.

c、
**How to make sure there is a solution (goal state reachability):**
1. **Check Goal Literal Presence in Graph:** The first necessary condition is that all literals in the goal state must appear in at least one proposition layer of the planning graph. If any goal literal is never generated (as ¬garbage was in our example), then no solution exists.
2. **Check Goal Set Non-Mutex:** Once all goal literals appear in a layer (say, P_k), we must check that these goal literals are not mutex with each other within P_k. If they are, it means they cannot simultaneously be true.
3. **Perform Backward Search (Solution Extraction):** The planning graph provides an optimistic estimate of reachability (ignoring mutexes during forward expansion). To confirm a solution, a backward search must be performed from the goal layer. This search attempts to find a set of non-mutex actions in the preceding action layer whose effects cover the goal literals, and recursively ensure their preconditions are met in earlier non-mutex layers. If such a non-conflicting path from the initial state to the goal can be extracted, then a solution exists. If the backward search fails to find such a path (due to unavoidable mutexes), then no solution exists.
    
**What can we add to make sure the graph reaches a steady-state?**
- . **Finite State/Action Space:** For any well-defined PDDL problem, there is a finite number of propositions and actions.
-  **Monotonicity (Additive Effects):** In a relaxed planning graph, propositions, once true in a layer, remain true in all subsequent layers. No propositions are ever removed. Actions, once applicable, generally remain applicable.
- **Fixed Point Detection:** Because of the finite and monotonic nature, the graph will eventually reach a point where no new propositions can be generated, and no new actions can become applicable. When the set of propositions in P_k is identical to the set of propositions in P_k-1, the graph has reached its fixed point or **steady-state**. The algorithm simply checks for this equality at each step to know when to stop expanding.


![[Pasted image 20250811151438.png]]
The delete relaxation heuristic, often denoted as hadd or hmax, operates by ignoring all negative effects deletions of actions in a planning problem, meaning that once a proposition becomes true, it remains true indefinitely. This heuristic is generally not accurate in providing an exact approximation of the minimum effort optimal cost to reach the goal. While it is admissible, meaning it never overestimates the true cost h(n) <= h*(n), it frequently underestimates it significantly. This inaccuracy stems from several factors: it completely ignores the need to resolve conflicts between propositions or actions that would be mutually exclusive in the real problem; it leads to an overly optimistic view of reachability, assuming preconditions remain met even if a real-world action would negate them; and it implicitly allows for maximum parallelism of non-mutex actions, whereas an actual plan might require more sequential steps due to dependencies ignored by the relaxation. In essence, the delete relaxation offers a computationally efficient lower bound on the true cost, but its disregard for crucial real-world constraints like negative effects and conflicts limits its accuracy in tightly approximating the optimal solution cost.


![[Pasted image 20250811151445.png]]
**d. Nothing ever becomes blocked**


![[Pasted image 20250811151504.png]]
![[Pasted image 20250811151510.png]]
![[Pasted image 20250811151517.png]]
**c. Drive to Hongkong and Capetown in parallel, then from SB to M.**


![[Pasted image 20250811151539.png]]
![[Pasted image 20250811151547.png]]
![[Pasted image 20250811160246.png]]
**Advantages of using Greedy Relaxed Planning Heuristic:**
1. **Significantly Reduced Node Expansions:** This is the most striking advantage observed.
    - For pb2.pddl: Uniform search expanded 5756 nodes, while A* expanded only 74 nodes, and Greedy Best-First expanded a mere 17 nodes.
    - For pb3.pddl: Uniform expanded 5791 nodes, A* expanded 1527 nodes, and Greedy Best-First expanded an astonishing 21 nodes.  
        This reduction is a direct result of the heuristic guiding the search more effectively towards the goal, avoiding exploration of irrelevant parts of the state space.
2. **Faster Runtime:** A direct consequence of fewer node expansions is faster execution.
    - For pb2.pddl: Uniform took 0.125s, A* took 0.0312s, and Greedy Best-First took 0.0156s.
    - For pb3.pddl: Uniform took 0.1562s, A* took 0.5s (interestingly slower here, possibly due to heuristic computation overhead for larger search space explored by A*), and Greedy Best-First took 0.0s (likely too fast to measure precisely, indicating extremely quick termination).  
        Greedy Best-First, especially, demonstrates extremely fast performance due to its aggressive pursuit of the seemingly best path.
3. **Guidance in Complex Problems:** The heuristic provides a strong directional bias to the search, which is crucial for larger or more complex problems where uninformed search (like Uniform Cost) would be computationally intractable. This is evident in the dramatic drop in expanded nodes.
**Disadvantages of using Greedy Relaxed Planning Heuristic:**
4. No Guarantee of Optimality for A (Potentially):* While A* aims for optimality when coupled with an admissible heuristic, the "greedy" nature of our pddl_greedy_relaxation_expander (which picks only one child) could, in theory, impact its admissibility depending on how costs are accumulated or if the "greedy choice" leads to an overestimation in some edge cases. However, in the provided results, for pb2.pddl and pb3.pddl, A* still found optimal solutions (Cost 15 and 17, matching Uniform). This suggests that for these specific problems and the heuristic's implementation, it remains admissible or sufficiently close to it to find the optimal path. This is a positive outcome for A in this specific test case, but a general theoretical drawback for potentially non-admissible greedy heuristics.*
5. **No Guarantee of Optimality for Greedy Best-First Search:** As expected, Greedy Best-First Search, by its nature, does not guarantee optimal solutions. It simply follows the path that looks best at each step. In this specific test, for pb2.pddl and pb3.pddl, it coincidentally found optimal solutions (Cost 15 and 17, matching Uniform). This implies the greedy heuristic was effective in guiding it to an optimal path for these particular problems. **However, this is not a general guarantee; for other problems, it could yield sub-optimal solutions.**
6. **Problem pb1.pddl Failure:** All three strategies (Uniform, A*, Greedy Best-First) failed to solve pb1.pddl and expanded only 16 nodes. This indicates that either pb1.pddl is inherently unsolvable with the given domain/initial state, or the search space for it is structured in a way that even with the heuristic, a solution could not be found within the search limits (though 16 nodes is very low, suggesting immediate failure). The greedy relaxed planning heuristic, despite its speed, did not help solve pb1.pddl where Uniform Cost Search also failed. This highlights that if a problem is truly difficult or unsolvable, the heuristic won't magically create a solution.
The greedy relaxed planning heuristic, as implemented, significantly improves the search efficiency (fewer nodes expanded, faster runtime) for pb2.pddl and pb3.pddl compared to uninformed Uniform Cost Search. While Greedy Best-First is the fastest, it typically comes with the trade-off of not guaranteeing optimality. Interestingly, in this specific test, both A* and Greedy Best-First (using this greedy heuristic) found optimal solutions for pb2.pddl and pb3.pddl, which is a strong positive result for these problems, suggesting the heuristic is quite good at guiding the search for them. The main theoretical disadvantage of not guaranteeing optimality (especially for Greedy Best-First) still holds, even if not demonstrated in these specific test cases. The heuristic did not offer a way to solve pb1.pddl, which seems to be unsolvable or particularly challenging for all tested strategies.

![[Pasted image 20250811151555.png]]

**I. Advantages of Optimal Relaxed Planning Heuristic:**
1. Guarantee of Optimality for A (Theoretically Stronger):* The optimal relaxed planning heuristic, being derived from an optimal solution of the relaxed problem (ignoring delete effects), is typically **admissible** (never overestimates the true cost). When used with A*, this theoretically guarantees finding an optimal solution, as observed for pb2.pddl (Cost 15, matching Uniform). This is its primary theoretical strength over non-admissible heuristics.
2. **Consistent Optimal Solution for Greedy Best-First:** Interestingly, for both pb2.pddl and pb3.pddl, Greedy Best-First Search with the Optimal Relaxed Heuristic still found optimal solutions (Cost 15 and 17), matching Uniform and A* with Greedy Heuristic. This suggests the optimal relaxed heuristic provides very strong guidance, even for a greedy search that doesn't guarantee optimality.
3. **Improved Heuristic Value Accuracy:** By exploring all possible relaxed actions to find the optimal relaxed plan, the h value produced by optimal_delete_relaxation_h is generally more accurate (a tighter lower bound) than greedy_delete_relaxation_h. This is important for A* to prune more effectively.

**II. Disadvantages of Optimal Relaxed Planning Heuristic:**
1. **Higher Heuristic Computation Cost and Total Runtime (Significant):** This is the most pronounced disadvantage observed in your data.
    - For pb2.pddl:
        - A* (Optimal Relaxed) had **859** nodes expanded (vs 74 for Greedy Relaxed A*) and took **16.8438s** (vs 0.0312s for Greedy Relaxed A*). This indicates that while the search space might be theoretically "smaller" due to better pruning, the computational overhead of calculating a more accurate h value for each expanded node is immense. The Runtime exploded.
        - Greedy Best-First (Optimal Relaxed) also saw an increase in runtime to **0.5469s** (vs 0.0156s for Greedy Relaxed G-BFS), despite similar node expansions (18 vs 17). This further points to the cost of heuristic calculation.
    - For pb3.pddl:
        - The problem became even more critical. A* (Optimal Relaxed) resulted in a **"Time out"** after 30 seconds, whereas A* (Greedy Relaxed) found a solution in 0.5s. This clearly shows that the increased complexity of calculating the optimal relaxed plan heuristic at each step becomes prohibitive for larger problem instances.
        - Greedy Best-First (Optimal Relaxed) also took **0.6094s** (vs 0.0s for Greedy Relaxed G-BFS), again highlighting the significant increase in heuristic computation time.
2. **No Benefit for Unsolvable Problems (pb1.pddl):** Like the greedy heuristic, the optimal relaxed heuristic offers no advantage for problems that are fundamentally unsolvable or out of reach. All strategies, including those with the optimal relaxed heuristic, failed on pb1.pddl

**III. Comparison to Greedy Relaxed Planning Heuristic (Q1):**
- **Trade-off between Accuracy and Speed:** The optimal relaxed heuristic provides a more accurate heuristic value, which is theoretically superior. However, this accuracy comes at a very high computational cost. The greedy heuristic, while less "accurate" in its H value calculation, is much faster to compute, and for these problems, it proved to be remarkably efficient while still finding optimal solutions.
- **Practicality:** For the tested "pacman" problems, the **greedy relaxed planning heuristic** (from Q1) actually offered a much better practical trade-off between speed and solution quality. It delivered optimal solutions (for pb2 and pb3) with significantly less runtime and fewer node expansions (for A*) or comparable node expansions (for G-BFS) than the optimal relaxed heuristic.
- **Scalability:** The "Time out" for pb3.pddl with A* (Optimal Relaxed) clearly demonstrates that while theoretically stronger, the optimal relaxed heuristic's computational demands can severely limit its scalability to larger or more complex problems, making the faster-to-compute greedy heuristic a more viable choice in practice, even if its theoretical guarantees are weaker.
The optimal relaxed planning heuristic is theoretically strong due to its admissibility and potentially more accurate h values, which can lead A* to optimal solutions. However, its significant computational cost for heuristic calculation, which involves solving a sub-problem (the relaxed plan itself) optimally at each node expansion, can render it impractically slow for even moderately sized problems, as demonstrated by the pb3.pddl timeout. In contrast, the simpler and faster-to-compute greedy relaxed heuristic (from Q1) proved to be more efficient for these specific problems, still yielding optimal solutions with much faster runtimes, making it a more pragmatic choice.