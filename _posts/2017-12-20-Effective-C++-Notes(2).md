---
layout:		post
title:		Effective C++ Notes(2)
subtitle:
date:		2017-12-20
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

## Item 13: Use objects to manage resources

A general convention to manage resouce by objects:
- Resources are acquired and immediately turned over to resource-managing objects.
- Resource-managing objects use their destructors to ensure that resources are released.


Quick notes:
- To prevent resource leaks, use RAII objects that acquire resources in their constructors and release them in their destructors.
- Two commonly useful RAII classes are shared_ptr and auto_ptr. shared_ptr is usually the better choice, because its behavior when
copied is intuitive. **Copying an auto_ptr sets it to null, because it will transferthe object's possession.**

## Item 14: Think carefully about copying behavior in resource-managing classes

When implementing copying functions(copy constructor, assignment operator), consider the following possible implementations:

### 1. Prohibit copying

In many cases, it makes no sense to allow RAII objects to be copied. Disallow them by following Item 6.
	
    
### 2. Reference-count the underlying resource

Sometimes it’s desirable to hold on to a resource until the last object using it has been destroyed. copying an RAII object should increment the count of the number of objects referring to the resource. This is the meaning of “copy” used by shared_ptr.

### 3. Copy the underlying resource

Copying the resource-managing object should also copy the resource it wraps, new memory will be allocated and new objects will be constructed on that memory. It's also called deep copy.

### 4. Transfer ownership of the underlying resource

On rare occasion, you may wish to make sure that only one RAII object refers to a raw resource and that when the RAII object is copied, ownership of the resource is transferred from the copied object to the copying object. The most common example is auto_ptr.

## Item 15: Provide access to raw resources in resourcemanaging classes

- APIs often require access to raw resources, so each RAII class should offer a way to get at the resource it manages.

- Both shared_ptr and auto_ptr offer a get member function to perform an explicit conversion, i.e., to return (a copy of) the raw pointer inside the smart pointer object.
	
    ```c++
    // API
    int daysHeld(const Investment *pi);
    // calling
    int days = daysHeld(pInv.get());
    ```
- RAII classes don’t exist to encapsulate something; they exist to ensure that a particular action — resource release — takes place. 

## Item 16: Use the same form in corresponding uses of new and delete

If you use [] in a new expression, you must use [] in the corresponding delete expression. If you don’t use [] in a new expression, you mustn’t use [] in the corresponding delete expression. Otherwise it will cause undefined behavior.

## Item 17: Store newed objects in smart pointers in standalone statements

- Uncertain execution sequence:
	```c++
	processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
	```
    A possible sequence that cause memory leak if exception is thrown in priority():
    1. Execute “new Widget”.
	2. Call priority.
	3. Call the tr1::shared_ptr constructor.


- Certain execution sequence:
	```c++
	std::shared_ptr<Widget> pw(new Widget); // store newed object in a smart pointer in a standalone statement
	processWidget(pw, priority()); // this call won’t leak
	```
    1. Execute “new Widget”.
	2. Call the tr1::shared_ptr constructor.
	3. Call priority.
	
## Item 18: Make interfaces easy to use correctly and hard to use incorrectly

- Good interfaces are easy to use correctly and hard to use incorrectly. You should strive for these characteristics in all your interfaces.
- Ways to facilitate correct use include consistency in interfaces and behavioral compatibility with built-in types.
- Ways to prevent errors include creating new types, restricting operations on types, constraining object values, and eliminating client resource management responsibilities.
- shared_ptr supports custom deleters. This prevents the cross-DLL problem, can be used to automatically unlock mutexes.

## Item 19: Treat class design as type design

Questions to be asked to yourself before class design:

