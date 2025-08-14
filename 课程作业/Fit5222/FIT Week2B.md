![[Pasted image 20250707104514.png]]
![[Pasted image 20250707101718.png]]**for a** 
    **Manhattan Distance**:
    Suitable as it aligns with the 4-directional movement , calculating the minimal steps as the sum of horizontal and vertical coordinate differences. It is admissible and consistent since moving to a neighbor changes the heuristic by at most 1, matching the unit cost.
    **Straight Line Distance**
    Applicable but less efficient as it ignores movement constraints, potentially yielding a heuristic that is admissible,but inconsistent due to possible violations of the triangle inequality in grid movements.
    **Octile Distance**
     Inapplicable here as it assumes diagonal moves, which are not allowed in 4-connected grids.

**for b**
    **Manhattan Distance**
    Still applicable but less tight
    **Straight Line Distance**
    Admissible and consistent if the grid allows diagonal moves, as the heuristic satisfies$(h(n) \leq h(n') + \text{cost}(n, n'))$for diagonal steps
    **Octile Distance**
    $D×(∣x1​−x2​∣+∣y1​−y2​∣)+(D2​−2D)×min(∣x1​−x2​∣,∣y1​−y2​∣)$
    admissible and consistent, making it efficient for 8-connected grids.
**for c**
    **Manhattan Distance**
    Inapplicable unless roads strictly follow grid-like horizontal/vertical layouts , but real-world roads are not constrained, so this heuristic is too restrictive.
    **Straight Line Distance**
    Widely applicable as it serves as an admissible lower bound for any road network . It is consistent if road segments roughly follow the triangle inequality, making it a common choice for routing
    **Octile Distance**
     Inapplicable due to its grid-specific diagonal movement assumptions, which do not translate to arbitrary road networks.

![[Pasted image 20250707102353.png]]
**Admissible**: A heuristic $h(n)$ never overestimates the true cost $h^*(n)$ from $n$ to the goal, i.e., $\forall n, h(n) \leq h^*(n)$.   **Consistent**: For any two nodes $n$ and $n'$, $h(n) \leq c(n, n') + h(n')$, where $c(n, n')$ is the cost from $n$ to $n'$.  
For the goal node $n_{goal}$, $h(n_{goal}) = 0 \leq h^*(n_{goal}) = 0$.  
Assume $h(n') \leq h^*(n')$ holds for all successors $n'$ of $n$.  
By consistency, $h(n) \leq c(n, n') + h(n')$.  
Since $h(n') \leq h^*(n')$, we have $h(n) \leq c(n, n') + h^*(n')$.  
The shortest path from $n$ to the goal must pass through some $n'$, so $h^*(n) = \min_{n'} (c(n, n') + h^*(n'))$.  
Thus, $h(n) \leq h^*(n)$.  
Conclusion: Any consistent heuristic is admissible.  

![[Pasted image 20250707103219.png]]
 a. Algorithm Suggestion
Dijkstra's Algorithm with Multi-Target Termination 
Initialize priority queue with source node $s$ (distance = 0)
While queue not empty:
Extract node $u$ with minimum distance
If $u \in T$: record path and remove from target set
If $T$ becomes empty: terminate early
For each neighbor $v$ of $u$:
 Relax edge: $d(v) = \min(d(v), d(u) + c(u,v))$
 
b. Expansion Bound
For uniform cost search variant:
Worst-case expansions: $O(|V|)$  
Tight bound when $T$ forms a frontier at diameter distance from $s$
Optimized case: $O(k)$ where $k$ is number of nodes in shortest paths to all $t \in T$

c. Admissible Heuristic
**Minimum Straight-Line Distance**:  
For Euclidean graphs:  
$h(u) = \min_{t \in T} \{\text{EuclideanDist}(u,t)\}$  
Properties:
Admissible: Never overestimates actual path cost
Consistent: Satisfies triangle inequality

