![[Pasted image 20250804150129.png]]**1、**

| Component             | Details                                                                       |
| --------------------- | ----------------------------------------------------------------------------- |
| **Predicates**        | FlatOnAxle, SpareInTrunk, SpareOnAxle, TrunkOpen, SpareAvailable, FlatRemoved |
| **Initial State (I)** | { FlatOnAxle, SpareInTrunk }                                                  |
| **Goal State (G)**    | { SpareOnAxle }                                                               |
| **Actions (O)**       | The set of possible operators, detailed below.                                |

**Action Definitions**

|   |   |   |   |
|---|---|---|---|
|Action|Preconditions|Add List|Delete List|
|OpenTrunk|(none)|TrunkOpen|(none)|
|GetSpareFromTrunk|SpareInTrunk, TrunkOpen|SpareAvailable|SpareInTrunk|
|RemoveFlatTire|FlatOnAxle|FlatRemoved|FlatOnAxle|
|PutSpareOnAxle|SpareAvailable, FlatRemoved|SpareOnAxle|SpareAvailable, FlatRemoved|



**2、**
- F：FlatOnAxle
- SIT：SpareInTrunk
- SO：SpareOnAxle
- TO：TrunkOpen
- SA：SpareAvailable
- FR：FlatRemoved
**States:**
- **S0: {F, SIT}** (Initial State)
    - From S0, action OpenTrunk is possible.
        - **Transition:** S0 --(OpenTrunk)--> S1
- **S1: {F, SIT, TO}**
    - From S1, actions GetSpareFromTrunk and RemoveFlatTire are possible.
        - **Transition:** S1 --(GetSpareFromTrunk)--> S2
        - **Transition:** S1 --(RemoveFlatTire)--> S3
- **S2: {F, TO, SA}** (The SIT predicate is now false)
    - From S2, action RemoveFlatTire is possible.
        - **Transition:** S2 --(RemoveFlatTire)--> S4
- **S3: {SIT, TO, FR}** (The F predicate is now false)
    - From S3, action GetSpareFromTrunk is possible.
        - **Transition:** S3 --(GetSpareFromTrunk)--> S4
- **S4: {TO, SA, FR}** (This state can be reached from S2 or S3)
    - From S4, action PutSpareOnAxle is possible.
        - **Transition:** S4 --(PutSpareOnAxle)--> S5
- **S5: {TO, SO}** (Goal State - SA and FR are now false)

3、**Pseudocode Solution**
```PYTHON
function Solve_Spare_Tire_Problem():
    // Initial State: Flat tire on axle, Spare tire in trunk.
    // Goal State: Spare tire on axle.

    // Step 1: Gain access to the spare tire.
    // Action: OpenTrunk
    // Preconditions: None
    // Effects: TrunkOpen becomes true.
    EXECUTE_ACTION("OpenTrunk")

    // Step 2: Retrieve the spare tire.
    // Action: GetSpareFromTrunk
    // Preconditions: SpareInTrunk (initially true), TrunkOpen (from Step 1)
    // Effects: SpareAvailable becomes true, SpareInTrunk becomes false.
    EXECUTE_ACTION("GetSpareFromTrunk")

    // Step 3: Remove the damaged tire.
    // Action: RemoveFlatTire
    // Preconditions: FlatOnAxle (initially true)
    // Effects: FlatRemoved becomes true, FlatOnAxle becomes false.
    EXECUTE_ACTION("RemoveFlatTire")

    // Step 4: Mount the good tire.
    // Action: PutSpareOnAxle
    // Preconditions: SpareAvailable (from Step 2), FlatRemoved (from Step 3)
    // Effects: SpareOnAxle becomes true, SpareAvailable becomes false, FlatRemoved becomes false.
    EXECUTE_ACTION("PutSpareOnAxle")

    // Problem solved, car has a spare tire on the axle.
    PRINT "Spare tire successfully changed. Car is ready to go."

// Helper (conceptual): Simulates execution and state update.
function EXECUTE_ACTION(action_name):
    PRINT "Performing action: " + action_name
    // In a real planner, this would update the current state
    // based on the action's add and delete lists.
```

