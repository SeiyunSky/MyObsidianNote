## I. Overview

This task compares and analyzes `myTeam.py` with the accompanying PDDL domain file `myTeam.pddl`. The overall architecture is a **hierarchical controller**: the high-level uses PDDL for symbolic planning (deciding "what to do"), and the low-level uses Q-learning to execute specific actions. When reading the source code, I found that the author gradually enhanced perception, priority-based goal selection, low-level strategy mapping, and performance optimization. Below, I outline the key changes in each iteration and present key code snippets from the final version for paragraph-by-paragraph analysis.







## II. Brief Introduction to Each Version

- **Version 1 (Baseline)**: Based on the Berkeley CS188 template, using PDDL as the high-level planner and basic Q-learning features/weights for low-level decision-making. The structure is clear but with limited perception and conservative weights.
    
- **Version 2 (Robustness Improvements)**: Added robustness changes: fallback strategies, returning an empty queue when a plan fails during detection to ensure no crashes when PDDL fails.  
  The final performance of this version achieved a 50% win rate and 20% draw rate against StaffTeam locally, but performed poorly on most maps. After submission, the actual score was very low, and it often lost to the StaffTeam baseline.  
  After inspection and parameter tuning, I found that on one hand, there was no historical trajectory, causing the agent to often fall into local optima. Compared to potentially unreachable opponents and inedible beans, the agent tended to pace back and forth to reduce penalties.  
  On the other hand, the agent could not properly handle the relationship between power pellets and enemy positions. When the score for eating ghosts was high, the agent would chase deep into the map and forget to carry beans back; when the score was low, it would not eat ghosts at all.  
  Based on these issues, I implemented the third version.
    
- **Version 3 (Final/Enhanced)**: The high-level added perceptions such as `ghost_is_close`/`capsule_is_close`, rewrote `getGoals` as a priority decision tree, mapped different rewards/features/weights for the low-level based on high-level actions, added an efficient BFS `efficient_dist_calculator`, and made more aggressive adjustments to features and weights.  
  This version is the one submitted to the repository. It achieved a 70% win rate and 5% draw rate locally. However, on alleyMap and jumboMap, due to the relatively narrow maps and the lack of role differentiation for the agent (attempts were made but with very poor results), the win rate on these maps was only around 20%.  
  Finally, I added some improvements for narrow maps, successfully achieving a 90% local win rate, with the win rate on specific maps increasing from 20% to 50%, completing the task.

![[Pasted image 20250821162303.png]]
![[Pasted image 20250821162311.png]]
![[Pasted image 20250821162323.png]]
![[Pasted image 20250821162329.png]]
![[Pasted image 20250821162335.png]]
![[Pasted image 20250821162341.png]]
![[Pasted image 20250821162349.png]]
![[Pasted image 20250821162357.png]]
![[Pasted image 20250821162403.png]]
![[Pasted image 20250821162408.png]]
![[Pasted image 20250821162415.png]]


## III. Core Code Analysis

#### Weights and Learning Parameters
```python
    QLWeights = {
            "offensiveWeights":{
                                        'closest-food': -100,
                                        'bias': 1, 
                                        'successorScore': 15,
                                        'chance-return-food': 1000,
                                        '#-of-ghosts-1-step-away': -50,
                                        'distance-to-ghost': -2,
                                        'distance-to-capsule': 15,
                                        },
            "defensiveWeights": {
                                        'numInvaders': -100, 
                                        'onDefense': 10, 
                                        'invaderDistance': -10,
                                        'stop': -10, 
                                        'reverse': -2
                                        },
            "escapeWeights":    {
                                        'onDefense': 100, 
                                        'enemyDistance': 3, 
                                        'stop': -10, 
                                        'distanceToHome': -50
                                        }
        }
    QLWeightsFile = BASE_FOLDER+'/QLWeightsMyTeam.txt'


```
- These weights directly determine Q-value calculation and low-level strategy selection. Setting `'closest-food'` to `-100` and `'chance-return-food'` to `1000` indicates that the author made the agent extremely eager for food and strongly inclined to bring carried food home as soon as possible (the weight for `chance-return-food` is very large).
- At the same time, the penalty for ghosts was reduced (`#-of-ghosts-1-step-away` changed from a typically more negative value to `-50`), an adjustment to make the agent more "bold".
- It is more aggressive in pursuing rewards and has a higher tolerance for risks. In competitions, if opponents are good at interception, it is vulnerable to counterattacks, but can score quickly when opponents are weak or food is scattered on the map.

