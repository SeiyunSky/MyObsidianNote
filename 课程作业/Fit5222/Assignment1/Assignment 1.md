# Summary of Solution for Question 1
## Problem Overview
The objective of Question 1 is to implement a single-agent pathfinding algorithm within the Flatland railway environment. The task is to find a valid and optimal path from a given starting position and direction to a target destination, without considering collisions with other agents.
## Algorithm: A* (A-Star) Search
To solve this problem, the **A* (A-Star) search algorithm** was implemented. A* is a well-known and highly efficient pathfinding algorithm that guarantees finding the shortest path if one exists.
### Why A*?
- **Optimal:** A* guarantees finding the path with the minimum cost (in this case, the shortest path) if the heuristic function used is admissible (i.e., it never overestimates the actual cost).
- **Complete:** It will always find a solution if one exists.
- **Efficient:** By using a heuristic to guide its search, A* explores a much smaller portion of the map compared to uninformed search algorithms like Breadth-First Search, making it significantly faster for most cases.
### Core Concepts of the Implementation
The A* algorithm works by selecting the path that minimizes the following function:  
f(n) = g(n) + h(n)
Where:
- g(n) is the actual cost from the start node to the current node n.
- h(n) is the estimated (heuristic) cost from node n to the goal.

The key components of our implementation are:
#### 1. State Representation
In Flatland, simply using (row, col) coordinates is insufficient to define a state. The possible transitions from a cell depend on the direction from which an agent enters it. Therefore, a state is uniquely defined by a tuple: **(row, col, direction)**.
#### 2. Heuristic Function (h(n))
We used the **Manhattan Distance** as the heuristic function. It calculates the sum of the absolute differences of the x and y coordinates between the current position and the goal. It's a perfect choice for a grid world where movement is restricted to four directions, as it's computationally cheap and never overestimates the true distance.
```python
def heuristic(pos, goal):
    return abs(pos[0] - goal[0]) + abs(pos[1] - goal[1])
```
#### 3. Data Structures
- **Open Set:** A **priority queue (min-heap)** was used to store the nodes to be visited. It efficiently retrieves the node with the lowest f_score, which is crucial for the algorithm's performance.
- **came_from Dictionary:** This dictionary stores the predecessor of each visited state, allowing for the reconstruction of the final path once the goal is reached.
- **g_score Dictionary:** This dictionary stores the g_score (the cost of the cheapest path from the start) for each visited state to avoid redundant computations.
## Code Structure
The solution is primarily contained within the get_path function and two helper functions.
- get_path(...): The main function that initializes the data structures and executes the A* search loop. It explores neighboring cells, calculates their scores, and adds them to the priority queue until the goal is found.
- heuristic(...): A helper function to compute the Manhattan distance.
- reconstruct_path(...): A helper function that traces back from the goal node using the came_from map to build the final list of coordinates representing the path.
## Conclusion
The implemented A* algorithm proved to be highly effective, successfully finding the optimal (shortest) path for all provided test cases with a 100% success rate. The results confirmed that the state representation, heuristic function, and overall algorithm logic were correctly implemented.

# Summary of Solution for Question 2

## Problem Overview
Question 2 elevates the complexity from a simple pathfinding task to one involving **dynamic obstacle avoidance**. While still planning for a single agent at a time, the algorithm must now find a collision-free path that accounts for the pre-determined movements of other agents already present in the environment.
## Algorithm: Time-Aware A* Search (A* with Reservation Table)
The solution extends the A* algorithm from Question 1 into a **Time-Aware A* Search**. This approach integrates the dimension of **time** directly into the pathfinding logic, transforming the problem from a 2D spatial search into a 3D space-time search.
### Why Time-Aware A*?
- **Handles Dynamic Obstacles:** A standard A* algorithm operating only on (x, y) coordinates cannot detect time-dependent collisions. By adding time to the state, we can check for conflicts at each specific timestep.
- **Maintains Optimality:** The algorithm still aims to find the shortest path in terms of time (which is equivalent to path length in this unweighted grid). The heuristic function continues to guide the search towards the goal efficiently.
### Core Modifications from Question 1
The implementation introduces three critical modifications to the original A* algorithm:
#### 1. Time-Aware State Representation
The state definition is expanded to include time. A state is no longer just (row, col, direction) but is now uniquely defined by the tuple **(row, col, direction, time)**. This is the fundamental change that allows the algorithm to reason about "when" an agent is at a certain location.
#### 2. Collision Detection via a Reservation Table
To efficiently check for collisions, we pre-process the existing_paths into a **reservation table**. This table is implemented as a Python set for O(1) average-case time complexity for lookups.
- **How it works:** Before the search begins, we iterate through all existing paths and add every occupied space-time coordinate (row, col, time) to the reserved_locations set.
- **Collision Types Handled:**
    - **Vertex Conflict:** An agent cannot move to a cell (x, y) at time if another agent is already there. This is checked by (x, y, time) in reserved_locations.
    - **Head-on/Swapping Conflict:** To prevent two agents from swapping adjacent cells in the same timestep, we also reserve the transition itself.