- How should objects of your new type be created and destroyed?
- How should object initialization differ from object assignment?
- What does it mean for objects of your new type to be passed by value?
- What are the restrictions on legal values for your new type?
- Does your new type fit into an inheritance graph? 
- What kind of type conversions are allowed for your new type?
- What operators and functions make sense for the new type?
- What standard functions should be disallowed? 
- Who should have access to the members of your new type?
- What is the “undeclared interface” of your new type? 
- How general is your new type? 
- Is a new type really what you need?

## Item 20: Prefer pass-by-reference-to-const to pass-byvalue

- Prefer pass-by-reference-to-const over pass-by-value. It’s typically more efficient and it avoids the slicing problem.

	Slicing problem:
    ```c++
    class Window {
	public:
		...
		std::string name() const; // return name of window
		virtual void display() const; // draw window and contents
	};
	
    class WindowWithScrollBars: public Window {
	public:
		...
		virtual void display() const;
	};
    
    void printNameAndDisplay(Window w) // incorrect! parameter
	{ // may be sliced!
		std::cout << w.name();
		w.display();
	}
    
    WindowWithScrollBars wwsb;
	printNameAndDisplay(wwsb);
    ```
    The parameter w will be constructed — it’s passed by value — as a Window object, and all the specialized information that
made wwsb act like a WindowWithScrollBars object will be sliced off.

	Inside printNameAndDisplay, w will always act like an object of class Window (because it is an object of class Window), regardless of the type of object passed to the function. In particular, the call to display inside printNameAndDisplay will always call Window::display, never Window-WithScrollBars::display.
	The way around the slicing problem is to pass w by reference-to-const: 
	```c++
	void printNameAndDisplay(const Window& w) // fine, parameter won’t
	{ // be sliced
		std::cout << w.name();
		w.display();
	}
    ```
	Now w will act like whatever kind of window is actually passed in.  

- The rule doesn’t apply to built-in types and STL iterator and function object types. For them, pass-by-value is usually appropriate.

## Item 21: Don’t try to return a reference when you must return an object

Never return a pointer or reference to a local stack object, a reference to a heap-allocated object, or a pointer or reference to a local static object if there is a chance that more than one such object will be needed. (Item 4 provides an example of a design where returning a reference to a local static is reasonable, at least in single-threaded environments.)

## Item 22: Declare data members private

- Declare data members private. It gives clients syntactically uniform access to data, affords fine-grained access control, allows invariants to be enforced, and offers class authors implementation flexibility.
- protected is no more encapsulated than public.

## Item 23: Prefer non-member non-friend functions to member functions
Example:

```c++
class WebBrowser {
public:
	...
	void clearCache();
	void clearHistory();
	void removeCookies();
	...
};

// member function version
class WebBrowser {
public:
	...
	void clearEverything(); // calls clearCache, clearHistory,
	// and removeCookies
	...
};

// non-member function version
void clearBrowser(WebBrowser& wb)
{
	wb.clearCache();
	wb.clearHistory();
	wb.removeCookies();
}
```

- Counterintuitively, the member function clearEverything actually yields less encapsulation than the non-member clearBrowser. 
- Offering the non-member function allows for greater packaging flexibility for functionality, and that, in turn, yields fewer compilation dependencies and an increase in class extensibility.

## Item 24: Declare non-member functions when type conversions should apply to all parameters

- Member function cannot be used for type conversion case flexiably, you have to always have user-defined class as first argument in binary operator overloading:
	```c++
	// member function version
	class Rational {
	public:
	...
	const Rational operator*(const Rational& rhs) const;
	...
	};
	```
	This will not work for:
	```c++
	Rational oneHalf(1, 2);
	result = 2 * oneHalf; // 2 doesn't have an operator * overloading accepting Rational class.
	```

- By using non-member function, it can work with implicit conversion seamlessly
	```c++
	// non-member function version with implicit conversion
	class Rational {
	public:
	...
	Rational(double rhs); // do not declare it as explicit
	...
	};

	const Rational operator*(const Rational& lhs, const Rational& rhs){
		return Rational(lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
	}
	```

