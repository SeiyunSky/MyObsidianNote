![[Pasted image 20250728101149.png]]
![[Pasted image 20250728101214.png]]

**Step 0: Initialization**
- **Action**: The search begins. The root node #1 is added to the OPEN list.
- **f1_min**: -
- **Focal Bound (w * f1_min)**: -
- **OPEN**: { #1(f1:10, f2:4) }
- **FOCAL**: {}
---

**Step 1: Populate FOCAL**

- **Action**: The FOCAL list is empty. Update f1_min from OPEN and populate FOCAL.
- **f1_min**: 10 (from node #1, the best node in OPEN).
- **Focal Bound**: 1.5 * 10 = 15
- **OPEN**: {} (Node #1 is moved out).
- **FOCAL**: { #1(f1:10, f2:4) } (since 10 ≤ 15).

---

**Step 2: Expand from FOCAL**

- **Action**: Expand all nodes in FOCAL. Currently, only node #1 is present.
- **f1_min**: 10
- **Focal Bound**: 15
- **OPEN**: { #2(f1:14, f2:3), #3(f1:16, f2:4) } (Children of #1 are added to OPEN, sorted by f1).
- **FOCAL**: {} (Node #1 is removed after expansion).

---

**Step 3: Repopulate FOCAL**

- **Action**: The FOCAL list is empty. Update f1_min from OPEN and repopulate FOCAL.
- **f1_min**: 14 (from node #2, the new best node in OPEN).
- **Focal Bound**: 1.5 * 14 = 21
- **OPEN**: {} (Eligible nodes will be moved to FOCAL).
- **FOCAL**: { #2(f1:14, f2:3), #3(f1:16, f2:4) }
    - **Explanation**: Both node #2 (f1=14) and node #3 (f1=16) have f1 values less than or equal to the bound of 21. They are moved to FOCAL. Inside FOCAL, they are sorted by f2, placing node #2 (f2=3) before node #3 (f2=4).

---

**Step 4: Expand from FOCAL (Part 1)**

- **Action**: FOCAL is not empty. Expand the first node according to f2 order, which is node #2.
- **f1_min**: 14
- **Focal Bound**: 21
- **OPEN**: { #5(f1:18, f2:3), #4(f1:22, f2:0) } (Children of #2 are added to OPEN, which is then sorted by f1).
- **FOCAL**: { #3(f1:16, f2:4) } (Node #2 is removed after expansion).

---

**Step 5: Expand from FOCAL (Part 2)**

- **Action**: FOCAL is still not empty. Continue by expanding the next node, #3.
- **f1_min**: 14
- **Focal Bound**: 21
- **OPEN**: { #6(f1:17, f2:2), #5(f1:18, f2:3), #7(f1:18, f2:2), #4(f1:22, f2:0) } (Children of #3 are added to OPEN, and the list is re-sorted by f1).
- **FOCAL**: {} (Node #3 is removed after expansion).

---

**Step 6: Repopulate FOCAL for the Third Time**

- **Action**: The FOCAL list is empty again. Update f1_min from OPEN and repopulate.
- **f1_min**: 17 (from node #6, the new best node in OPEN).
- **Focal Bound**: 1.5 * 17 = 25.5
- **OPEN**: {}
- **FOCAL**: { #4(f1:22, f2:0), #6(f1:17, f2:2), #7(f1:18, f2:2), #5(f1:18, f2:3) }
    - **Explanation**: All nodes currently in OPEN (#4, #5, #6, #7) have f1 values less than or equal to the bound of 25.5. All are moved to FOCAL. The FOCAL list is then sorted by f2. Node #4, with f2=0, is now at the front of the queue.

---

**Step 7: Goal Found**

- **Action**: FOCAL is not empty. Expand the first node according to f2 order, which is node #4.
- **f1_min**: 17
- **Focal Bound**: 25.5
- **OPEN**: {}
- **FOCAL**: { #6(f1:17, f2:2), #7(f1:18, f2:2), #5(f1:18, f2:3) }
- **Result**: **Node #4 is a goal node. The search terminates.**
### Conclusion
The sequence of expanded nodes is: **#1, #2, #3, #4**.
The solution found is **node #4**, which has an f1 cost of 22.


![[Pasted image 20250728101222.png]]
**a）**、If f2 is equal to f1, the efficiency of focal search degrades to be identical to that of the standard A* algorithm.
The purpose of the FOCAL list is to select a node from a set of "good enough" candidates (defined by f1 <= w * f1_min) using a secondary heuristic, f2, that prioritizes "easier" or "more promising" paths. However, if f2 is the same as f1, the sorting criterion within the FOCAL list becomes identical to the sorting criterion of the OPEN list.
Consequently, the algorithm will always select the node with the minimum f1 value from the FOCAL list for expansion. This node is, by definition, the best node in the entire OPEN list as well. The secondary heuristic f2 has no effect, and the weight w becomes irrelevant. The node expansion order will be exactly the same as in a standard A* search, and the potential advantage of focal search—gaining speed by sacrificing guaranteed optimality—is lost.

**b）**、When f2 is the node's depth, focal search exhibits a **Breadth-First Search** tendency, and its efficiency becomes highly dependent on the problem's structure.
This strategy is designed to find a solution quickly, especially if the solution path has few steps (not necessarily a low cost). Within the cost-bound of the FOCAL list, the algorithm prioritizes exploring paths closer to the root node. If a shallow solution with an acceptable cost exists, this approach can find it very rapidly. It is useful for problems where finding any solution is more critical than finding an optimal one.
This strategy completely ignores any domain-specific heuristic information. A shallow node depth does not guarantee that the node is on a path to a low-cost goal. The search might be misguided into exploring many high-cost or dead-end shallow branches. As a result, the quality (f1 value) of the first solution found might be poor, likely near the upper bound of w * f1_min.


**c）**、The choice of f2 should aim to capture a secondary objective or guide the search towards structurally simpler solutions.
1. **MAPF (Multi-agent pathfinding)**:
    - **A good f2**: **The Number of Conflicts**.
    - **Reasoning**: In MAPF, the primary objective (f1) is to minimize the sum of path costs. A core challenge is resolving collisions. Among sets of paths with similar f1 values, the one with fewer conflicts is closer to a valid, collision-free solution. Prioritizing the expansion of nodes with fewer conflicts can lead to a valid solution much faster and effectively prune the search tree.
2. **Path finding on weighted terrains**:
    - **A good f2**: **The Number of steps on high-cost terrain**.
    - **Reasoning**: f1 represents the total path cost (e.g., the sum of terrain weights). For two paths with a similar total cost, one might briefly cross a very expensive tile, while the other takes a slightly longer route on cheap tiles. f2 can express a preference for "flatter" or "safer" paths by minimizing steps on high-cost terrain, guiding the search towards smoother or less risky routes.
3. **Pipe routing with bends costs**:
    - **A good f2**: **The Number of bends**.
    - **Reasoning**: In pipe routing, f1 is typically a combination of total pipe length and penalties for bends. However, in engineering practice, fewer bends often mean lower manufacturing costs, easier installation, and better flow dynamics. Therefore, among routing options with a similar total cost (f1), prioritizing the one with the fewest bends (f2) guides the search toward more practical and elegant solutions.


![[Pasted image 20250728101227.png]]
Increasing the bounded suboptimal weight w results in a fundamental trade-off: it generally decreases the quality of the solution found but can potentially increase the speed of the search. Specifically, a larger w relaxes the admission condition for the FOCAL list (f1 <= w * f1_min), which makes the FOCAL list larger and gives the secondary heuristic f2 more nodes to choose from. This makes the search behavior more greedy with respect to f2, which can lead to finding a solution faster if f2 is an effective guide, but can also slow the search down by misguiding it into exploring many unpromising nodes. However, the most certain effect is a degradation in solution quality, as the algorithm is permitted to expand nodes with higher f1 costs, meaning the first solution it finds will almost certainly be more suboptimal.

![[Pasted image 20250728101233.png]]
Compared to the conservative windowing approach from Q1, these more flexible methods for maintaining the FOCAL list offer significant advantages by making the search process more dynamic, efficient, and responsive. The conservative strategy is batch-oriented and must wait for FOCAL to be empty before updating, leading to wasted computation on known suboptimal nodes. The advantages of flexible strategies are primarily: first, they increase efficiency and reduce unnecessary node expansions by allowing the algorithm to immediately "re-focus" on newly discovered, more promising paths; second, they provide greater dynamism, moving away from a rigid batch-processing model to one that adapts to the evolving search frontier; and finally, they enable better utilization of heuristics because the secondary heuristic f2 is always applied to a more current and relevant set of candidate nodes. In summary, these strategies optimize the algorithm's decision-making by allowing more frequent synchronization between the FOCAL and OPEN lists, thereby improving the overall efficiency of finding a high-quality solution.



![[Pasted image 20250728101238.png]]
#### Method 1: Greedy/Independent A* Planning
This is the simplest and fastest approach. It completely ignores interactions between agents. For each agent, a standard single-agent pathfinding algorithm like A* is run independently to find its shortest path from its start to its goal location.
- **Advantages**:
    - **Extremely Fast**: All planning tasks are parallel, single-agent problems.
    - **Simple**: Very easy to implement.
#### Method 2: Prioritized Planning
This is a sequential approach. First, assign a unique priority to every agent, either randomly or based on a heuristic (e.g., path length). Then, plan a path for each agent one by one, in descending order of priority. When planning for a low-priority agent, the paths of all higher-priority agents are treated as dynamic obstacles that must be avoided.
- **Advantages**:
    - **Can Generate Feasible Solutions**: If successful, this method directly produces a collision-free, feasible solution, providing a very good starting point for LNS.
    - **Relatively Fast**: Much faster than complete MAPF solvers.

#### Method 3: Using a Complete Suboptimal Solver
Before starting LNS, use a complete, bounded-suboptimal MAPF solver, such as ECBS , to compute an initial solution.
- **Advantages**:
    - **High-Quality Start**: Immediately yields a collision-free, feasible solution with a quality guarantee (e.g., cost is within a factor w of optimal).
    - **Complete**: Guarantees finding a solution if one exists.
    - **Faster LNS Convergence**: Starting from a high-quality solution can help LNS converge to an even better solution more quickly.

![[Pasted image 20250728101242.png]]
After MAPF-LNS2 finds a feasible solution, we can continue to improve its solution quality by designing smarter "destroy" and "replan" strategies. For the destroy strategy instead of selecting agents randomly, we can target those most impactful to the solution quality, such as agents with the highest path cost, the longest path in terms of steps, or those who experience the most conflicts or waits along their paths. Another strategy is to select a group of agents that are geographically close or whose paths intersect, as they often have significant potential for combined optimization. For the replan strategy, we can move beyond simple greedy methods and employ more powerful local solvers. For instance, for the small group of "destroyed" agents, we could run a local, time-limited CBS or ECBS to find a locally optimal or near-optimal path configuration for them, treating all other agents' paths as static or dynamic obstacles. This "destroy-and-optimize" cycle can be repeated until a time limit is reached or the solution quality no longer improves significantly.

![[Pasted image 20250728101249.png]]
python piglet.py -p example/arena2.map.scen -s a-star --focal 2 -n 200 -o focal_results.csv
![[Pasted image 20250728104300.png]]
python piglet.py -p example/arena2.map.scen -s a-star --heuristic-weight 1.5 -n 200 -o wastar_results.csv
![[Pasted image 20250728104337.png]]
This experiment compares Focal Search, based on a conservative windowing strategy (with w=1.5 and f2=h(n)), against the standard Weighted A* algorithm (with w=1.5). The results reveal a profound trade-off. Weighted A* demonstrates absolute superiority in search efficiency, with its number of expanded nodes being, on average, an order of magnitude smaller than that of Focal Search. However, Focal Search, by virtue of its "forced exploration" mechanism, succeeded in finding lower-cost, higher-quality solutions for several complex problems.
This indicates that while the conservative Focal Search strategy sacrifices speed, its broader search scope can sometimes avoid the local optima traps caused by the purely greedy strategy of Weighted A*. Therefore, these two algorithms serve different objectives: Weighted A* is suitable for scenarios with extreme requirements for solving speed, whereas the conservative Focal Search demonstrates its unique value in scenarios where one is willing to invest greater computational cost in exchange for a higher-quality solution.

