# C++

## Table of Contents

- [Useful Functions](#useful-functions)
- [Miscellaneous](#miscellaneous)

# Useful Functions

- `atan2` vs `atan` 
  
  - `atan2(y/x)` will resolve negative angles, and therefore work for full 360 degrees, `atan(y/x)` only works for 180 degrees

- `ceil()` - Rounds a float up to an integer 

# Miscellaneous

## Function Declaration:

`void foo()` *Obsolete* Means "a function foo taking an unspecified number of arguments of unspecified type". 
`void foo(void)` Means "a function foo taking no arguments"

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

## Abstract Class

A class is made **abstract** by declaring at least one of its functions as `pure virtual` function. 

Abstract classes cannot be used to instantiate objects and serves only as an *interface*.  Failure to override a pure virtual function in a derived class, then attempting to instantiate objects of that class, is a compilation error.

A pure virtual function is specified by placing `= 0` in its declaration as follows:

```
class Box {
   public:
      // pure virtual function
      virtual double getVolume() = 0;
  private:
      double length;      // Length of a box
//...
```

### Multiple class inheritance

C++ supports multi clas inheritance. [https://www.geeksforgeeks.org/multiple-inheritance-in-c/](https://www.geeksforgeeks.org/multiple-inheritance-in-c/)

## Casting

[StackOverflow](https://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used)

  - `static_cast<T>(expression)` 
    - Converts between types using a combination of implicit and user-defined conversions. 
    - Use `static_cast` for ordinary type conversions.
    - Does one address offset at runtime (low runtime impact) and no safety checks that a downcast is correct.
  
  - `dynamic_cast<T>(expression)` 
    - Safely converts pointers and references to classes up, down, and sideways along the inheritance hierarchy. 
    - Use `dynamic_cast` for converting pointers/references within an inheritance hierarchy.
    - Does the same address offset at runtime like `static_cast`, but also and an expensive safety check that a downcast is correct using RTTI.
    - This safety check allows you to query if a base class pointer is of a given type at runtime by checking a return of nullptr which indicates an invalid downcast. Therefore, if your code is not able to check for that nullptr and take a valid non-abort action, you should just use static_cast instead of dynamic cast.
  
  - `const_cast<T>(expression)`
    - only `const_cast` may be used to cast away (remove) constness or volatility.
    - Avoid this unless you are stuck using a const-incorrect API.

  - `reinterpret_cast<T>(expression`
    - Converts between types by reinterpreting the underlying bit pattern. 
    - Use `reinterpret_cast` for low-level reinterpreting of bit patterns. Use with extreme caution
    - Does nothing at runtime, not even the address offset. The pointer must point exactly to the correct type, not even a base class works. You generally don't want this unless raw byte streams are involved.
    - It is purely a compile-time directive which instructs the compiler to treat expression as if it had the type new-type. 

  - `(type)value` (C-style cast)
    - When the C-style cast expression is encountered, the compiler attempts to interpret it as the following cast expressions, in this order: `const_cast`, `static_cast`, `reinterpret_cast`

  - `type(value)` (function-style cast)
    - Converts between types using a combination of explicit and implicit conversions. 

