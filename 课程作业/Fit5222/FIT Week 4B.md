![[Pasted image 20250721160728.png]]
![[Pasted image 20250721160737.png]]
There are two agents, Agent 1 (A1) and Agent 2 (A2), on a grid.
To avoid a collision, one must wait for the other. 
A1's shortest path to its goal takes 5 steps. A2's shortest path to its goal takes 2 steps.
Conflict: A1's path and A2's path intersect in a way that requires one agent to wait for the other to pass.
a)
    A makespan-optimal plan requires that the agent with the longest journey, A1, proceeds along its 5-step path without any delay, as this defines the minimum possible makespan. To achieve this, A2 must wait for A1 to clear the shared part of their paths. For instance, A2 might have to wait for 3 timesteps before it can start its 2-step journey. In this plan, A1 finishes in 5 steps (cost=5), and A2 also finishes in 5 steps (3 steps waiting + 2 steps moving), resulting in a total cost of 5. The makespan is therefore **5** (max(5, 5)), which is optimal. However, the Sum-of-Costs (SOC) is 5 + 5 = **10**. As we will see in the next example, this SOC is not optimal, meaning this plan is makespan-optimal but not SOC-optimal.

b)
    An SOC-optimal plan aims to minimize the total combined cost, which means minimizing any waiting or detours. The most efficient way to resolve the conflict is for the agent with the longer path (A1) to wait for the agent with the shorter path (A2). In this case, A2 proceeds immediately and finishes in its minimum time of 2 steps (cost=2). A1 must wait briefly, for example for 1 timestep, to let A2 pass before continuing on its 5-step path. A1's total cost would then be 1 (waiting) + 5 (moving) = 6 steps. The Sum-of-Costs (SOC) for this plan is 2 + 6 = **8**, which is lower than the SOC of 10 in the previous plan and is therefore optimal. However, the makespan is now **6** (max(2, 6)), which is worse than the makespan of 5 from the previous plan. This plan is therefore SOC-optimal but not makespan-optimal.

![[Pasted image 20250721161533.png]]
    The search tree for Cooperative A* (CA*) is exponentially larger than the collective search trees of Operator Decomposition (OD). This is because CA* uses a coupled approach, searching in the high-dimensional joint state space where a single search node represents the simultaneous positions of all N agents. This method leads to a combinatorial explosion in its branching factor, which is approximately b^N (where 'b' is the number of moves for one agent). As a result, the CA* search tree grows to a scale of $O((b^N)^k)$, quickly becoming computationally intractable for even a small number of agents.
    In stark contrast, Operator Decomposition (OD) uses a decoupled strategy that breaks the problem into N smaller, separate searches—one for each agent. Each search is performed in the low-dimensional space of a single agent, with a manageable branching factor of just 'b'. While OD performs N such searches, the total computational effort is roughly $O(N * b^k)$, which is dramatically smaller and more scalable than CA*. This significant reduction in search space is OD's primary advantage, though it comes at the cost of sacrificing the guarantees of optimality and completeness that CA* provides.

![[Pasted image 20250721161959.png]]
a)
There are **3 independent groups** of agents.  The large obstacle in the center of the map effectively separates the agents:
- **Group 1 (Left side):** {Pink, Light Blue}. These two agents start and end on the left side of the map (columns A-F). Their shortest paths inevitably conflict, so they must be planned together.
- **Group 2 (Top-right):** {Green}. This agent has a very short, direct path (H2 to J2) that does not interfere with any other agent's likely paths. It can be planned independently.
- **Group 3 (Right side):** {Orange, Grey, Yellow}. These three agents all operate on the right side of the map (columns H-L). Their start and goal locations are intertwined, guaranteeing multiple path conflicts that require coordinated planning. For example, Grey and Yellow must swap locations, and Orange's path from bottom to top cuts across theirs.

b)
To find the earliest arrival times, we must find a conflict-free plan for each group. The times below represent the cost (number of timesteps) of each agent's path in an optimal solution for that group.
- **Pink Agent (F2 → A4):** Arrival time is **8**.
    - Its shortest path is 7 steps. However, it conflicts with the Light Blue agent at cell E2 at the first timestep. By waiting one step to let the Light Blue agent pass, its path cost becomes 8.
- **Light Blue Agent (E1 → B6):** Arrival time is **8**.
    - Its shortest path is 8 steps. To resolve the conflict with the Pink agent, it proceeds without waiting, allowing the Pink agent to wait instead.
- **Green Agent (H2 → J2):** Arrival time is **2**.
    - As an independent agent with no conflicts, it can take its shortest path of 2 steps.
