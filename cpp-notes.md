# C++

## Table of Contents

- [Memory](#memory)
- [Useful Functions](#useful-functions)

# Memory

- Pointer vs References 
  - Reference cannot be reassigned
  - Example: `float& damageRef = 1.0`
    - This means that damageRef refers to the memory storing 1.0 
    - Reassigning a value to damageRef will change the value in memory

# Useful Functions

- `atan2` vs `atan` 
  
  - `atan2(y/x)` will resolve negative angles, and therefore work for full 360 degrees, `atan(y/x)` only works for 180 degrees

- `ceil()` - Rounds a float up to an integer 
