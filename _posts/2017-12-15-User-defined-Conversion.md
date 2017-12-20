---
layout:		post
title:		User-defined Conversion
subtitle:
date:		2017-12-15
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

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

**Conversion to bool is usually intended for use in conditions. As a result, operator bool ordinarily should be defined as explicit.**