- **Orange Agent (H5 → L2):** Arrival time is **6**.
    - Its shortest path is 6 steps (e.g., via H5→I5→I3→J3→K3→L3→L2). This path successfully avoids the initial locations of the Grey and Yellow agents, allowing it to move first within its group.
- **Grey Agent (I4 → L6):** Arrival time is **11**.
- **Yellow Agent (L4 → I6):** Arrival time is **11**.
    - The Grey and Yellow agents have a complex interaction. After the Orange agent clears its path (by t=6), Grey and Yellow must perform a swap. This requires one agent to take a significant detour around the other, adding many steps to their base shortest paths (which are 5 steps each). A possible solution results in both arriving at timestep 11.

c)
This refers to the constraints that a Conflict-Based Search (CBS) algorithm would use to resolve conflicts. The most direct and unavoidable conflicts give rise to the following constraints:
1. **A vertex constraint between Pink and Light Blue.** Their shortest paths both require them to be at cell **E2 at timestep 1**. To resolve this, a constraint must be added for one agent. For the solution in (b), the constraint is **(Pink, E2, t=1)**, which forbids the Pink agent from being at E2 at t=1.
2. **An edge constraint between Grey and Yellow.** Their shortest paths require them to swap positions by moving along the same corridor in opposite directions. For example, between timestep 1 and 2, Grey wants to move J4 → K4 while Yellow wants to move K4 → J4. This is a classic edge conflict. The applicable constraint would be **(Grey, (J4, K4), t=2)** or **(Yellow, (K4, J4), t=2)** to prevent the swap and force a detour.
3. **Multiple vertex constraints within Group 3.** The paths of Orange, Grey, and Yellow conflict at multiple locations and times. For example, a shortest path for Orange might conflict with Yellow's starting location at L4, leading to a constraint like **(Orange, L4, t=3)**.

![[Pasted image 20250721162503.png]]
To guarantee that a MAPF instance is "well-formed," we need to design the environment or the task-assignment process in such a way that it is impossible or always suboptimal for one agent's optimal path to pass through another agent's start or goal location. Here are three strategies to achieve this:
1. **Dedicated Start and Goal Zones**  
    This strategy involves physically separating the areas where agents can start and end their tasks from the main transit areas. For example, in a warehouse grid, you could designate the entire top row for picking up items (start points) and the entire bottom row for dropping them off (goal points). The area in between is the transit zone. An agent's optimal path will always be a direct route through the transit zone. It would be highly inefficient and thus non-optimal for an agent to travel into the start/goal zone of another agent as part of its path.
2. **Dead-End Start/Goal Locations**  
    Another strategy is to place all start and goal locations on "spurs" or "dead-end" cells that branch off the main travel paths. Imagine a main corridor with one-way traffic. Each pickup (start) and drop-off (goal) station is located on a short, dead-end branch off this corridor. For an agent to travel along its optimal path, it would never need to enter and then immediately exit another station's dead-end branch, as this would be a pointless detour. Therefore, the start and goal locations of other agents will never appear on its optimal path.
3. **Graph-Based Vertex Roles and Rules**  
    A more formal approach is to model the environment as a directed graph and assign specific roles to its vertices. Vertices can be classified as 'Start', 'Goal', or 'Transit'. We can then enforce strict rules: 'Start' vertices can only have outgoing edges, 'Goal' vertices can only have incoming edges, and 'Transit' vertices can have both. This structure makes it impossible for a goal location to be an intermediate point on any path, because it is a "sink" with no way out. Similarly, a start location cannot be an intermediate point because it has no incoming edges. This provides a formal guarantee that the instance is well-formed.

![[Pasted image 20250721162621.png]]

a)
Conflict-Based Search (CBS) is a two-level algorithm designed for Multi-Agent Path Finding (MAPF). Its primary components, as inferred from the slides and general knowledge of the algorithm, are:
1. **High-Level Search:** This is the master planner that searches a **Constraint Tree (CT)**.Each node in this tree represents a subproblem defined by a set of constraints. The high-level search explores this tree, typically using a best-first search strategy (like A*), to find a node whose solution is free of conflicts.The cost of each node is usually the sum-of-costs of the individual paths in its solution.
2. **Low-Level Search:** This is a single-agent pathfinding engine, such as A* or its variants (like the FOCAL search mentioned in the slides).Its job is to find an optimal path for a single agent from its start to its goal, while strictly adhering to the set of constraints (e.g., (agent, location, time)) provided by the high-level search for that specific agent
3. **Conflict Detection and Resolution:** This is the core logic that bridges the two levels. After the low-level search generates paths for a given CT node, a validation step checks for collisions (conflicts) between the paths of different agents.If a conflict is found, the high-level search resolves it by "splitting" the current CT node. It creates two new child nodes, each inheriting the parent's constraints plus one new constraint that forces one of the conflicting agents to avoid the point of conflict.