---
#### #### Initializing PDDL Solver + Training Parameters
```python
    def registerInitialState(self, gameState: GameState):
        self.pddl_solver = pddl_solver(BASE_FOLDER+'/myTeam.pddl')
        self.highLevelPlan: List[Tuple[Action,pddl_state]] = None
        ...
        self.position_history = [] 
        self.trainning = True 
        self.epsilon = 0.1 
        self.alpha = 0.005 
        self.discountRate = 0.8

```
- Here, the PDDL parser is bound to `myTeam.pddl` and learning parameters are initialized. `position_history` is used to detect back-and-forth movement and heavily penalize it. The learning rate was reduced to ensure stable learning, and a discount rate of 0.8 was chosen to reduce the impact of historical learning on the agent. Since we ultimately need to compare with other agents, which are stronger than StaffTeam, over-learning from comparisons with StaffTeam would lead to performance degradation.

---

#### Key Segment of `chooseAction`
```python
        objects, initState = self.get_pddl_state(gameState)
        positiveGoal, negtiveGoal = self.getGoals(objects,initState)

        if not self.stateSatisfyCurrentPlan(initState, positiveGoal, negtiveGoal):
            self.highLevelPlan: List[Tuple[Action,pddl_state]] = self.getHighLevelPlan(objects, initState,positiveGoal, negtiveGoal)
            ...
        if not self.highLevelPlan or len(self.highLevelPlan) == 0:
            highLevelAction = "defend_base"
        else:
            highLevelAction = self.highLevelPlan[self.currentActionIndex][0].name
            
        MixedAgent.CURRENT_ACTION[self.index] = highLevelAction

        if not self.posSatisfyLowLevelPlan(gameState):
            self.lowLevelPlan = self.getLowLevelPlanQL(gameState, highLevelAction)
            self.lowLevelActionIndex = 0

        if not self.lowLevelPlan or self.lowLevelActionIndex >= len(self.lowLevelPlan):
             legalActions = gameState.getLegalActions(self.index)
             return random.choice(legalActions) if legalActions else Directions.STOP

        lowLevelAction = self.lowLevelPlan[self.lowLevelActionIndex][0]
        self.lowLevelActionIndex+=1
        return lowLevelAction
```
- First, construct the PDDL initial state and select goals via `getGoals`; re-plan if the current high-level plan is unsatisfactory.
- If PDDL generates an empty plan, directly set `highLevelAction` to `"defend_base"` to ensure robustness. Initially, to solve the crash issue with no plan, I directly returned an empty plan list. While this fixed the error, it significantly reduced the agent's effectiveness. In the final version, I changed returning an empty list to returning a safe fallback, as a conservative strategy better facilitates bean collection and base protection.
---
#### PDDL Initial State Enhancement in `get_pddl_state`

```python
        # NEW: Check for nearby threats (non-scared ghosts)
        dist_to_ghost = self.get_dist_to_closest_threat(gameState)
        if dist_to_ghost is not None and dist_to_ghost <= CLOSE_DISTANCE:
            states.append(("ghost_is_close", myObj))

        # NEW: Check for nearby opportunities (capsules)
        dist_to_capsule = self.get_dist_to_closest_capsule(gameState)
        if dist_to_capsule is not None and dist_to_capsule <= CLOSE_DISTANCE:
            states.append(("capsule_is_close", myObj))

```
- This is key to enhancing high-level perception: injecting "dangers/opportunities" directly as initial predicates in PDDL, allowing PDDL to consider these urgent conditions when generating plans.
- Intuitive effect: When there are nearby ghosts and a nearby capsule, priority strategies are triggered. Elevating low-level perception to symbolic planning input enables more reliable "tactical" switching.

---
#### Design of Goal Selector `getGoals`

