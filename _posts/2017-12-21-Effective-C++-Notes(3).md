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

## Item 28: Avoid returning “handles” to object internals

- Don't make your program self-contradictory design
	```c++
    class Point { // class for representing points
	public:
		Point(int x, int y);
		...
		void setX(int newVal);
		void setY(int newVal);
		...
	};

	struct RectData { // Point data for a Rectangle
		Point ulhc; // ulhc = “ upper left-hand corner”
		Point lrhc; // lrhc = “ lower right-hand corner”
	};
	
    class Rectangle {
	...
	private:
		std::tr1::shared_ptr<RectData> pData; // see Item13 for info on
	}; // tr1::shared_ptr
    ```
	
    Below code is self-contradictory because designer means not to change point data by declaring it as const member function, but the point retured by reference, which is still able to be modified.
    ```c++
    class Rectangle {
	public:
		...
		Point& upperLeft() const { return pData->ulhc; }
		Point& lowerRight() const { return pData->lrhc; }
		...
	};
	```

- Take care of the risk that retured handle would outlive the boject  it refers to
	```c++
    class GUIObject { ... };
	const Rectangle // returns a rectangle by value
	boundingBox(const GUIObject& obj);
    ```
    
    ```c++
    GUIObject *pgo;
	const Point *pUpperLeft = &(boundingBox(*pgo).upperLeft());
    ```
	
	boundingBox’s return value — temp — will be destroyed, and that will indirectly lead to the destruction of temp’s Points. That, in turn, will leave pUpperLeft pointing to an object that no longer exists; pUpperLeft will dangle by the end of the statement that created it! 
    
    Exception:
    operator[] allows you to pluck individual elements out of strings and vectors, and these operator[]s work by returning references to the data in the containers — **data that is destroyed when the containers themselves are**.

## Item 29: Strive for exception-safe code

### Definition of exception safe
- Leak no resource
- Don't allow data structures to become corrupted

### Exception-safe function offers guarantees

- Basic gurantee: if an exception is thrown, everything in the program remains in a valid state
- Sttrong guarantee: if an exception is thrown, the state of the program is unchanged
- No guarantee: promise never to throw exceptions, because they always do what they promise to do

### Best Practice:

- Use RAII trick to take care of resources to avoid leak.
- Not to change the status of an object to indicate that something has happened until something actually has.
- The strong guarantee can often be implemented via copy-and-swap, but the strong guarantee is not practical for all functions.
- A function can usually offer a guarantee no stronger than the weakest guarantee of the functions it calls.

## Item 30: Understand the ins and outs of inlining

- Virtual functions cannot be inline functions since virtual function doesn't know its implementation until running time, while inline function is known at complilation.
- Limit most inlining to small, frequently called functions. This facilitates debugging and binary upgradability, minimizes potential code bloat, and maximizes the chances of greater program speed.
- Don’t declare function templates inline just because they appear in header files.

## Item 31: Minimize compilation dependencies between files

- The general idea behind minimizing compilation dependencies is to depend on declarations instead of definitions. Two approaches
based on this idea are Handle classes and Interface classes.
- Library header files should exist in full and declaration-only forms. This applies regardless of whether templates are involved.


## Item 32: Make sure public inheritance models “is-a”

- Public inheritance means “is-a.” Everything that applies to base classes must also apply to derived classes, because every derived
class object is a base class object.

## Item 33: Avoid hiding inherited names

- Names in derived classes hide names in base classes. Under public inheritance, this is never desirable.
- To make hidden names visible again, employ using declarations or forwarding functions.

Code example:
```c++
class Base {
private:
int x;
public:
virtual void mf1() = 0;
virtual void mf1(int);
virtual void mf2();
void mf3();
void mf3(double);
...
};

class Derived: public Base {
public:
virtual void mf1();
void mf3();
void mf4();
...
};
```

According to the scope-based name hiding rule, so all functions named mf1 and mf3 in the base class are hidden by the functions named mf1 and mf3 in the derived class.
```c++
Derived d;
int x;
...
d.mf1(); // fine, calls Derived::mf1
d.mf1(x); // error! Derived::mf1 hides Base::mf1
d.mf2(); // fine, calls Base::mf2
d.mf3(); // fine, calls Derived::mf3
d.mf3(x); // error! Derived::mf3 hides Base::mf3
```

## Item 34: Differentiate between inheritance of interface and inheritance of implementation

**Member function interfaces are always inherited**

