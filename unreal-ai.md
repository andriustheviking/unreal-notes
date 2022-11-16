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

# Nav Mesh

AI Requires a **NavMesh** to layout valid paths. These are created using **Nav Mesh Bounds Volume** in the editor. It's a volume that automatically calculates navigability of geometry within the volume. **Tip:** Show > Navigation will highlight navigable area as green.

**PathFollowingComponent** Component in AIController
