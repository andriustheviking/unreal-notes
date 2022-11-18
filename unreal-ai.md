# Unreal AI

## Table of Contents

- [AI Controllers](#ai-controllers)
- [Behavior Tree / Blackboard](#behavior-tree--blackboard)
- [Nav Mesh](#nav-mesh)

---

# AI Controllers

- AI Controllers are automatically created and assigned to non-player character blueprints manually placed in a level.

- **Note:** By default, characters spawned at runtime have **autoposses OFF**. Can set to automatically have ON in BP Details

- AI Movement requires an [AI Navigation Mesh](#nav-mesh)

### AI Controller Methods:

- `SetFocus()` / `SetFocalPoint()`
  
  - Sets what the AI is looking at. 
  - Takes a priorty level `EAIFocusPriority` with three different levels 

- `ClearFocus`

- `MoveToActor()` / `MoveToLocation()`

- `LineOfSight()`

- `UBlackboardComponent* GetBlackboardComponent()` - returns the [UBlackboardComponent](#ublackboardcomponent)

### Using Behavior Tree and Blackboard

- `RunBehaviorTree(UBehaviorTree)` - starts executing [behavior tree](#behavior-tree)

- `UBlackboardComponent* GetBlackBoardComponet()` - gets the AI Controller's BB

# Behavior Tree / Blackboard

One way to implement AI is to use Unreal's BT/BB architecture.

The Blackboard receives GameWorld information which the Blackboard consumes to drive behavior.

## Behavior Tree

- [Documentation](https://docs.unrealengine.com/5.1/en-US/behavior-trees-in-unreal-engine/)

- A BT holds a reference to a BB object. *(This is assignable in BT details panel, and will even autofill)*

- Can view BT of AIControllers live in Editor, selecting the specific controller via a dropdown.

### Behavior Tree Graph

- The BT graph has a root node that can only have one child node. From there we build the behavior tree.

- Behavior Tree Nodes:

  - **Selector**
    - A Selector executed child node in order left to right until a child node "succeeds"

    - **Decorator** is a task that succeeds or fails

      - **Blackboard** can check blackboard values to return success/failure
        - **Flow Control**
          - **Observer Aborts** - How to handle condition changing while active
            - **None**
            - **Self** - If condition becomes **false**, stop execution and reevaluates Selector
            - **Lower Piority** - If condition becomes **true**, during another sequence, stop that sequence and execute this one.
            - **Both** - Does both.

  - **Sequence**
    - Sequences execute all child nodes in order left to right

  - **Task**
    - A Task is a single action (ie Move To, Wait, ...)

### `UBehaviorTree`

## Blackboard

[Documentation](https://docs.unrealengine.com/4.26/en-US/BlueprintAPI/AI/Components/Blackboard/)

### `UBlackboardComponent`

- `#include "BehaviorTree/BlackboardComponent.h"`

- Blackboards store values as a map, with SetValue GetValue methods:
  - Generic typedef: `SetValue<T>(FName&, T)` and `T GetValue<T>(FName&)` 
  - Utility Setter / Getters Examples:
    - `UObject* GetValueAsObject(FName&)` / `SetValueAsObject(FName&, UObject*)` 
    - `UClass* GetValueAsClass(FName&)` / `SetValueAsClass(FName&, UClass*)`
    - Value Types 
      - `bool GetValueAsBool(FName&)` / `SetValueAsBool(FName&, bool)` 
      - etc..
    - `ClearValue(FName&)`

# Nav Mesh

AI Requires a **NavMesh** to layout valid paths. These are created using **Nav Mesh Bounds Volume** in the editor. It's a volume that automatically calculates navigability of geometry within the volume. **Tip:** Show > Navigation will highlight navigable area as green.

**PathFollowingComponent** Component in AIController