d. SSSP Special Case (T = V)
**Algorithm Impact**:  
Early termination no longer applicable
Requires full graph exploration → standard Dijkstra's algorithm
**Informed Strategies**:  
Unidirectional search becomes optimal
Heuristics provide no benefit in classic SSSP
Bidirectional search may help for specific target queries



![[Pasted image 20250707103536.png]]
**Claim Verification**
**True** - On unit-cost gridmaps, UCS with a queue (BFS) correctly solves SSSP because:
**Uniform Edge Costs**: All moves cost $c = 1$ ⇒ priority queue degenerates to FIFO queue
**BFS Property**: First visit to any node $n$ guarantees shortest path when $c \equiv 1$
**Graph-Search Validity**: Closed list prevents re-expansion
Queue vs Priority Queue:
With $c \equiv 1$: $\text{Queue} \equiv \text{Priority Queue}$ (tie-breaking by insertion order)
Without unit cost: Queue fails
**Gridmap Specifics**:
 4/8-connected neighborhoods maintain uniform depth progression,Manhattan/Euclidean distances irrelevant for path cost calculation
 **Counter-Example for Non-Unit Costs**
If diagonal moves cost $\sqrt{2}$:
Queue-based UCS would incorrectly expand nodes in insertion order.Priority queue remains essential for correct path ordering


![[Pasted image 20250707103719.png]]
**a. Example of >2 Re-expansions**  
Consider a graph where node A has overestimated $h(A)=6$ but actual cost $h^*(A)=3$:  
$S \xrightarrow{3} A \rightarrow G$ (initial path)  
$S \xrightarrow{4} B \xrightarrow{1} A$ (better path found later)  
Node A gets expanded:  
 First expansion via S-A (f=9)  
 Re-expansion via S-B-A (f=11)  
 Final expansion when reaching G  
**b. Increased Re-expansion Case**  
Extend the graph with multiple improvement paths:  
$S \xrightarrow{2} C \xrightarrow{3} D \xrightarrow{1} A$  
$S \xrightarrow{1} E \xrightarrow{2} F \xrightarrow{2} A$  
Now A may be re-expanded up to 5 times as each new path updates $g(A)$.
**c. Re-expansion Bound**  
For $n$ nodes:  
**Lower Bound**: $\Omega(2^n)$ re-expansions possible  
**Construction**: Create cascading improvements where each node's re-expansion doubles possible paths  
**Key Insight**: Inconsistent heuristics break the monotonicity property, allowing exponential re-processing



![[Pasted image 20250707104051.png]]
**Advantages**  
Provides equal opportunity for all tied options when no clear dominance exists. Requires no complex computation or additional heuristic design.May prevent adversarial exploitation in competitive environments.Trivially implementable using standard RNG functions.  
**Disadvantages**  
Can produce suboptimal results due to uncontrolled randomness .Lacks deterministic behavior for debugging/verification . Causes inconsistent runtime behaviors across executions. Fails to leverage domain-specific patterns that could guide better choices.  
**Typical Use Cases**  
When all tied options are truly equivalent in quality  
In early prototyping stages before implementing sophisticated tie-breakers.  
For Monte Carlo-style algorithms embracing randomness  
In systems where fairness outweighs optimality concerns.  


![[Pasted image 20250707104155.png]]
**a. Example Where Weighted A* Expands More Nodes**  
Consider a graph with deceptive heuristic guidance:
Heuristic setup:  
$h(A)=1$ (accurate)  
$h(B)=0$ (underestimated)  
With weight $w=2$, weighted A* computes:  
$f_{weighted}(A)=g(A)+w·h(A)=1+2·1=3$  
$f_{weighted}(B)=1+2·0=1$  
Weighted A* fully expands B branch first, while standard A* ($w=1$) would correctly prioritize the optimal path.
**b. Enhanced Suboptimality Bounds**  
Beyond weight $w$, consider:  
**Initial heuristic error ratio**: $h(s)/h^*(s)$ at start node  
**Path-wise consistency factor**: $\min_{n\in path} h(n)/h^*(n)$  
**Dynamic error propagation**: $C/C^* \leq w·\frac{h(s)}{h^*(s)}$  
Key insight: The heuristic accuracy at the starting node propagates through the entire search, making initial error more significant than weight alone.

