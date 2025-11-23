# Defining Variables and Expressions

## Header file and namespace
To use QUBO++, you need to include the header file `qbpp.hpp` and use the `qbpp` namespace.

## Defining variables and expressions
You can define a variable by calling `qbpp::var("variable_name")` with `auto` type deduction.
Expressions are constructed using standard arithmetic operators such as `+`, `-`, and `*`.

The following sample program defines three variables `a`, `b`, and `c`, and an expression `f`, which is printed using `std::cout`:
```cpp
#include "qbpp.hpp"

int main() {
  auto a = qbpp::var("a");
  auto b = qbpp::var("b");
  auto c = qbpp::var("c");
  auto f = (a + b - 1) * (b + c - 1);
  std::cout << "f = " << f << std::endl;
}
```
The expression `(a + b - 1) * (b + c - 1)` is automatically expanded and stored in `f`.

In this QUBO++ program, the variables `a`, `b`, and `c` are objects of class `qbpp::Var`, and the expression f is an object of class `qbpp::Expr`.

Assuming the header and library paths are properly set up, this program (saved as `test.cpp`) can be compiled with `g++` as follows:
```bash
$ g++ test.cpp -o test
```
Running the executable prints the expanded expression:
```bash
$ ./test
f = 1 +a*b +b*b +a*c +b*c -a -b -b -c
```

## Simplifying expression
The expression stored in a `qbpp::Expr` object can be simplified by calling the `simplify()` member function:
```cpp
  std::cout << "f = " << f.simplify() << std::endl;
```
With this change, the output of the program becomes:
```txt
f.simplify() = 1 -a -2*b -c +a*b +a*c +b*b +b*c
```
Assuming that all variables take binary values (0 or 1), we can use the identity 
$b^2=b$ to further simplify the expression.
For this purpose, we use simplify_as_binary() instead:
```cpp
  std::cout << "f = " << f.simplify_as_binary() << std::endl;
```
Then the output becomes:
```txt
f = 1 -a -b -c +a*b +a*c +b*c
```

The simplify functions reorder the variables within each term and the terms within the expression so that lower-degree terms appear first, and terms of the same degree are sorted in the lexicographical order of their variables.
The variables themselves are ordered according to the order in which they were defined.


## Additional note

### Simplifying expressions with spin variables
If variables are assumed to take spin values $-1$/$+1$, the identity $b^2 = 1$ can be used to further simplify the expression.
In this case, the expression can be simplified using the `simplify_as_spin()` member function:
```cpp
  std::cout << "f = " << f.simplify_as_spin() << std::endl;
```
Then the output becomes:
```txt
f = 2 -a -2*b -c +a*b +a*c +b*c
```

### Global functions for simplification
Member functions update the expression stored in `f`.
If you do not want to modify `f`, you can instead use the global functions
`qbpp::simplify(f)`, `qbpp::simplify_as_binary(f)`, and `qbpp::simplify_as_spin(f)`, which return the simplified expressions without changing `f`.
