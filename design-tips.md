# Design Tips

## Game Design Tips

### Shooting

- For 3rd and even 1st person shooter, the projectile should aim toward the center of screen, not necessarily the barrel direction.

## Unreal Design Tips

### UI

Generally UI should be implemented on the PlayerController. Especially true for multiplayer games.

### Character Death

Detach Controller from Characters on Death, and disable collision (if necessary). 

**Note:** Make sure to call `DetachFromControllerPendingDestroy` after any work is needed by the Player Controller. For example, we can't get the PlayerController from a Pawn if it's been detached!

Example:
	```
	DetachFromControllerPendingDestroy();
	GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	```