- **Benefit:** This pre-processing step avoids repeatedly looping through existing_paths inside the main A* loop, drastically improving performance.
#### 3. Expanded Action Space: Adding the "Wait" Action
In a dynamic environment, sometimes the best action is to do nothing. We expanded the agent's possible actions to include **waiting** for one timestep at its current location.
- **Implementation:** During each iteration of the A* loop, in addition to exploring valid forward moves, we also evaluate a "wait" state where the position and direction remain the same, but time is incremented by 1.
- **Importance:** This action is crucial for resolving conflicts where an agent must let another pass before it can proceed.

## Code Structure
- get_path(...): The main function now accepts existing_paths. It first builds the reservation table and then executes the time-aware A* loop. Inside the loop, it checks for collisions for every potential move (including waiting) before adding a new state to the priority queue.
- reconstruct_path(...): This helper function remains largely the same but now traces back through states that include the time dimension.
- **Pruning:** A check if current_time >= max_timestep: is included to prune branches that exceed the episode's time limit, preventing infinite loops.
## Conclusion
By transforming the standard A* algorithm into a time-aware search and using an efficient reservation table for collision detection, the solution successfully navigates the agent through a dynamic environment. It reliably finds a collision-free path to the goal or correctly returns an empty list if no such path exists within the given constraints.

# Summary of Solution for Question 3

## Problem Overview

Question 3 presented a true Multi-Agent Pathfinding (MAPF) challenge, requiring the simultaneous, conflict-free planning for all agents in a dynamic environment. The complexity was further increased by the introduction of mid-execution malfunctions, necessitating a robust replan function.

## Chosen Algorithm: Prioritized Planning with Enhancements

To tackle this MAPF problem, we implemented a **Prioritized Planning** framework. This decoupled approach was chosen for its excellent balance between implementation simplicity and effectiveness. It works by establishing a priority order for agents and planning their paths sequentially, where each planned path becomes a dynamic obstacle for all subsequent, lower-priority agents.

The core planning for each agent was handled by the **Time-Aware A* Search** algorithm developed in Question 2, which operates in a 3D space-time domain (row, col, direction, time).

### The Critical Performance Bottleneck: Pathological Cases

During testing, a critical performance bottleneck was identified in specific, complex scenarios. The Plan Time for certain test cases skyrocketed from milliseconds to nearly 30 seconds.

- **Root Cause:** This issue arises when a low-priority agent becomes trapped by the paths of higher-priority agents. The Time-Aware A* algorithm, in its attempt to find a solution, would enter a "calculation trap." It would endlessly explore "wait" actions, causing its search space (the open_set priority queue) to explode exponentially with millions of near-identical states, leading to a performance collapse.
    

### The Solution: Futility Pruning

To solve this, a crucial optimization named **Futility Pruning** was introduced into the core plan_single_agent_path function.

- **Purpose:** The goal of futility pruning is to prevent the algorithm from wasting computational resources on branches of the search tree that are clearly hopeless.
    
- **Mechanism:**
    
    1. A visit_count dictionary was introduced to track the number of times a **physical state** (defined by row, col, direction) has been expanded from the priority queue.
        
    2. A FUTILITY_THRESHOLD (e.g., 150) was set as a "patience limit."
        
    3. Inside the A* loop, before exploring a state's neighbors, the algorithm checks its visit count. If the count for that physical state exceeds the threshold, the algorithm assumes it's stuck in a loop and immediately prunes this branch by using continue, effectively abandoning the unpromising path.
        

This simple yet powerful heuristic ensures that the planner "fails fast" in pathological cases instead of hanging, maintaining excellent overall performance.

## Core Functions Implementation

- **get_path (Initial Planning):**
    
    1. **Prioritization:** Agents are prioritized based on their Manhattan distance to their target, with shorter-distance agents getting higher priority.
        
    2. **Sequential Planning:** The algorithm iterates through the prioritized list. For each agent, it calls the core A* planner (plan_single_agent_path), using the paths of all previously planned agents as dynamic obstacles in a reservation table.
        
- **replan (Dynamic Replanning):**
    
    1. **Re-Prioritization:** When a replan is triggered, agents are re-prioritized. Highest priority is given to newly malfunctioning agents, followed by agents that failed their previous move.
        
    2. **Reservation Table Construction:** A new reservation table is built for the replan, containing:
        
        - The future paths of all non-failing, non-malfunctioning agents.
            
        - The "waiting" paths of malfunctioning agents during their downtime.
            
    3. **Replanning Execution:** The prioritized list of agents to be replanned is then processed sequentially, using the core A* planner to find new, conflict-free path segments from their current state.
        

## Conclusion

The final solution is a robust and efficient Prioritized Planning system. By implementing a standard MAPF algorithm and enhancing it with a critical **Futility Pruning** mechanism, we successfully balanced the need for finding valid paths with the practical necessity of avoiding performance bottlenecks in complex, constrained scenarios. The system demonstrates the ability to generate initial plans and dynamically adapt to unforeseen events in real-time.