Example:

```c++
class Shape {
public:
	virtual void draw() const = 0;
	virtual void error(const std::string& msg);
	int objectID() const;
	...
};

class Rectangle: public Shape { ... };
class Ellipse: public Shape { ... };
```

### Three cases:

- to inherit only the interface (declaration) of a member function
	
    **The purpose of declaring a pure virtual function is to have derived classes inherit a function interface only.**

	Incidentally, it is possible to provide a definition for a pure virtual function. That is, you could provide an implementation for Shape::draw, and C++ wouldn’t complain, but the only way to call it would be to qualify the call with the class name:
    ```c++
    Shape *ps = new Shape; // error! Shape is abstract
	Shape *ps1 = new Rectangle; // fine
	ps1->draw(); // calls Rectangle::draw
	Shape *ps2 = new Ellipse; // fine
	ps2->draw(); // calls Ellipse::draw
	ps1->Shape::draw(); // calls Shape::draw
	ps2->Shape::draw(); // calls Shape::draw
    ```

- to inherit both a function’s interface and implementation, but allow them to override the implementation they inherit

	**The purpose of declaring a simple virtual function is to have derived classes inherit a function interface as well as a default implementation.**
    
- to inherit a function’s interface and implementation without allowing them to override anything
    
    **The purpose of declaring a non-virtual function is to have derived classes inherit a function interface as well as a mandatory implementation.**

```c++
class Base
{
public:
    int b;
    void Display()
    {
        cout<<"Base: Non-virtual display."<<endl;
    };
    virtual void vDisplay()
    {
        cout<<"Base: Virtual display."<<endl;
    };
};

class Derived : public Base
{
public:
    int d;
    void Display()
    {
        cout<<"Derived: Non-virtual display."<<endl;
    };
    virtual void vDisplay()
    {
        cout<<"Derived: Virtual display."<<endl;
    };
};

Base ba;
Derived de;

Base *b = &ba;
b->Display();
b->vDisplay();
b = &de;
b->Display();
b->vDisplay();
```

Output:
```
Base: Non-virtual display.
Base: Virtual display.
Base: Non-virtual display. // Non-virtual function doesn't have dynamic binding, so based on Base pointer, it's calling Base's display.
Derived: Virtual display.
```
### Two Mistakes:
- To declare all functions non-virtual. That leaves no room for specialization in derived classes; non-virtual destructors are
particularly problematic

- To declare all member functions virtual. Sometimes this is the right thing to do, but some functions should not be redefinable
in derived classes, and whenever that’s the case, you’ve got to say so by making those functions non-virtual.


## Item 35: Consider alternatives to virtual functions

Orignal plan:
```c++
class GameCharacter {
public:
	virtual int healthValue() const; // return character’s health rating; derived classes may redefine this
};
```

### The Template Method Pattern via the Non-Virtual Interface Idiom

Here's a better design that would retain healthValue as a public member function but make it non-virtual and have it call a
private virtual function to do the real work

	
   ```c++
   class GameCharacter {
   public:
		int healthValue() const{ // derived classes do not redefine this		 
			... // do “before” stuff — see below
			int retVal = doHealthValue(); // do the real work
			... // do “after” stuff — see below
			return retVal;
        }
   ...
   private:
   		virtual int doHealthValue() const {// derived classes may redefine this
   		... // default algorithm for calculating
   		} // character’s health
   };
   ```
**Having clients call private virtual functions indirectly through public non-virtual member functions — is known as the non-virtual interface (NVI) idiom.**

The NVI idiom allows derived classes to redefine a virtual function, thus giving them control over **how** functionality is implemented, but the base class reserves for itself the right to say **when** the function will be called. **It's ok to redefine a private virtual function in derived class.**

### The Strategy Pattern via Function Pointers

```c++
class GameCharacter; // forward declaration

// function for the default health calculation algorithm
int defaultHealthCalc(const GameCharacter& gc);

class GameCharacter {
public:
	typedef int (*HealthCalcFunc)(const GameCharacter&); 

	explicit GameCharacter(HealthCalcFunc hcf = defaultHealthCalc) 	: healthFunc(hcf )	{}

	int healthValue() const { return healthFunc(*this); }
	...
private:
	HealthCalcFunc healthFunc;
};
```

