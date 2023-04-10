# Unreal Chaos Car

- Enable Physics on Mesh
- Set wheel collision response to `Disabled`, as they will use ray-casting to interact with world
- Set wheel physics type to `Kinematic`

# Tips and Tricks

- Most of the legacy vehicle settings still work in UE5
- Physical material surfaces still affect chaos vehicles.
- Chaos allows for any number of wheels. (From motorcylces to semis)

# [`AWheeledVehiclePawn`](https://docs.unrealengine.com/4.26/en-US/API/Plugins/ChaosVehicles/AWheeledVehiclePawn/)

ChaosWheeledVehicle is the base wheeled vehicle pawn actor. By default it uses UChaosWheeledVehicleMovementComponent for its simulation, but this can be overridden by inheriting from the class and modifying its constructor like so: `Super(ObjectInitializer.SetDefaultSubobjectClass<UMyMovement>(VehicleComponentName))` Where UMyMovement is the new movement type that inherits from UChaosVehicleMovementComponent

# [`UChaosVehicleMovementComponent`](https://docs.unrealengine.com/4.26/en-US/API/Plugins/ChaosVehicles/UChaosVehicleMovementComponent/)

Subclass of `UPawnMovementComponent` 

## Setup interface:

```cpp
	// Setup

	/** Get our controller */
	virtual AController* GetController() const override;

	/** Get the mesh this vehicle is tied to */
	class UMeshComponent* GetMesh() const;

	/** Get Mesh cast as USkeletalMeshComponent, may return null if cast fails */
	USkeletalMeshComponent* GetSkeletalMesh();

	/** Get Mesh cast as UStaticMeshComponent, may return null if cast fails */
	UStaticMeshComponent* GetStaticMesh();

	/** Create and setup the Chaos vehicle */
	virtual void CreateVehicle();

	virtual TUniquePtr<Chaos::FSimpleWheeledVehicle> CreatePhysicsVehicle();

	/** Skeletal mesh needs some special handling in the vehicle case */
	virtual void FixupSkeletalMesh() {}

	/** Allocate and setup the Chaos vehicle */
	virtual void SetupVehicle(TUniquePtr<Chaos::FSimpleWheeledVehicle>& PVehicle);

	/** Do some final setup after the Chaos vehicle gets created */
	virtual void PostSetupVehicle();

	/** Adjust the Chaos Physics mass */
	virtual void SetupVehicleMass();

	void UpdateMassProperties(FBodyInstance* BI);

	/** When vehicle is created we want to compute some helper data like drag area, etc.... Derived classes should use this to properly compute things like engine RPM */
	virtual void ComputeConstants();
```

## Torque Curve

The engine torque curve.

- Can configure on instance, or create an external curve object
- Recommended curve: `(0, 0.5), (1000, 1.0), (4000, 0.8), (5000, 0.5)`
- Set max RPM to max X value in torque value (ie 5000)

## Throttle

Update the VehicleMovementComponent throttle in Tick:
```cpp
void AChaosKart::UpdateThrottle(const FInputActionValue& Value)
{
	float Throttle = Value.GetMagnitude();
	auto VehicleMovement = GetVehicleMovementComponent();
	VehicleMovement->SetThrottleInput(Throttle);
}

void AChaosKart::UpdateTurn(const FInputActionValue& Value)
{
	float Steering = Value.GetMagnitude();
	auto VehicleMovement = GetVehicleMovementComponent();
	VehicleMovement->SetSteeringInput(Steering);
}

void AChaosKart::UpdateBrake(const FInputActionValue& Value)
{
	float Brake = Value.GetMagnitude();
	auto VehicleMovement = GetVehicleMovementComponent();
	VehicleMovement->SetBrakeInput(Brake);
}
```

## Wheels

The Vehicle Movement component applies ChaosWheels Classes to selected bones in the skeleton.

### ChaosWheelClass

- **AxleType**: Front or Rear
- **Radius** and **Width** -- should match the wheel dimensions
	- Tip: To quickly get dimensions, drop the mesh in the level, hit Alt+J to switch to top wireframe and use middle mouse to measure
- **Affected By Steering** - Turns with steering
- **Affected By Engine** - 

# Vehicle Animation

- Create an Animation Blueprint from the vehicle skeletal mesh, and make its parent class a **`VehicleAnimationInstance`**

- In the AnimGraph create the following: **Mesh Space Ref Pose** -> **Wheel Controller** -> *(Component to local)* -> **Output Post**. Then assign the AnimBP in the Chaos Car Pawn details, and *Voila!*

## Suspension Animation

[Driving Around: Exploring Chaos Vehicles | State of Unreal 2022](https://youtu.be/Wc6lUXOhRO0?t=713)

# Vehicle Settings

## Center of Mass

- Enable **Center of Mass Override** to change center of mass and adjust accordingly. 
	- Z-axis should always be > 0. (50.0) is a good start
	- X-axis affects vehicle cornering (ei)

## Cornering

- To adjust cornering (turning) start with Wheel *Friction Force Multipliers*, and then other wheel parameters for fine-tuning. 

- Adjust the friction force between the front and back wheels to balance between understeering and oversteering

- Differential types, brakes, torque, all affect vehicle dynamics.

- Cornering Stiffness, Slide Slip Modifier, Slip Threshold and Skid Threshold all fine-tune cornering. As well as suspension settings, etc..

## Vehicle Debugger

Can use debugger via console:

- Legacy Vehicle debugger: 
	- `showdebug vehicle` 
	- `p.Vehicle.NextDebugPage` - cycle through debug pages

- Chaos Debugger
	- `p.Chaos.DebugDraw.Enabled 1` - enables chaos debugger
	- Useful debuffers:
		- `p.Vehicle.ShowCOM 1` - shows center of mass
		- `p.Vehicle.ShowAllForces 1`
		- `p.Vehicle.ShowWheelForces 1`
		- `p.Vehicle.ShowSuspensionForces 1`
		- `p.Vehicle.SHowSuspensionRaycasts 1`