```python
    def getGoals(self, objects: List[Tuple], initState: List[Tuple]) -> Tuple[List[Tuple], List[Tuple]]:
        init_set = set(initState)
        my_agent_obj = "a{}".format(self.index)

        if (("ghost_is_close", my_agent_obj) in init_set) and (("capsule_is_close", my_agent_obj) in init_set):
            print(f"Agent {self.index} Goal -> COUNTER ATTACK")
            positiveGoal = [("counter_attacked",)]
            negtiveGoal = []
            return positiveGoal, negtiveGoal

        if (("ghost_is_close", my_agent_obj) in init_set):
            print(f"Agent {self.index} Goal -> URGENT ESCAPE")
            positiveGoal = [("returned_home_safely",)]
            negtiveGoal = []
            return positiveGoal, negtiveGoal

        if (("5_food_in_backpack", my_agent_obj) in init_set):
            print(f"Agent {self.index} Goal -> STRATEGIC RETREAT")
            positiveGoal = [("returned_home_safely",)]
            negtiveGoal = []
            return positiveGoal, negtiveGoal

        if (("winning_gt10",) in init_set):
            print(f"Agent {self.index} Goal -> SECURE VICTORY (DEFEND)")
            positiveGoal = [("defended_base",)]
            negtiveGoal = []
            for obj, obj_type in objects:
                if obj_type in ["enemy1", "enemy2"]:
                    negtiveGoal.append(("is_pacman", obj))
            return positiveGoal, negtiveGoal

        print(f"Agent {self.index} Goal -> SAFE ATTACK")
        positiveGoal = [("cleared_food_safely",)]
        negtiveGoal = []
        return positiveGoal, negtiveGoal

```
- The `getGoals` method sets goals for the agent through hierarchical conditional checks, with priorities from highest to lowest: counterattack (ghost and capsule nearby simultaneously), urgent escape (only ghost nearby), strategic retreat (collected 5 foods), secure victory (defend when leading by over 10 points), and finally default to safe attack.
---

#### Low-Level Q-learning Mapping `getLowLevelPlanQL`

 ```python
         if highLevelAction in ["safe_attack", "eat_capsule_to_counter"]:
            rewardFunction = self.getOffensiveReward
            featureFunction = self.getOffensiveFeatures
            weights = self.getOffensiveWeights()
            learningRate = self.alpha
        elif highLevelAction in ["strategic_retreat", "urgent_escape"]:
            rewardFunction = self.getEscapeReward
            featureFunction = self.getEscapeFeatures
            weights = self.getEscapeWeights()
            learningRate = self.alpha
        elif highLevelAction in ["defend_base", "secure_victory_patrol"]:
            rewardFunction = self.getDefensiveReward
            featureFunction = self.getDefensiveFeatures
            weights = self.getDefensiveWeights()
            learningRate = self.alpha
        else: # Fallback
            print(f"Warning: High-level action '{highLevelAction}' not mapped to a Q-learning strategy. Defaulting to offensive.", file=sys.stderr)
            rewardFunction = self.getOffensiveReward
            featureFunction = self.getOffensiveFeatures
            weights = self.getOffensiveWeights()
            learningRate = self.alpha

 ```

- This is the hub of hierarchical control: the high-level action name determines which set of reward/feature/weights the low-level uses.

---
#### `efficient_dist_calculator`
```python
    def efficient_dist_calculator(self, pos, targets, walls):
        if not targets:
            return None
        
        fringe = deque([(pos, 0)])
        expanded = {pos}
        target_set = set(targets)
        
        while fringe:
            current_pos, dist = fringe.popleft()
            
            if current_pos in target_set:
                return dist

            nbrs = Actions.getLegalNeighbors(current_pos, walls)
            for nbr in nbrs:
                if nbr not in expanded:
                    expanded.add(nbr)
                    fringe.append((nbr, dist + 1))
        return None

```

- A common issue in the original implementation was using `list.pop(0)`, which leads to O(n²) behavior. Here, `deque`'s `popleft()` is used, reducing complexity back to O(n). Additionally, converting `targets` to a set improves target location efficiency. Compared to the initial baseline, it has better performance. The first version often took 2 minutes for a single test match, but after improving BFS, a local calculation can be completed in 10 seconds.

