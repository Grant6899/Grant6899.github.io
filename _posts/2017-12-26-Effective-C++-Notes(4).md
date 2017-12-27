---
layout:		post
title:		Effective C++ Notes(4)
subtitle:
date:		2017-12-26
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Interview
---

## Item 41: Understand implicit interfaces and compile time polymorphism


- The world of object-oriented programming revolves around explicit interfaces and runtime polymorphism.

	An explicit interface typically consists of **function signatures**, i.e., function names, parameter types, return types, etc. Polymorphism occurs at runtime through **virtual functions**.
    
	```c++
	class Widget {
	public:
		Widget();
		virtual ~Widget();
		virtual std::size_t size() const;
		virtual void normalize();
		void swap(Widget& other);
		...
	};
	```
 
- The world of templates and generic programming is fundamentally different, implicit interfaces and compile-time polymorphism move to the fore.
	```c++
    template<typename T>
	void doProcessing(T& w){
		if (w.size() > 10 && w != someNastyWidget) {
		T temp(w);
		temp.normalize();
		temp.swap(w);
		}
	}
    ```
	Interfaces are implicit and based on **valid expressions**.
	The interface that w must support is determined by the operations performed on w in the template. In this example, it appears that w’s type (T) must support:
	- the size, normalize, and swap member functions; 
	- copy construction (to create temp); 
	- comparison for inequality (for comparison with someNastyWidget
	- the set of expressions that must be valid in order for the template to compile is the implicit interface that T must support.
    
    Polymorphism occurs during compilation through template instantiation and function overloading resolution.
    - The calls to functions involving w such as operator> and operator!= may involve instantiating templates to make these calls succeed. Such instantiation occurs during compilation. Because instantiating function templates with different template parameters leads to different functions being called, this is known as **compile-time polymorphism**.

## Item 42: Understand the two meanings of typename

- To declare a template type parameter
	
    "typename" and "class" means the same thing in this case.
	```c++
	template<class T> class Widget; // uses “class”
	template<typename T> class Widget; // uses “typename”
	```

- to identify **only** nested dependent type names
	```c++
	template<typename C> // typename allowed (as is “class”)
	void f(const C& container, // typename not allowed because C is not nested type name
		typename C::iterator iter); // typename required
	```
    
	- Exception:    
	
		- typename must not precede nested dependent type names in a list of base classes 
		- typename must not precede nested dependent type names as a base class identifier in a member initialization list

	```c++
    template<typename T>
	class Derived: public Base<T>::Nested { // base class list: typename not allowed
	public: 
		explicit Derived(int x) 
			: Base<T>::Nested(x) // base class identifier in mem. init. list: typename not allowed
		{ 
			typename Base<T>::Nested temp; // use of nested dependent type
			... // name not in a base class list or
		} // as a base class identifier in a mem. init. list: typename required
	};
    ```

	- traits example:
	```c++
    template<typename IterT>
	void workWithIterator(IterT iter) {
		typename std::iterator_traits<IterT>::value_type temp(*iter);
	...
	}
    ```

## Item 43: Know how to access names in templatized base classes

Begin this topic with example:

```c++
class CompanyA {
public:
	...
	void sendCleartext(const std::string& msg);
	void sendEncrypted(const std::string& msg);
	...
};

class CompanyB {
public:
	...
	void sendCleartext(const std::string& msg);
	void sendEncrypted(const std::string& msg);
	...
};
... // classes for other companies

class MsgInfo { ... }; // class for holding information used to create a message

template<typename Company>
class MsgSender {
public:
	... // ctors, dtor, etc.
	void sendClear(const MsgInfo& info){
		std::string msg;
		create msg from info;
		Company c;
		c.sendCleartext(msg);
	}
    
	void sendSecret(const MsgInfo& info) // similar to sendClear, except
	{ ... } // calls c.sendEncrypted
};


template<typename Company>
class LoggingMsgSender: public MsgSender<Company> {
public:
	... // ctors, dtor, etc.
	void sendClearMsg(const MsgInfo& info){
		write "before sending" info to the log;
		sendClear(info); // call base class function; this code will not compile!
		write "after sending" info to the log;
	}
	...
};
```

Three ways to make the code above compile:

- to preface calls to base class functions with “this->”:
	```c++
    template<typename Company>
	class LoggingMsgSender: public MsgSender<Company> {
	public:
	...
		void sendClearMsg(const MsgInfo& info){
			write "before sending" info to the log;
			this->sendClear(info); // okay, assumes that sendClear will be inherited
			write "after sending" info to the log;
		}
	...
	};
    ```
- to employ a using declaration:
	```c++
    template<typename Company>
	class LoggingMsgSender: public MsgSender<Company> {
	public:
		using MsgSender<Company>::sendClear; // tell compilers to assume that sendClear is in the base class
		void sendClearMsg(const MsgInfo& info){
			...
			sendClear(info); // okay, assumes that sendClear will be inherited
		}
		...
	};
    ```
- to explicitly specify that the function being called is in the base class(not desired because if the function being called is virtual, explicit qualification turns off the virtual binding behavior):
 	```c++
    template<typename Company>
	class LoggingMsgSender: public MsgSender<Company> {
	public:
		...
		void sendClearMsg(const MsgInfo& info){
			...
			MsgSender<Company>::sendClear(info); // okay, assumes that sendClear will be inherited
			...
        }
        ...
	};
 	```

### a specialized version of MsgSender for CompanyZ:
```c++
template<> // a total specialization of MsgSender; the same as the general template, except sendClear is omitted
class MsgSender<CompanyZ> { 
public: 
	void sendSecret(const MsgInfo& info)
	{ ... }
};
```

## Item 44: Factor parameter-independent code out of templates

- Templates generate multiple classes and multiple functions, so any template code not dependent on a template parameter causes bloat.
- Bloat due to non-type template parameters can often be eliminated by replacing template parameters with function parameters or class data members.
- Bloat due to type parameters can be reduced by sharing implementations for instantiation types with identical binary representations.

## Item 45: Use member function templates to accept “all compatible types"

- Use member function templates to generate functions that accept all compatible types.
	
    If we'd like to see conversions below:
    ```c++
    template<typename T>
	class SmartPtr {
	public: // smart pointers are typically
		explicit SmartPtr(T *realPtr); // initialized by built-in pointers
		...
	};
	
    SmartPtr<Top> pt1 = // convert SmartPtr<Middle> -> SmartPtr<Top>
	SmartPtr<Middle>(new Middle); 
	SmartPtr<Top> pt2 = // convert SmartPtr<Bottom> -> SmartPtr<Top>
	SmartPtr<Bottom>(new Bottom); 
	SmartPtr<const Top> pct2 = pt1; // convert SmartPtr<Top> -> SmartPtr<const Top>
	```
    
    We need a **constructor template**:
    ```c++
    template<typename T>
	class SmartPtr {
	public:
		template<typename U>
		SmartPtr(const SmartPtr<U>& other) // initialize this held ptr with other’s held ptr
			: heldPtr(other.get()) { ... } 
		
        T* get() const { return heldPtr; }
			...
	private: // built-in pointer held
		T *heldPtr; // by the SmartPtr
	};
    ```
    
- If you declare member templates for generalized copy construction or generalized assignment, you’ll still need to declare the normal copy constructor and copy assignment operator, too.
	```c++
    template<class T> class shared_ptr {
	public:
		shared_ptr(shared_ptr const& r); // copy constructor
		
        template<class Y> // generalized copy constructor
		shared_ptr(shared_ptr<Y> const& r); 
		
        shared_ptr& operator=(shared_ptr const& r); // copy assignment
		
        template<class Y> // generalized copy assignment
		shared_ptr& operator=(shared_ptr<Y> const& r); 
		...
	};
    ```
    
## Item 46: Define non-member functions inside templates when type conversions are desired

- When writing a class template that offers functions related to the template that support implicit type conversions on all parameters, define those functions as friends inside the class template.

	```c++
    template<typename T>
	class Rational {
	public:
		Rational(const T& numerator = 0, // see Item20 for why params
		const T& denominator = 1); // are now passed by reference
		const T numerator() const; // see Item28 for why return
		const T denominator() const; // values are still passed by value,
		... // Item 3 for why they’re const
	};
	
    template<typename T>
	const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs)
	{ ... }
    ```
    Code above will not  work. 
    ```c++
    Rational<int> oneHalf(1, 2); // this example is from Item 24, except Rational is now a template
	Rational<int> result = oneHalf * 2; // error! won’t compile
    ```
    the deduction for the other parameter is not so simple. operator*’s second parameter is declared to be of type Rational<T>, but the second argument passed to operator* (2) is of type int. **Because implicit type conversion functions are never considered during template argument deduction.**
    
	Solution: Mixed-mode design
    ```c++
    template<typename T>
	class Rational {
	public:
	...
    	friend const Rational operator*(const Rational& lhs, const Rational& rhs){ // declare operator* function 
			return Rational( lhs.numerator() * rhs.numerator(), lhs.denominator() * rhs.denominator());
        }
    };
	
    template<typename T> // define operator* functions
	const Rational<T> operator*(const Rational<T>& lhs, const Rational<T>& rhs)
	{ ... }
    ```
    A declared function (not a function template), compilers can use implicit conversion functions (such as Rational’s non-explicit
constructor) when calling it.


## Item 47: Use traits classes for information about types

- Traits classes make information about types available during compilation. They’re implemented using templates and template specializations.
- In conjunction with overloading, traits classes make it possible to perform compile-time if...else tests on types.

**Refer to [STL源码剖析](https://github.com/Grant6899/books/blob/master/STL%E6%BA%90%E7%A0%81%E5%89%96%E6%9E%90.pdf) 3.6 to see the implementation of iterator and its relationship between type traits conventions.**

    





