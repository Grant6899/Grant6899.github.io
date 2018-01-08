---
layout:		post
title:		Top-level const and Low-level const
subtitle:
date:		2018-01-07
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - const
---

## Top-level const and Low-level const

- Top-level const: indicate that the pointer itself is a const. Top-level const can appear in any object type, i.e., one of the built-in arithmetic types, a class type, or a pointer type.

- Low-level const: a pointer that points to a const object. Low-level const appears in the base type of compound types such as **pointers or references**.

Note that pointer types, unlike most other types, can have both top-level and low-level const independently.


### Copying rules


The distinction between top-level and low-level matters when we copy an object. 
**When we copy an object, top-level consts are ignored, but low-level const is never ignored.** 

When we copy an object, both objects must have 
1. the same low-level const qualification **or** 
2. LHS has more strict constness than RHS (In other words, there must be a conversion between the types of the two objects. In general, we can convert a nonconst to const but not the other way round)

**Address of a const object is a low-level const**

```c++
int i = 0;
int *const p1 = &i; // p1 has top-level const, &i has no const
const int ci = 42; // we cannot change ci; const is top-level
const int *p2 = &ci; // p2 has low-level const, &ci has low-level const
const int *const p3 = p2; // right-most const is top-level, left-most is low-level
const int &rc = ci; // const in reference types is always low-level
```

| Variable  | Type | Top-level const  | Low-level const  |
|---|---|---|---|
| i   | int  | non-const  | N/A  |
| ci  | int  | const  | N/A  |
| p   | int* | non-const | non-const |
| p1  | int* const  | const   | non-const  |
| p2  | const int* | non-const | const |
| p3  | const int * const | const | const |
| r  | int& | N/A | non-const |
| rc| const int& | N/A | const |


```c++
i = ci; // ok: copying the value of ci; top-level const in ci is ignored 
p2 = p3; // ok: pointed-to type matches; top-level const in p3 is ignored

int *p = p3; // error: p3 has a low-level const but p doesn't
p2 = p3; // ok: p2 has the same low-level const qualification as p3 
p2 = &i; // ok: we can convert int* to const int*, p2 has low-level const but &i doesn't
```

Special cases for reference:
```c++
int &r = ci; // error: can't bind an ordinary int& to a const int object 
const int &rc = i; // ok: can bind const int& to plain int
```

## The *auto* Type Specifier

Unlike type specifiers, such as *double*, that name a specific type, *auto* tells the compiler to deduce the type from the initializer. 

By implication, a variable that uses *auto* as its type specifier **must have an initializer**.

Attention:
- **When we use a reference, we are really using the object to which the reference refers**
- *auto* ordinarily ignores top-level const

```c++
const int ci = i, &cr = ci;
auto b = ci; // b is an int (top-level const in ci is dropped)
auto c = cr; // c is an int (cr is an alias for ci whose const is top-level) auto d = &i; // d is an int*(& of an int object is int*)
auto e = &ci; // e is const int*(& of a const object is low-level const)
```

## The *decltype* Type Specifier

*decltype* returns the type of its operand. The compiler analyzes the expression to determine its type but does not evaluate the expression.

```c++
const int ci = 0, &cj = ci;
decltype(ci) x = 0; // x has type const int
decltype(cj) y = x; // y has type const int& and is bound to x 
decltype(cj) z; // error: z is a reference and must be initialized
```

When we apply decltype to an expression that is not a variable, we get the type that that expression yields.
```c++
// decltype of an expression can be a reference type
int i = 42, *p = &i, &r = i;
decltype(r + 0) b; // ok: addition yields an int; b is an (uninitialized) int 
decltype(*p) c; // error: c is int& and must be initialized
```


Deduction done by decltype depends on the form of its given expression.

When we apply decltype to a variable without any parentheses, we get the type of that variable. If we wrap the variable’s name in one or more sets of parentheses, the compiler will evaluate the operand as an expression. A variable is an expression that can be the left-hand side of an assignment. As a result, decltype on such an expression yields a reference:
```c++
// decltype of a parenthesized variable is always a reference 
decltype((i)) d; // error: d is int& and must be initialized 
decltype(i) e; // ok: e is an (uninitialized) int
```

Attention:

**Remember that decltype((variable)) (note, double parentheses) is always a reference type, but decltype(variable) is a reference type only if variable is a reference.**