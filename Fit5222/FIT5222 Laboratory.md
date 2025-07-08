![[Pasted image 20250623150400.png|400]]
# 1， 
![[Pasted image 20250623150418.png|500]]
- **BFS**
    python piglet.py -p ./example/example_8_puzzle.scen  -f tree -s breadth -t 30 -n 10 
    ![[Pasted image 20250623153208.png|1222]]
- DFS
    python piglet.py -p ./example/example_8_puzzle.scen -f tree -s depth -t 30 -n 10
    ![[Pasted image 20250623154012.png|1200]]
- UCS
    python piglet.py -p ./example/example_8_puzzle.scen -f graph -s uniform -t 30 -n 10 
    ![[Pasted image 20250623153933.png|1100]]
 **Answer**:
    The data reveals an inherent limitation: Both BFS/DFS fail catastrophically on 8-puzzle under tree framework, while uniform graph search succeeds universally. This proves tree-based searches are fundamentally ill-suited for permutation problems due to inability to reuse states—a theoretical constraint independent of hardware.DFS generates 3x more nodes  than BFS  yet equally fails, exposing blind search algorithms' core flaw: Without heuristic guidance, both strategies inevitably drown in combinatorial explosion—a limitation rooted in algorithmic theory rather than implementation.Uniform graph search achieves 100% success but with wildly varying path costs, demonstrating even working algorithms face theoretical ceilings: When multiple local optima exist, non-heuristic strategies cannot guarantee optimality, highlighting inherent gaps in classical search paradigms.

![[Pasted image 20250623150426.png|500]]
![[Pasted image 20250630211353.png]]
![[Pasted image 20250630214447.png]]

![[Pasted image 20250630214503.png]]

TotalCost is **29**
**I believe my current plan cannot be further optimized because collisions are inevitable in this simulator scenario. The most effective solution is to have one train wait until the other has completely cleared the shared track section. This waiting strategy appears to be the only viable approach given the layout constraints and collision mechanics shown in the simulation.**