- Different instances of the same character type can have different health calculation functions.
	```c++
	class EvilBadGuy: public GameCharacter {
	public:
	explicit EvilBadGuy(HealthCalcFunc hcf = defaultHealthCalc)
	: GameCharacter(hcf )
	{ ... }
	...
	};
	int loseHealthQuickly(const GameCharacter&); // health calculation
	int loseHealthSlowly(const GameCharacter&); // funcs with different behavior
	EvilBadGuy ebg1(loseHealthQuickly); // same-type characters with different health-related behavior
	EvilBadGuy ebg2(loseHealthSlowly); 
    ```
- Health calculation functions for a particular character may be changed at runtime. For example, GameCharacter might offer a member function, setHealthCalculator, that allowed replacement of the current health calculation function.


- Replace virtual functions with std::function data members, thus allowing use of any callable entity with a signature compatible with what you need. This, too, is a form of the Strategy design pattern.


## Item 36: Never redefine an inherited non-virtual function

```c++
class B {
public:
void mf();
...
};

class D: public B {
public:
void mf(); // hides B::mf; see Item33
...
};

D x; // x is an object of type D
B *pB = &x;
D *pD = &x;
pB->mf(); // calls B::mf
pD->mf(); // calls D::mf
```

**Non-virtual functions like B::mf and D::mf are statically bound. Virtual functions, on the other hand, are dynamically bound.**

If D redefines mf, there is a contradiction in your design. 
- If D really needs to implement mf differently from B, and if every B object really has to use the B implementation for mf, then it’s simply not true that every D is-a B. In that case, D shouldn’t publicly inherit from B. 
- On the other hand, if D really has to publicly inherit from B, and if D really needs to implement mf differently from B, then it’s just not true that mf reflects an invariant over specialization for B. In that case, mf should be virtual. 

## Item 37: Never redefine a function’s inherited default parameter value

Never redefine an inherited default parameter value, because default parameter values are statically bound, while virtual functions — the only functions you should be redefining — are dynamically bound.


## Item 38: Model “has-a” or “is-implemented-in-terms-of” through composition 

Composition is the relationship between types that arises when objects of one type contain objects of another type. For example:

```c++
class Address { ... }; // where someone lives

class PhoneNumber { ... };

class Person {
public:
...
private:
std::string name; // composed object
Address address; // ditto
PhoneNumber voiceNumber; // ditto
PhoneNumber faxNumber; // ditto
};
```
In this example, Person objects are composed of string, Address, and PhoneNumber objects. Among programmers, the term composition has lots of synonyms. It’s also known as layering, containment, aggregation, and embedding.

- Composition has meanings completely different from that of public inheritance.
- In the application domain, composition means has-a. In the implementation domain, it means is-implemented-in-terms-of.

## Item 39: Use private inheritance judiciously

**Private inheritance means is-implemented-in-terms-of.** If you make a class D privately inherit from a class B, you do so because you are interested in taking advantage of some of the features available in class B, not because there is any conceptual relationship between objects of types B and D.


Suppose we’re working on an application involving Widgets, and we decide we need to better understand how Widgets are being used. For example, not only do we want to know things like how often Widget member functions are called, we also want to know how the call ratios change over time. 

And we find an existing class:
```c++
class Timer {
public:
	explicit Timer(int tickFrequency);
	virtual void onTick() const; // automatically called for each tick
	...
};
```
A Timer object can be configured to tick with whatever frequency we need, and on each tick, it calls a virtual function. We can redefine that virtual function so that it examines the current state of the Widget world.

In order for Widget to redefine a virtual function in Timer, Widget must inherit from Timer. But public inheritance is inappropriate in this case. It’s not true that a Widget is-a Timer.

```c++
class Widget: private Timer {
private:
	virtual void onTick() const; // look at Widget usage data, etc.
	...
};
```

By virtue of private inheritance, Timer’s public onTick function becomes private in Widget, and we keep it there when we redeclare it. Again, putting onTick in the public interface would mislead clients into thinking they could call it.

## Item 40: Use multiple inheritance judiciously
- Multiple inheritance is more complex than single inheritance. It can lead to new ambiguity issues and to the need for virtual inheritance.
- Virtual inheritance imposes costs in size, speed, and complexity of initialization and assignment. It’s most practical when virtual base classes have no data. To see details, check [virtual inheritance](https://grant6899.github.io/2017/12/17/Virtual-Inheritance/).
- Multiple inheritance does have legitimate uses. One scenario involves combining public inheritance from an Interface class with private inheritance from a class that helps with implementation.

