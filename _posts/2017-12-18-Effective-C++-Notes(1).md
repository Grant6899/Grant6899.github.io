---
layout:		post
title:		Effective C++ Notes(1)
subtitle:
date:		2017-12-18
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

## Item 1: View C++ as a federation of languages

- **C**. Way down deep, C++ is still based on C. Blocks, statements, the preprocessor, built-in data types, arrays, pointers, etc., all come from C. 

- **Object-Oriented C++**. This part of C++ is what C with Classes was all about: classes (including constructors and destructors), encapsulation, inheritance, polymorphism, virtual functions (dynamic binding), etc. 

- **Template C++**. This is the generic programming part of C++, the one that most programmers have the least experience with. Template
considerations pervade C++, and it’s not uncommon for rules of good programming to include special template-only clauses.

- **The STL**. The STL is a template library, of course, but it’s a very special template library. Its conventions regarding containers, iterators, algorithms, and function objects mesh beautifully.

## Item 2: Prefer consts, enums, and inlines to #defines

Marcos are hard to debug since it's simple replacement.
- For simple constants, prefer const objects or enums to #defines. 
- For function-like macros, prefer inline functions to #defines.

## Item 3: Use const whenever possible


- Declaring something const helps compilers detect usage errors. const can be applied to objects at any scope, to function parameters and return types, and to member functions as a whole.
- Compilers enforce bitwise constness, but you should program using logical constness.
- When const and non-const member functions have essentially identical implementations, code duplication can be avoided by having the
non-const version call the const version.

## Item 4: Make sure that objects are initialized before they’re used

Reading uninitialized values yields undefined behavior. The best way to deal with this seemingly indeterminate state of affairs
is to always initialize your objects before you use them. 

- For nonmember objects of built-in types, you’ll need to do this manually. 
- For almost everything else, the responsibility for initialization falls on constructors. Make sure that all constructors initialize everything in the object. In a constructor, prefer use of the member initialization list to assignment inside the body of the constructor. List data members in the initialization list in the same order they’re declared in the class.

## Item 5: Know what functions C++ silently writes and calls

Compilers may implicitly generate a class’s default constructor, copy constructor, copy assignment operator, and destructor.

```c++
class Empty {
public:
	Empty() { ... } // default constructor
	Empty(const Empty& rhs) { ... } // copy constructor
	~Empty() { ... } // destructor — see below
	// for whether it’s virtual
	Empty& operator=(const Empty& rhs) { ... } // copy assignment operator
};

{
	Empty e1; // default constructor;
	Empty e2(e1); // copy constructor
	e2 = e1; // copy assignment operator
} // e1, e2's destructor when going out of scope
```

## Item 6: Explicitly disallow the use of compiler-generated functions you do not want

Before C++11, we can declare functions in class definition but not provide their implementations. In this way, complier will not generate them implicitly. 

After C++11, you can explicitly tell complier if a function should be generated by default:

```c++
class Empty {
public:
	Empty() = delete; // default constructor
	Empty(const Empty& rhs) = delete; // copy constructor
	~Empty() { ... } // destructor — see below
	// for whether it’s virtual
	Empty& operator=(const Empty& rhs) = delete; // copy assignment operator
};
```

## Item 7: Declare destructors virtual in polymorphic base classes

```c++
class ClxBase{
public:
    ClxBase() {};
    virtual ~ClxBase() {};

    virtual void DoSomething() { cout << "Do something in class ClxBase!" << endl; };
};

class ClxDerived : public ClxBase{
public:
    ClxDerived() {};
    ~ClxDerived() { cout << "Output from the destructor of class ClxDerived!" << endl; }; 

    void DoSomething() { cout << "Do something in class ClxDerived!" << endl; };
};
```

Call them by
```c++
ClxBase *pTest = new ClxDerived;
pTest->DoSomething();
delete pTest;
```

Output:
```
Do something in class ClxDerived!
Output from the destructor of class ClxDerived!
```

**If desctructor is not virtual, when you delete a derived object by a base class pointer, derived class's destructor will not be called.**

- Polymorphic base classes should declare virtual destructors. If a class has any virtual functions, it should have a virtual destructor.

- Classes not designed to be base classes or not designed to be used polymorphically should not declare virtual destructors.

## Item 8: Prevent exceptions from leaving destructors

- Destructors should never emit exceptions. If functions called in a destructor may throw, the destructor should catch any exceptions, then swallow them or terminate the program.
   
   - Swallow:
     ```c++
     DBConn::~DBConn(){
		try { db.close(); }
		catch (...) {
		//make log entry that the call to close failed;
		}
	 }
     ```
    
   - Terminate:
   	 ```c++
     DBConn::~DBConn(){
	  	 try { db.close(); }
		 catch (...) {
		 //make log entry that the call to close failed;
		 std::abort();
		 }
	 }
     ```
     
- If class clients need to be able to react to exceptions thrown during an operation, the class should provide a regular (i.e., non-destructor) function that performs the operation.     
    
## Item 9: Never call virtual functions during construction or destruction

- During base class construction, virtual functions never go down into derived classes. Instead, the object behaves as if it were of the base type. Because base class constructors execute before derived class constructors, derived class data members have not been initialized when base class constructors run.
     
- The same reasoning applies during destruction. Once a derived class destructor has run, the object’s derived class data members assume undefined values, so C++ treats them as if they no longer exist.

## Item 10: Have assignment operators return a reference to *this

Chain assignment in C++:

```c++
int x, y, z;
x = y = z = 15;
```

To make the chain assignment work, you should follow a certain convention when implementing assignment operator:
```c++
Widget& operator=(const Widget& rhs){ // return type is a reference to
...
return *this; // return the left-hand object
}
```

## Item 11: Handle assignment to self in operator=

- Make sure operator= is well-behaved when an object is assigned to itself. Techniques include comparing addresses of source and target objects, careful statement ordering, and copy-and-swap.

	- Identity test
		```c++
    	Widget& Widget::operator=(const Widget& rhs){
			if (this == &rhs) return *this; // identity test: if a self-assignment,
			// do nothing
			delete pb;
			pb = rhs.pb;
			return *this;
		}
    	```
	- Share underlying resource
	 
      above will make a shallow copy, make pointer of two instances point to the same underlying object.
	- Deep copy
		```c++
    	Widget& Widget::operator=(const Widget& rhs){
			if (this == &rhs) return *this; // identity test: if a self-assignment,
			// do nothing
			delete pb;
			pb = new Bitmap(*rhs.pb);
			return *this;
		}
    	```
	- Copy and swap
		```c++
        Widget& Widget::operator=(const Widget& rhs){
			Widget temp(rhs); // make a copy of rhs’s data
			swap(temp); // swap *this’s data with the copy’s
			return *this;
		}
        ```

- Make sure that any function operating on more than one object behaves
correctly if two or more of the objects are the same.

     
## Item 12: Copy all parts of an object
     
When you declare your own copying functions, you are indicating to compilers that there is something about the default implementations you don’t like. If you add a data member to a class, you need to make sure that you update the copying functions, too. 
     
**Derived class copying functions must invoke their corresponding base class functions.**

- Copy all local data members
- Invoke the appropriate copying function in all base classes

```c++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs) : Customer(rhs), // invoke base class copy ctor
priority(rhs.priority) // copy local member
{
	logCall("PriorityCustomer copy constructor");
}

PriorityCustomer& PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
	logCall("PriorityCustomer copy assignment operator");
	Customer::operator=(rhs); // assign base class parts
	priority = rhs.priority;
	return *this;
}
```