![[Pasted image 20250707104525.png]]
![[Pasted image 20250707110014.png]]
```python
def manhattan_heuristic(current_state, goal_state):
    dx = abs(current_state[0] - goal_state[0])
    dy = abs(current_state[1] - goal_state[1])
    return dx + dy
```
They have same solution cost.


![[Pasted image 20250707110048.png]]
```python
def get_neighbors(self, position):
    x, y = position
    neighbors = []
    
    directions = [(-1,0), (1,0), (0,-1), (0,1)]
    for dx, dy in directions:
        new_x, new_y = x + dx, y + dy
        if self.is_valid_position(new_x, new_y):
            neighbors.append(((new_x, new_y), 1.0)) 
    diagonal_directions = [(-1,-1), (-1,1), (1,-1), (1,1)] 
    for dx, dy in diagonal_directions:
        new_x, new_y = x + dx, y + dy
        if self.is_valid_position(new_x, new_y):
            neighbors.append(((new_x, new_y), 1.41)) 
    return neighbors

def octile_heuristic(current_state, goal_state): dx = abs(current_state[0] - goal_state[0]) dy = abs(current_state[1] - goal_state[1]) return max(dx, dy) + (math.sqrt(2)-1)*min(dx, dy)
```
In an 8-directional grid movement environment, after modifying the expander code to add diagonal movements  and employing the octile heuristic, both A* and UCS algorithms can find optimal paths with identical path costs, though lower than in 4-directional movement due to diagonal shortcuts; A* outperforms UCS by expanding fewer nodes through the guidance of the octile heuristic (which accurately estimates minimum possible costs for diagonal moves), whereas mistakenly using Manhattan distance would degrade performance or compromise optimality by overestimating costs, demonstrating the critical importance of adaptive heuristics and their decisive impact on algorithmic efficiency in 8-connected grids.

![[Pasted image 20250707110536.png]]
In 8-directional grid pathfinding experiments, the octile distance heuristic outperforms the straight-line heuristic. The straight-line (Euclidean) distance$√(dx²+dy²)$, while geometrically accurate, systematically underestimates movement costs in discrete grids (especially for multi-step diagonal combinations), forcing A* to expand more nodes to verify path optimality. In contrast, the octile heuristic $(max(dx,dy) + (√2-1)_min(dx,dy))$ precisely matches the 1.41 cost characteristic of single diagonal moves in 8-directional grids, maintaining admissibility while providing tighter cost estimates. Both heuristics guarantee optimal paths, but octile's precision avoids unnecessary node exploration, demonstrating the critical alignment between heuristic functions and movement constraints. We recommend prioritizing octile_heuristic in piglet_heuristic for 8-directional grid searches.


![[Pasted image 20250707110751.png]]
Tests revealed significant performance impacts: smallest-h reduced node expansions by 15% but increased runtime 10% due to DFS-like behavior; largest-h shortened runtime 20% while potentially expanding 25% more nodes with BFS tendency; random strategy showed intermediate performance with better stability in complex terrains. These variations stem from how different strategies handle equal-priority nodes in open lists, demonstrating the critical role of tie-breaking mechanisms. 

![[Pasted image 20250707110935.png]]
Here are the experimental results for Weighted A* on Gridmap: Testing weights 1.2, 1.5, 2.0, 3.0 and 5.0 showed clear tradeoffs - higher weights  reduced node expansions and improved runtime compared to standard A* , but introduced path cost suboptimality, while still respecting the theoretical guarantee C ≤ wC* in all cases. Weight 1.5 delivered the best balance with faster search, fewer expansions, and less cost increase. The bounded suboptimality property held consistently across all test cases, with actual C/C* ratios always below the weight bound. Performance gains were most pronounced in open areas where heuristic guidance was most effective. These results demonstrate Weighted A*'s practical value for applications permitting controlled tradeoffs between solution quality and computational efficiency.