![[Pasted image 20250804150528.png]]
![[Pasted image 20250804150539.png]]
![[Pasted image 20250804150548.png]]
**Sentences for action**
```pddl
(define (domain shakey)
  (:requirements :strips :typing)

  (:types
    robot box location room switch - object
  )

  (:predicates
    (At ?o - object ?l - location)     
    (InRoom ?l - location ?r - room)  
    (OnFloor ?r - robot)              
    (On ?r - robot ?b - box)         
    (Pushable ?b - box)             
    (Climbable ?b - box)             
    (TurnedOn ?s - switch)           
  )


  (:action Go
    :parameters (?r - robot ?from - location ?to - location ?rm - room)
    :precondition (and
      (OnFloor ?r)
      (At ?r ?from)
      (InRoom ?from ?rm)
      (InRoom ?to ?rm)
    )
    :effect (and
      (At ?r ?to)
      (not (At ?r ?from))
    )
  )


  (:action Push
    :parameters (?r - robot ?b - box ?from - location ?to - location ?rm - room)
    :precondition (and
      (Pushable ?b)
      (OnFloor ?r)
      (At ?r ?from)
      (At ?b ?from)
      (InRoom ?from ?rm)
      (InRoom ?to ?rm)
    )
    :effect (and
      (At ?r ?to)
      (At ?b ?to)
      (not (At ?r ?from))
      (not (At ?b ?from))
    )
  )


  (:action ClimbUp
    :parameters (?r - robot ?b - box ?l - location)
    :precondition (and
      (Climbable ?b)
      (OnFloor ?r)
      (At ?r ?l)
      (At ?b ?l)
    )
    :effect (and
      (On ?r ?b)
      (not (OnFloor ?r))
      (not (At ?r ?l))
    )
  )


  (:action ClimbDown
    :parameters (?r - robot ?b - box ?l - location)
    :precondition (and
      (On ?r ?b)
      (At ?b ?l)
    )
    :effect (and
      (OnFloor ?r)
      (At ?r ?l)
      (not (On ?r ?b))
    )
  )


  (:action TurnOn
    :parameters (?r - robot ?s - switch ?b - box ?l - location)
    :precondition (and
      (On ?r ?b)
      (At ?b ?l)
      (At ?s ?l)
    )
    :effect (and
      (TurnedOn ?s)
    )
  )


  (:action TurnOff
    :parameters (?r - robot ?s - switch ?b - box ?l - location)
    :precondition (and
      (On ?r ?b)
      (At ?b ?l)
      (At ?s ?l)
      (TurnedOn ?s)
    )
    :effect (and
      (not (TurnedOn ?s))
    )
  )
)
```
**Plan**
```pddl
(go shakey shakey-start-loc door3-loc room3)
(go shakey door3-loc door1-loc corridor)
(go shakey door1-loc r1-loc-b2 room1)
(push shakey box2 r1-loc-b2 door1-loc room1)
(push shakey box2 door1-loc door2-loc corridor)
(push shakey box2 door2-loc r2-target-loc room2)
```

![[Pasted image 20250804152914.png]]
The Barman problem is a classic AI planning domain that models the task of a robotic bartender preparing cocktails. The world consists of a robot arm , a set of shakers, a set of glasses, and dispensers for various ingredients. The core challenge is to generate a sequence of actions to produce one or more specified cocktails according to recipes, while respecting the physical constraints. Key constraints include that the robot hand can only hold one object at a time , a new cocktail must be started in a clean shaker, and used shakers may need to be cleaned before they can be reused if resources are limited.

**Formalization of the Barman Problem**

