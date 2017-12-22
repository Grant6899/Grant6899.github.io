---
layout:		post
title:		Type Conversion and Casting functions
subtitle:
date:		2017-12-19
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

## Type Conversion

A conversion operator is a special kind of member function that converts a value of a class type to a value of some other type. A conversion function typically has the general form:
```c++
operator type() const;
```

```
class SmallInt {
public:
	SmallInt(int i = 0): val(i){
		if (i < 0 || i > 255)
			throw std::out_of_range("Bad SmallInt value");
	}
	operator int() const { return val; }
private:
	std::size_t val;
};
```
- Conversion operators have no explicitly stated return type and no parameters
- They must be defined as member functions
- Conversion operations ordinarily should not change the object they are converting, usually should be defined as const members

Here's a bad example:
```c++
class SmallInt;
operator int(SmallInt&); // error: non-member

class SmallInt {
public:
	int operator int() const; // error: return type
	operator int(int = 0) const; // error: parameter list
	operator int*() const { return 42; } // error: 42 is not a pointer
};
```

If the conversion operator is explicit, we can still do the conversion. However, with one exception, we must do so explicitly through a cast:

```c++
class SmallInt {
public:
	// the compiler won't automatically apply this conversion
	explicit operator int() const { return val; }	// other members as before
};
```
To use it:
```c++
SmallInt si = 3; // ok: the SmallInt constructor is not explicit
si + 3; // error: implicit conversion is required, but operator int is explicit
static_cast<int>(si) + 3; // ok: explicitly request the conversion
```

**Explicit and implicit constructors can convert other types to the current class. Vice versa, explicit and implicit conversion operator can convert current class to other types.**


**Conversion to bool is usually intended for use in conditions. As a result, operator bool ordinarily should be defined as explicit.**

## Casting functions

### C-style cast
```c++
(T) expression // cast expression to be of type T
```

### Fucntion-style cast
```c++
T(expression) // cast expression to be of type T
```

### C++'s casting functions

- const_cast
	```c++
    const_cast<T>(expression)
    ```
	const_cast is typically used to **cast away** the constness of objects. It is the **only** C++-style cast that can do this.

- dynamic_cast
	```c++
    dynamic_cast<T>(expression)
    ```
    dynamic_cast is primarily used to perform “safe downcasting,” i.e., to determine whether an object is of a particular type in an inheritance hierarchy. It is the only cast that cannot be performed using the old-style syntax. It is also the only cast that may have **a significant runtime cost**. 
    
- reinterpret_cast
	```c++
    reinterpret_cast<T>(expression)
    ```
    reinterpret_cast is intended for low-level casts that yield implementation-dependent (i.e., unportable) results, e.g., casting a pointer to an int. **Such casts should be rare outside low-level code.**
    
- static_cast
	```c++
    static_cast<T>(expression)
    ```
    static_cast can be used to force implicit conversions (e.g., **non-const object to const object**, int to double, etc.). It can also be used to perform the reverse of many such conversions (e.g., void* pointers to typed pointers, pointer-to-base to pointer-to-derived), though it cannot cast from const to non-const objects. (Only const_cast can do that.)
    

## Rules and Conventions

- ### The old-style casts continue to be legal, but the new forms are preferable

	- Easier to identify (both for humans and "grep")
	- The more narrowly specified purpose of each cast, the easier for compilers to diagnose usage errors

	The only case that an old-style cast is preferable is when calling an explicit constructor to pass an object to a function:
	
    ```c++
    class Widget {
	public:
	explicit Widget(int size);
	...
	};
	void doSomeWork(const Widget& w);
	doSomeWork(Widget(15));
    ```

- ### Type conversions often lead to code that is executed at runtime
	The behavior of the generated code varies across different platforms and compilers

- ### Slicing function calling
	
    Wrong example:
	```c++
    class Window { // base class
	public:
		virtual void onResize() { ... } // base onResize impl
	...
	};
    
	class SpecialWindow: public Window { // derived class
	public:
		virtual void onResize() { // derived onResize impl;
		static_cast<Window>(*this).onResize(); // cast *this to Window, then call its onResize; this doesn’t work!
		... // do SpecialWindow- specific stuff
	};
    ```
	The cast creates a new, temporary copy of the base class part of *this, then invokes onResize on the copy.
    
    Correct way:
    ```c++
    class SpecialWindow: public Window { // derived class
	public:
		virtual void onResize() { // derived onResize impl;
		Window::onResize(); // call Window::onResize on *this
		... // do SpecialWindow- specific stuff
	};
    ```

- ### Avoid using dynamic_cast because it's expensive

	The need for dynamic_cast generally arises because you want to perform derived class operations on what you believe to be a derived class Implementations object, but you have **only a pointer- or reference-to-base** through which to manipulate the object. 

	The best solution is to use containers that store pointers altogether with virtual functions provided in classes pointed to. By this way, **dynamic binding** can work seamlessly with those pointers and you don't need to care about casting at all. You can also save all objects into the same container meanwhile.
    
    ```c++
    class Window {
	public:
		virtual void blink() {} // default impl is no-op;
		... // see Item34 for why
	}; // a default impl may be a bad idea
	
    class SpecialWindow: public Window {
	public:
		virtual void blink() { ... } // in this class, blink
		... // does something
	};
	
    typedef std::vector<std::tr1::shared_ptr<Window> > VPW;
	VPW winPtrs; // container holds
	// (ptrs to) all possible Window types
	for (VPW::iterator iter = winPtrs.begin();iter != winPtrs.end();++iter) // note lack of dynamic_cast
    	(*iter)->blink();
    ```
    
    
    
