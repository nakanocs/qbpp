---
layout: default
title: "Reference Variables"
---

# Quick Reference: Variables and Expressions

## Data types used in `qbpp::Expr`
- **`coeff_t`**:  
  The integer data type used for coefficients in `qbpp::Term` objects.  
  The default type is `int32_t`.  
  To change this type, define the `COEFF_TYPE` macro at compile time, for example:
```
-DCOEFF_TYPE=int16_t
```
- **`energy_t`**:  
The integer data type used to compute energy values of `qbpp::Expr` objects,  
as well as for integer constant terms in `qbpp::Expr`.  
The default type is `int64_t`.  
To change this type, define the `ENERGY_TYPE` macro at compile time, for example:
```
-DENERGY_TYPE=int32_t
```
The bit width of `energy_t` is guaranteed to be equal to or larger than that of `coeff_t`.

- **`vindex_t`**:  
Defined as `uint32_t` and used to store a unique integer ID for each `qbpp::Var` object.  
In most cases, it is not necessary to change this data type.

## Available integer data types
- **Standard integer types**:  
`int8_t`, `int16_t`, `int32_t`, `int64_t`

- **Multiprecision integer types** (implemented using the Boost.Multiprecision library):  
`qbpp::int128_t`, `qbpp::int256_t`, `qbpp::int612_t`, `qbpp::int1024_t`, `qbpp::cpp_int`

- **`qbpp::cpp_int`**:  
An integer type with unlimited precision.

> **WARNING**  
> To maximize performance, QUBO++ does not check for arithmetic overflow.  
> During development and testing, it is recommended to use wider bit widths for
> `coeff_t` and `energy_t`.  
> If the required bit widths are unclear, use `qbpp::cpp_int` to ensure correctness,
> and switch to fixed-width integer types after validation.

## Printing class objects
Most classes in QUBO++ can be printed using the `<<` operator with `std::ostream`,
which is useful for debugging.
For example, an object `obj` in QUBO++ can be printed to `std::cout` as follows:
```cpp
std::cout << obj << std::endl;
```
This invokes either `obj.str()` or `str(obj)`, which returns a std::string
containing a textual representation of obj.
This design allows easy inspection of internal states without relying on a debugger.


## Variable classes
- **`qbpp::Var`**:
  A class that holds a unique 32-bit integer ID and a string representing the variable name.


> **NOTE**  
> A `qbpp::Var` object represents a variable symbolically.  
> No specific data type is associated with it.  
> It can be used to represent binary, spin, or other types of variables.

### Variable creation functions
The following functions are provided to create variables:

- **`qbpp::var("name")`**:  
  Creates a `qbpp::Var` object with the given name `"name"`.

- **`qbpp::var("name", s1)`**:  
  Creates a one-dimensional array (vector) of `qbpp::Var` objects with the base name `"name"`.  
  Each element is represented as `name[i]`.  
  The resulting type is `qbpp::Vector<qbpp::Var>`.

- **`qbpp::var("name", s1, s2)`**:  
  Creates a two-dimensional array (matrix) of `qbpp::Var` objects with the base name `"name"`.  
  Each element is represented as `name[i][j]`.  
  The resulting type is `qbpp::Vector<qbpp::Vector<qbpp::Var>>`.

- **`qbpp::var("name", s1, s2, ...)`**:  
  Creates a higher-dimensional array of `qbpp::Var` objects with the base name `"name"`.  
  Each element is represented as `name[i][j]...`.  
  The resulting type is a nested `qbpp::Vector`.

> **NOTE**  
> If `"name"` is omitted, numbered names such as `"{0}"`, `"{1}"`, ... are automatically assigned in creation order.

## `qbpp::Var` member functions
For a `qbpp::Var` instance `x`, the following member functions are available:

- **`std::string x.str()`**:  
  Returns the name of `x`.

- **`vindex_t x.index()`**:  
  Returns the unique integer ID of `x`.

Usually, there is no need to call these member functions explicitly in QUBO++ programs.

## Integer variable class
- **`qbpp::VarInt`**:  
  A class derived from `qbpp::Expr` that represents an integer variable with a specified range.

### Integer variable creation functions
The following functions are provided to create integer variables:

- **`qbpp::var_int("name")`**:  
  Returns an internally used helper object and does not create a `qbpp::VarInt` by itself.  
  To define a `qbpp::VarInt`, the range must be specified using the `<=` operator, as shown below.

- **`l <= qbpp::var_int("name") <= u`**:  
  Here, `l` and `u` must be integers.  
  This expression creates a `qbpp::VarInt` object with the name `"name"`,  
  which internally contains a `qbpp::Expr` object representing all integers in the range `[l, u]`.  
  Internally, this also creates `qbpp::Var` objects used in the underlying expression.

- **`l <= qbpp::var_int("name", s1) <= u`**:  
  Creates a one-dimensional array (vector) of `qbpp::VarInt` objects with the base name `"name"`  
  and the same range `[l, u]`.  
  Each element is represented as `name[i]`.  
  The resulting type is `qbpp::Vector<qbpp::VarInt>`.  
  Higher-dimensional arrays of `qbpp::VarInt` objects can be created in the same way as `qbpp::Var` objects.

- **`qbpp::var_int("name") == k`**:
  Creates a `qbpp::VarInt` object named `"name"` that represents the constant integer value `k`.
  Typically, `k` is set to 0, and the variable is later updated to a desired range.

- **`qbpp::var_int("name", s1) == k`**:
  Creates a one-dimensional array (vector) of `qbpp::VarInt` objects with the base name `"name"`,
  each representing the constant integer value `k`.
  Higher-dimensional arrays are also supported.

### Integer variable member functions
For a `qbpp::VarInt` instance `x`, the following member functions are available:

- **`std::string x.name()`**:  
  Returns the name of `x`.

- **`std::string x.str()`**:  
  Returns the string representation of the underlying expression.

- **`energy_t x.min_val()`**:  
  Returns the minimum value `l` of `x`.

- **`energy_t x.max_val()`**:  
  Returns the maximum value `u` of `x`.

- **`const qbpp::Vector<qbpp::Var>& x.vars()`**:  
  Returns the const reference of the `qbpp::Var` object vector used to represent the integer variable.

- **`const qbpp::Vector<energy_t>& x.coeffs()`**:  
  Returns the const reference of the integer coefficient vector.  
  
The following expression is equivalent to the expression stored in `x`:
```cpp
x.min_val() + qbpp::sum(x.coeffs() * x.vars())
```