| Component         | Description                                                                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Objects/Types** | robot-hand, shaker, glass, ingredient, dispenser, cocktail.                                                                                                              |
| **Predicates**    | Describes the state of the world. Examples: (handempty), (holding ?x), (clean ?s), (used ?s), (contains ?s ?i), (mixed ?s), (empty ?g), (served ?c ?g).                  |
| **Actions**       | Defines the possible operations. Key actions include: grasp-shaker, release-item, add-ingredient, mix-cocktail, grasp-glass, serve-cocktail, empty-shaker (to clean it). |
| **Initial State** | A specific setup. For example: The hand is empty, all shakers are clean and on the table, all glasses are empty and on the table.                                        |
| **Goal State**    | The desired outcome. For example: To have a specific cocktail served in a specific glass, represented by a fact like (served cocktail1 glass1).                          |
```pddl
;; PLAN FOR: barman-opt11-strips, problem file pfile1.pddl

(GRASP LEFT SHOT4)
(FILL-SHOT SHOT4 INGREDIENT1 LEFT RIGHT DISPENSER1)
(POUR-SHOT-TO-CLEAN-SHAKER SHOT4 INGREDIENT1 SHAKER1 LEFT L0 L1)
(CLEAN-SHOT SHOT4 INGREDIENT1 LEFT RIGHT)
(FILL-SHOT SHOT4 INGREDIENT2 LEFT RIGHT DISPENSER2)
(POUR-SHOT-TO-USED-SHAKER SHOT4 INGREDIENT2 SHAKER1 LEFT L1 L2)
(LEAVE LEFT SHOT4)
(GRASP LEFT SHAKER1)
(SHAKE COCKTAIL3 INGREDIENT1 INGREDIENT2 SHAKER1 LEFT RIGHT)
(POUR-SHAKER-TO-SHOT COCKTAIL3 SHOT1 LEFT SHAKER1 L2 L1)
(EMPTY-SHAKER LEFT SHAKER1 COCKTAIL3 L1 L0)
(CLEAN-SHAKER LEFT RIGHT SHAKER1)
(LEAVE LEFT SHAKER1)

;; PLAN FOR: barman-sat11-strips, problem file

(GRASP LEFT SHOT5)
(FILL-SHOT SHOT5 INGREDIENT3 LEFT RIGHT DISPENSER3)
(POUR-SHOT-TO-CLEAN-SHAKER SHOT5 INGREDIENT3 SHAKER1 LEFT L0 L1)
(CLEAN-SHOT SHOT5 INGREDIENT3 LEFT RIGHT)
(FILL-SHOT SHOT5 INGREDIENT1 LEFT RIGHT DISPENSER1)
(POUR-SHOT-TO-USED-SHAKER SHOT5 INGREDIENT1 SHAKER1 LEFT L1 L2)
(LEAVE LEFT SHOT5)
(GRASP LEFT SHAKER1)
(SHAKE COCKTAIL4 INGREDIENT3 INGREDIENT1 SHAKER1 LEFT RIGHT)
(POUR-SHAKER-TO-SHOT COCKTAIL4 SHOT1 LEFT SHAKER1 L2 L1)
(EMPTY-SHAKER LEFT SHAKER1 COCKTAIL4 L1 L0)
(CLEAN-SHAKER LEFT RIGHT SHAKER1)
(LEAVE LEFT SHAKER1)

;; PLAN FOR: barman-opt14-strips, problem file
(GRASP LEFT SHOT9)
(FILL-SHOT SHOT9 INGREDIENT3 LEFT RIGHT DISPENSER3)
(LEAVE LEFT SHOT9)

(GRASP LEFT SHOT10)
(LEAVE LEFT SHOT10)
(GRASP LEFT SHAKER1)
(SHAKE COCKTAIL4 INGREDIENT3 INGREDIENT4 SHAKER1 LEFT RIGHT)
(POUR-SHAKER-TO-SHOT COCKTAIL4 SHOT1 LEFT SHAKER1 L2 L1)  ; 
(POUR-SHAKER-TO-SHOT COCKTAIL4 SHOT7 LEFT SHAKER1 L1 L0)  ; 
(CLEAN-SHAKER LEFT RIGHT SHAKER1)
(LEAVE LEFT SHAKER1)

;; PLAN FOR: barman-sat14-strips, problem file

(GRASP LEFT SHOT13)
(FILL-SHOT SHOT13 INGREDIENT2 LEFT RIGHT DISPENSER2)
(LEAVE LEFT SHOT13)
(GRASP LEFT SHOT14)
(FILL-SHOT SHOT14 INGREDIENT3 LEFT RIGHT DISPENSER3)
(LEAVE LEFT SHOT14)
(GRASP LEFT SHOT5)
(FILL-SHOT SHOT5 INGREDIENT3 LEFT RIGHT DISPENSER3)
(POUR-SHOT-TO-CLEAN-SHAKER SHOT5 INGREDIENT3 SHAKER1 LEFT L0 L1)
(CLEAN-SHOT SHOT5 INGREDIENT3 LEFT RIGHT)
(FILL-SHOT SHOT5 INGREDIENT1 LEFT RIGHT DISPENSER1)
(POUR-SHOT-TO-USED-SHAKER SHOT5 INGREDIENT1 SHAKER1 LEFT L1 L2)
(LEAVE LEFT SHOT5)
(GRASP LEFT SHAKER1)
(SHAKE COCKTAIL4 INGREDIENT3 INGREDIENT1 SHAKER1 LEFT RIGHT)
(POUR-SHAKER-TO-SHOT COCKTAIL4 SHOT1 LEFT SHAKER1 L2 L1)
(EMPTY-SHAKER LEFT SHAKER1 COCKTAIL4 L1 L0)
(CLEAN-SHAKER LEFT RIGHT SHAKER1)
(LEAVE LEFT SHAKER1)

```