---

#### offensive Feature Expansion
```python
        food_list = self.getFood(nextState).asList()
        if food_list:
            dist = self.efficient_dist_calculator(next_pos, food_list, walls)
            if dist is not None:
                features["closest-food"] = float(dist) / (walls.width * walls.height)
        
        dist_to_ghost = self.get_dist_to_closest_threat(nextState)
        if dist_to_ghost is not None:
            if dist_to_ghost < 5:
                features['distance-to-ghost'] = float(dist_to_ghost) / (walls.width * walls.height)

        dist_to_capsule = self.get_dist_to_closest_capsule(nextState)
        if dist_to_capsule is not None:
             features['distance-to-capsule'] = 1.0 / (dist_to_capsule + 1)

```

- `closest-food` is normalized to the map area, resulting in a smaller value. Normalization based on map size addresses the earlier issue of poor performance on some maps.
- `distance-to-capsule` uses `1/(d+1)`, which gives high values when the capsule is very close, an effective way to encourage power-up collection.

---
#### Defense/Escape Reward
```python
    def getDefensiveReward(self, gameState: GameState, nextState: GameState):
        reward = -1
        ...
        if len(nextInvaders) < len(invaders):
            reward += 200

        if len(invaders) > 0 and len(nextInvaders) > 0:
            if next_closest_invader_dist < closest_invader_dist:
                reward += 15
            else:
                reward -= 10

        if gameState.getAgentState(self.index).configuration.direction == Directions.STOP:
            reward -= 3

        if len(self.position_history) > 1 and nextMyPos == self.position_history[-2]:
            reward -= 6

        return reward

```
- Large rewards are given for eating invaders (+200), and small rewards for approaching invaders (+15), encouraging proactive defensive behavior.
- Using `position_history` here to penalize "back-and-forth movement" enhances behavioral smoothness. Overall, it is reasonable.
---

#### PDDL Domain File
```pddl
actions: attack, defence, go_home, patrol, consolidate_lead
```
-  The PDDL introduces action groups: attack, defense, go home, patrol, and consolidate lead. For consolidating the lead, tests were conducted with leads between 3 and 20 points, and the 5-10 point range was found to be optimal.
- In subsequent tests, I discovered an issue with the consolidate lead setting: before the agent scores enough points, a trigger is needed to prioritize scoring tasks. Thus, rather than controlling the agent based on the current score, it is better to combine the number of beans collected but not yet returned home with the score.
- However, after adding this optimization to `getGoal`, I found the effect was not good, only yielding significant benefits on medium maps, while being completely counterproductive on narrow-passage maps. This is likely because the design is too rigid and unsuitable for all 11 maps.







## IV. Personal Insights

- During the task, I experienced both the intricacies of engineering implementation and was drawn to the idea of hierarchical design. The high-level using PDDL for symbolic planning and the low-level using Q-learning for specific actions – this hybrid "symbol + learning" architecture reminded me of hierarchical control theory discussed in class, effectively combining theory and engineering.
-  I was deeply impressed by the small optimization in `efficient_dist_calculator`: some seemingly minor adjustments to data structures (replacing `list.pop(0)` with `deque`) can significantly affect operational efficiency. This makes me more aware of choosing appropriate data structures in future programming assignments.
- I have gained a deeper understanding of the importance of "feature engineering" and "reward engineering". The interaction between different feature normalization methods and reward designs intuitively illustrates the idea in machine learning systems that "design is more important than training" in practical engineering applications.
---
![[Pasted image 20250821162424.png]]
## Summary
- This code reading and optimize assignment has taught me a lot: the design ideas of hierarchical architecture, the process from perception to symbolization to planning, how different features/rewards affect learning behavior, and the performance differences brought about by small optimizations.
- In the process of writing this report, I repeatedly compared the original code and tried to understand the intention behind each modification. By pasting the original text paragraph by paragraph and adding annotations, I feel that I have a clearer grasp of the overall logic and local implementation details of this implementation.
- This learning experience served as a huge work:studying the papers provided in the assignment, reading code, understanding motivations, optimize code,recording findings, and compiling them into a document. This process will be very helpful for future assignments and debugging of complex projects.