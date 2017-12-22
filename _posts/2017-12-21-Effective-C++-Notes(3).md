---
layout:		post
title:		Effective C++ Notes(3)
subtitle:
date:		2017-12-21
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

## Item 26: Postpone variable definitions as long as possible

### Consider when to define variable when taking exception into consideration
```c++
// this function defines the variable "encrypted" too soon
std::string encryptPassword(const std::string& password){
	using namespace std;
	string encrypted;
	if (password.length() < MinimumPasswordLength) {
		throw logic_error("Password is too short");
	}
	... // do whatever is necessary to place an
	// encrypted version of password in encrypted
	return encrypted;
}

// this function postpones encrypted’s definition until it’s truly necessary
std::string encryptPassword(const std::string& password){
	using namespace std;
	if (password.length() < MinimumPasswordLength) {
		throw logic_error("Password is too short");
	}
	string encrypted;
	... // do whatever is necessary to place an
	// encrypted version of password in encrypted
	return encrypted;
}
```

### Postpone the definition until you have initialization arguments for it.
```c++
// this function postpones encrypted’s definition until it’s necessary, but it’s still needlessly inefficient
std::string encryptPassword(const std::string& password){
	... // import std and check length as above
	string encrypted; // default-construct encrypted
	encrypted = password; // assign to encrypted
	encrypt(encrypted);
	return encrypted;
}

// finally, the best way to define and initialize encrypted
std::string encryptPassword(const std::string& password){
	... // import std and check length
	string encrypted(password); // define and initialize via copy constructor
	encrypt(encrypted);
	return encrypted;
}
```

### Variable definition inside loop vs outside loop

```c++
// Approach A: define outside loop 
Widget w;
for (int i = 0; i < n; ++i) { 
	w = some value dependent on i; 
}
... 

// Approach B: define inside loop
for (int i = 0; i < n; ++i) {
	Widget w(some value dependent on i);
} 
...
```
- Approach A: 1 constructor + 1 destructor + n assignments.
- Approach B: n constructors + n destructors.

Unless you know that:
- (1) assignment is less expensive than a constructor-destructor pair and 
- (2) you’re dealing with a performance-sensitive part of your code 

you should default to using Approach B.


## Item 27: Minimize casting

- Avoid casts whenever practical, especially dynamic_casts in performance-sensitive code. If a design requires casting, try to develop a cast-free alternative.
- When casting is necessary, try to hide it inside a function. Clients can then call the function instead of putting casts in their own code.
- Prefer C++-style casts to old-style casts. They are easier to see, and they are more specific about what they do.

Please refer [Type Conversion and Casting](https://grant6899.github.io/2017/12/19/Type-Conversion-and-Casting-functions/) to see more details.