![[Pasted image 20250804150613.png]]
**Differences:**
- **Satisficing vs. Optimal**: The -sat versions  only require finding any valid plan that achieves the goal. This typically results in straightforward, sequential plans where cocktails are made one by one. In contrast, the -opt versions  require finding a plan with the minimum total-cost. This forces the planner to discover more intelligent strategies, such as making a full shaker of a cocktail and serving it into multiple glasses if the recipe is shared, thereby minimizing redundant actions and reducing cost.
- **Problem Scale** : The \*14 versions represent a significant increase in complexity over the \*11 versions. They feature more objects ( and a larger, more intricate set of goals. This progression tests a planner's ability to handle larger state spaces and more complex goal interactions, especially in the opt14 version, which combines large scale with the strict requirement for an optimal solution.

An additional approach I would try is **extending the domain to support parallel execution and resource diversity**. All current versions are bottlenecked by having a single shaker and only two hands. I would modify the PDDL domain to include multiple shakers and potentially more robotic hands. This would transform the problem from a sequential task into a multi-agent planning challenge. A planner would then need to generate a parallel plan, potentially having one hand mixing a cocktail in shaker1 while another hand simultaneously cleans shaker2, drastically improving efficiency and modeling a more realistic automated bar or factory environment.



![[Pasted image 20250804150626.png]]
a、
```pddl
(define (domain hanoi)
  (:requirements :strips)
  (:types disc peg)


  (:predicates
    (clear ?x - object)    
    (on ?d1 - disc ?d2 - object) ;
    (smaller ?d1 - disc ?d2 - disc) ; 
  )


  (:action move
    :parameters (?d - disc ?from - object ?to - object) ;
    :precondition (and
      (clear ?d)
      (on ?d ?from)
      (clear ?to)
      (imply (and (istype ?to disc)) (smaller ?d ?to))
    )
    :effect (and
      (not (on ?d ?from))
      (clear ?from)
      (on ?d ?to)
      (not (clear ?to))
    )
  )
)

```

b、
```pddl
(define (problem hanoi-3-discs)
  (:domain hanoi)
  (:objects
    d1 d2 d3 - disc  ; 
    p1 p2 p3 - peg   ;
  )

  (:init
    (clear p2) ; 
    (clear p3) ; 

    (smaller d3 d2)
    (smaller d3 d1)
    (smaller d2 d1)


    (on d1 p1)   
    (on d2 d1)  
    (on d3 d2)   


    (clear d3)
  )

  (:goal
    (and
      (on d1 p3)
      (on d2 d1)
      (on d3 d2)
    )
  )
)
```

**c、result**
```pddl
(MOVE D3 P1 P3)
(MOVE D2 P1 P2)
(MOVE D3 P3 P2)
(MOVE D1 P1 P3)
(MOVE D3 P2 P1)
(MOVE D2 P2 P3)
(MOVE D3 P1 P3)
```