b)
```Python
function CBS_Algorithm(agents, map):
    // 1. Initialize High-Level Search
    root_node = create_root_node()
    root_node.solution = find_all_agent_paths_with(LowLevelSearch, {}) 
    root_node.cost = calculate_sum_of_costs(root_node.solution)
    OPEN_LIST = priority_queue()
    OPEN_LIST.add(root_node)

    // 2. High-Level Search Loop
    while OPEN_LIST is not empty:
        current_node = OPEN_LIST.pop_lowest_cost()
        conflict = find_first_conflict(current_node.solution)
        if conflict is NULL:
            return current_node.solution 
        child_node_A = create_child_node_from(current_node)
        add_constraint(child_node_A, constraint(agent_A, location, time))
        child_node_A.solution[agent_A] = LowLevelSearch(agent_A, child_node_A.constraints)
        child_node_A.cost = calculate_sum_of_costs(child_node_A.solution)
        OPEN_LIST.add(child_node_A)
        child_node_B = create_child_node_from(current_node)
        add_constraint(child_node_B, constraint(agent_B, location, time))
        child_node_B.solution[agent_B] = LowLevelSearch(agent_B, child_node_B.constraints)
        child_node_B.cost = calculate_sum_of_costs(child_node_B.solution)
        OPEN_LIST.add(child_node_B)
    return "No solution found"
```


c)When a node's solution contains multiple collisions, the choice of which one to resolve can significantly impact the algorithm's performance. Instead of picking one at random, a smart heuristic should be used.
A highly effective heuristic is to **prioritize conflicts based on their type, particularly identifying and resolving "cardinal" conflicts first**.
- **Definition:** A conflict is considered **cardinal** if resolving it unavoidably increases the cost for both conflicting agents. This means neither agent has an alternative path of the same optimal length that avoids the conflict. A conflict is **semi-cardinal** if it forces a cost increase for only one of the two agents, and **non-cardinal** if both agents can find alternative optimal paths to avoid it.
- **Heuristic Strategy:** The strategy is to always choose a cardinal conflict to resolve if one exists. If there are no cardinal conflicts, choose a semi-cardinal one. If all conflicts are non-cardinal, one can be chosen arbitrarily or based on a secondary criterion (like the earliest time of occurrence).
- **Why it Works:** By splitting on a cardinal conflict, the costs of both resulting child nodes in the Constraint Tree are guaranteed to be higher than their parent's cost. This increases the lower-bound cost of solutions in that branch of the search tree, allowing the high-level A* search to more effectively prune suboptimal branches and converge on the truly optimal solution much faster.

![[Pasted image 20250721163803.png]]
![[Pasted image 20250721163815.png]]

![[Pasted image 20250721163823.png]]
Q1:
 python piglet.py -f graph -p ./example/example_grid_8_8.scen -s a-star -m -n X
Agent = 3
![[Pasted image 20250721165420.png]]

Agent = 4
![[Pasted image 20250721165514.png]]

Agent = 5
![[Pasted image 20250721165533.png]]

Agent =6
![[Pasted image 20250721165551.png]]

Agent = 7    TimeOut 
![[Pasted image 20250721165647.png]]

On an 8x8 map, centralized A* can find the optimal solution for up to 6 agents. When the number of agents exceeds 6, the number of nodes generated is too large due to the exponential growth of the branching factor at a rate of b^N, and the program cannot be completed due to long computation time or memory exhaustion.

Q2:
 python piglet.py -f graph -p ./example/example_grid_8_8.scen -s uniform -m -n 3
 ![[Pasted image 20250721170055.png]]
 python piglet.py -f graph -p ./example/example_grid_8_8.scen -s breadth -m -n 3
![[Pasted image 20250721170140.png]]
Comparing the performance of A*, Uniform Cost Search (UCS), and Breadth-First Search (BFS) for the 3-agent problem highlights the critical role of a good heuristic. A* successfully found the optimal solution by expanding just 9 nodes. In contrast, both UCS and BFS, which are uninformed (blind) search strategies, failed to solve the same problem within the 30-second time limit. Before timing out, UCS had already expanded 9,655 nodes and BFS had expanded 8,821 nodes, orders of magnitude more than A*. This stark difference proves that the pigelet_multi_agent_heuristic (sum of Manhattan distances) effectively guides the A* search toward the goal, drastically pruning the enormous search space and making the problem solvable, whereas UCS and BFS waste computation exploring irrelevant paths.


