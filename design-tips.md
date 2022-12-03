# Design Tips

## Game Design Tips

### Shooting

- For 3rd and even 1st person shooter, the projectile should aim toward the center of screen, not necessarily the barrel direction.

## Unreal Design Tips

### Character Death

Detach Controller from Characters on Death, and disable collision (if necessary). 

Example:
	```
	DetachFromControllerPendingDestroy();
	GetCapsuleComponent()->SetCollisionEnabled(ECollisionEnabled::NoCollision);
	```