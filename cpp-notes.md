# C++

## Table of Contents

- [Miscellaneous](#miscellaneous)
- [Useful Functions](#useful-functions)

# Miscellaneous

## Pointer vs References

- Reference cannot be reassigned
- Example: `float& damageRef = 1.0`
  - This means that `damageRef` refers to the memory storing 1.0 
  - Reassigning a value to `damageRef` will change the value in memory

## pure, const, pure const

- **Note: Not sure `pure` exists outside UE Editor**

- `const` keyword with compile time enforcement that prevents data from being mutated. Variables, functions and methods can be `const`

- `pure` is a promise that function does not mutate values or have other side effects

- `pure const` is pure function enforcement at compile time  

- *impure* just means not pure

## virtual and override

### `virtual`
- Declaring a base method virtual signals to the compiler that calls should check for inherited class implementation. Otherwise base class calls to a child class object will call the base class implementation.

### `override`
- This signals to the compiler (and developer) that the method we're implementing is overriding a virtual method. 
- Will not compile if there's no overridden method, or if overridden method is not virual

**Note:** Make sure to consider when you want to call super methods or not

# Useful Functions

- `atan2` vs `atan` 
  
  - `atan2(y/x)` will resolve negative angles, and therefore work for full 360 degrees, `atan(y/x)` only works for 180 degrees

- `ceil()` - Rounds a float up to an integer 


