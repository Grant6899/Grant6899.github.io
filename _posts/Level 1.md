# Level 1 Notes  

标签（空格分隔）： C++ Adv_C++_Notes

---
## Lambda Function

Replacement for global functions or function objects(functors), can generate unnamed function objects.

- General syntax:
```
[capture variables] (input arguments) <mutable> -> return type
{
function implementation
}
```

- Return type deduction
When lambda is only **one** return expression, return type deduced automatically.

- Capture-default

|Syntax     |         Description |
|:--------: |:------------------- |
|[=var]|passes var by value|
|[&var]|passes var by reference|
|[=]| passes all variables by value|
|[&]| passes all variables by reference|
|[=, &var]|passes all variables by value except var which is passed by reference|

- Keyword **mutable**

Variables captured by value, can't be changed because the lambda function is const in the generated function object. To make the lambda function non-const, define it as mutable. The original variable stays unchanged, but within the function body, the captured variable can be modified.

- Capture-default in class
|Syntax     |         Description |
|:--------: |:------------------- |
|[this]| captures only the this pointer|
|[=]|captures this pointer + local variable by value|
|[&]|captures this pointer + local variable by reference|

Inside lambda function access member by:
this->member or member.

Sample code:
```c++
// Class to calculate the sine value of a vector with input values.
class CalculateSine
{
private: double m_amplitude; public:
CalculateSine(double amplitude): m_amplitude(amplitude) {}
// Calculate sine.
vector<double> Calculate(vector<double>& input) {
vector<double> result(input.size()); // Create result vector.
// Transform the input vector using a lambda function.
// The lambda function must capture the m_amplitude member variable.
// [m_amplitude] or [this->m_amplitude] does not work.
// Use [this] instead. Using [=] or [&] will also capture the this pointer. // Now you can use the members implicitly(m_amplitude)
// or explicitly (this->m_amplitude).
transform(input.begin(), input.end(), result.begin(),
[this](double v) -> double
{ return m_amplitude*sin(6.28319*v/360.0); } );
return result; // Return the result. }
};
```
Remark: **Unsolved questions: Understnding of iterators(such as output iterator)**

---

## New C++11 Language Features

- nullptr
NULL macro:
```c++
NULL
```
A real null pointer:
```c++
nullptr
```
- Static Assert

```c++
static_assert(bool_constexpr, message);
```
A static assert declaration may appear at namespace and block scope (as a block declaration) and inside a class body (as a member declaration)
If bool_constexpr returns true, this declaration has no effect. Otherwise a compile-time error is issued, and the text of message, if any, is included in the diagnostic message.

In the following example, the static_assert declaration has namespace scope. Because the compiler knows the size of type void *, the expression is evaluated immediately. This can save compilation time when failed.

```c++
static_assert(sizeof(void *) == 4, "64-bit code generation is not supported.");  
```

- decltype
 determine type of expression at compile time
 
Example:

```c++
int i =20;
double d=3.16;
decltype(i) j = i; //j is an int.
decltype(j*d) l = j* d; //Result of int * double is double.

template<typename T1, typename T2>
decltype(v1 * v2) operateor() (const T1& v1, const T2& v2) {return v1*v2;}

```
It can be a replacement of auto, but auto is more convenient.


- lvalue vs. rvalue and varaible binding

lvalue names objects that persist beyond the single expression:
```c++
++x;
```
rvalue is temporary taht disappers beyond the full expression:
```c++
x++;
```

**Rvalues and lvalues can both be cound to const references, only lvalues can be bound to non-const references.**

C++11 introduces new rvalue reference(&&).
**Only rvalues can be bound to rvalue references and they are eligible for move operations (lvalues cannot be moved).**

Example:
```c++
//Return mutable rvalue.
double MutableRValue(){
return 3.14;
}

//Return const rvalue.
const double ConstRValue(){
return 2.72;
}

//Create a mutable lvalue.
double mutalbeLValue = 3.14;

//Create a const lvalue.
const double constLValue = 2.72

//Bind to reference.
double& a1 = mutableLValue; //OK
double& a2 = constLValue; //Error. const can't bind to non-const reference.
double& a3 = MutableRValue(); //Error. rvalues can't bind to non-const reference.
double& a4 = ConstRValue(); //Error. rvalues can't bind to non-const reference.

//Bind to const reference.
const double& a1 = mutableLValue; //OK
const double& a2 = constLValue; //OK
const double& a3 = MutableRValue(); //OK
const double& a4 = ConstRValue(); //OK

//Bind to rvalue reference.
double&& c1 = mutableLValue; //Error. lvalue can't bind to rvalue reference.
double&& c2 = constLValue; //Error. lvalue can't bind to rvalue reference.
double&& c3 = MutableRValue(); //OK
double&& c4 = ConstRValue(); //Error. const can't bind to non-const reference.

//Bind to const rvalue reference.
const double&& d1 = mutableLValue; //Error. lvalue can't bind to rvalue ref.
const double&& d2 = constLValue; //Error. lvalue can't bind to rvalue ref.
const double&& d3 = MutableRValue(); //OK
const double&& d4 = ConstRValue(); //OK
```
Application: create 'move' constructor & assignment operator that moves the state of a temporary.

- Move semantics
 It would be much more efficient if we could move the state of a temporary to the resulting object.
 C++11 rvalue references makes this possible by:
 - Create move constructor with rvalue reference
 - Create assignment operator with rvalue reference
 Then the dynamic state is moved instead of copied.
 
- **constexpr** Specifier
example:
```c++
#define MEANING_OF_LIFE 42
const int MeaningOfLife = 42;
constexpr int MeaningOfLife () {return 42;}

class C{
 private:
   int v;
 public:
   constexpr C(int d) : v(d) {} 
};
```
If Arugument d is known at compile-time then data members of the constructed C instance are also known during compilation.

- Keyword **noexcept**
 used to specify that a function cannot throw:
```c++
void func() noexcept;
```
If such a function throws then the program will be terminated(std::terminate() and then std::abort())

- Uniform Initialization
Everything intializes with "{}".
 - Initializer Lists
An instance of std::initializer_list<T> is a lightweight proxy object that provides access to an array of objects of type T.

Initialization priority:
initializer_list > regular constructor > aggregate initialization

- Variadic Templates

Recursive expanding args:
```c++
void print(){
	cout << "empty" << endl;
}

template <class T, class... Args>
void print(T head, Args... rest){
	cout << "parameter " << head << endl;
	print(rest...);
}


int main(void){
	print(1,2,3,4);
	return 0;
}
```
Output:
```c++
parameter 1
parameter 2
parameter 3
parameter 4
empty
```
- Alias Template

 Example:
```c++
#include<vector>
template<class T> 
struct Alloc { /* ... */ };

template<class T>
using Vec = std::vector<T, Alloc<T>>;
 
Vec<int> v; // equal to std::vector<int, Alloc<int>> v;
```

--- 
## Advanced language features

- STL Bind
Currying: Technique to translate the evaluation of a function that takes multiple arguments into a sequence of functions, each with a single argument.

Function programming idea.
Example:
```c++
template <typename T>
std::function<T (const T&)> bind1st(
  const std::function<T (const T& x, const T& y)>& f, const T& x_)
  {
    return [=] (const T& y){
    return f(x_, y);
    }
  }
```
---

## Data Structures Introduction

- UnorderedCollections
- Union
- Template Template Parameters
 - General Syntax:

 ```c++
 template<other params,...,template<TemplateTypeParams> class ParameterName,other params,...>
 ```

 - Example:
```c++
template<typename T,template<typename E,typename Allocator=allocator<E> >class Container=vector>  //Default container is vector  
class Grid  
{  
public:  
 //Omitted for brevity  
 Container<T>* mCells;  
}; 

template<typename T,template<typename E,typename Allocator=allocator<E> >class Container>  

Grid<T,Container>::Grid(int inWidth,int inHeight):  
mWidth(inWidth),mHeight(inHeight)  
{  
mCells=new Container<T> [mWidth];   
for(int i=0;i<mWidth;++i)  
  mCells[i].resize(mHeight);  
}  
```

- High order function
 - It takes one or more functions as input and outputs a function
 - All other functions are *first-order* functions
 - Examples:
  the derivatives of a function; integration.
  Vector space of functions
  function composition
  maps

---

## Tuple

- Tuples are useful for 'aggregating' related data
- They resolve many issues with multiple return types and multiple input arguments, used as input arguments and return types

Example:
```c++
#include <iostream>  
#include <vector>  
#include <string>  
#include <tuple>  
  
using namespace std;  
  
std::tuple<std::string, int>  
giveName(void)  
{  
    std::string cw("Caroline");  
    int a(2013);  
    std::tuple<std::string, int> t = std::make_tuple(cw, a);  
    return t;  
}  
  
int main()  
{  
    std::tuple<int, double, std::string> t(64, 128.0, "Caroline");  
    std::tuple<std::string, std::string, int> t2 =  
            std::make_tuple("Caroline", "Wendy", 1992);  
  
    //return number of elements  
    size_t num = std::tuple_size<decltype(t)>::value;  
    std::cout << "num = " << num << std::endl;  
  
    //get the first element's type
    std::tuple_element<1, decltype(t)>::type cnt = std::get<1>(t);  
    std::cout << "cnt = " << cnt << std::endl;  
  
    //comparison
    std::tuple<int, int> ti(24, 48);  
    std::tuple<double, double> td(28.0, 56.0);  
    bool b = (ti < td);  
    std::cout << "b = " << b << std::endl;  
  
    //tuple as returned value     
    auto a = giveName();  
    std::cout << "name: " << get<0>(a)  
            << " years: " << get<1>(a) << std::endl;  
  
    return 0;  
}  
```
Output:
```c++
num = 3  
cnt = 128  
b = 1  
name: Caroline years: 2013  
```

## Some Other Features in C++11

- Alias Templates
 **using** keyword
 Example:
```c++
#include <vector>
 
template<class T> struct Alloc { /* ... */ };
template<class T>
using Vec = std::vector<T, Alloc<T>>;
 
Vec<int> v;
```
- Intermezzo : Range-based loops
 - New kind of *for loop* in C++11
 Example:
```c++
 for (auto& elem : vec){ elem +=2; }
```  

- Functionality relating to Classes
 - **final** Specifier
  - Applies to both classes and virtual functions declarations and definitions
  - Indicates that the class cannot be derived from or that the the function cannot be overridden by derived classes
```c++
class SealedClass final{
//Derivation not possible
};

/* 
class DerivedSeal : public SealedClass{
//will not compile
};
*/

std::cout << std::is_final<SealedClass>::value << "\n";
```

- Specifier explicit
 - Used in order to disallow implicit conversions or copy-initialization
 - Applies to constructors and user-difined ***conversion operator***
 
The explicit specifier specifies that a constructor or conversion function (since C++11) doesn't allow implicit conversions or copy-initialization. It may only appear within the decl-specifier-seq of the declaration of such a function within its class definition.

Remark: A constructor with a single non-default parameter (until C++11) that is declared without the function specifier explicit is called a converting constructor.
Both constructors (other than copy/move) and user-defined conversion functions may be function templates; the meaning of explicit doesn't change.
```c++
struct A
{
    A(int) { }      // converting constructor
    A(int, int) { } // converting constructor (C++11)
    operator bool() const { return true; }
};
 
struct B
{
    explicit B(int) { }
    explicit B(int, int) { }
    explicit operator bool() const { return true; }
};
 
int main()
{
    A a1 = 1;      // OK: copy-initialization selects A::A(int)
    A a2(2);       // OK: direct-initialization selects A::A(int)
    A a3 {4, 5};   // OK: direct-list-initialization selects A::A(int, int)
    A a4 = {4, 5}; // OK: copy-list-initialization selects A::A(int, int)
    A a5 = (A)1;   // OK: explicit cast performs static_cast
    if (a1) ;      // OK: A::operator bool()
    bool na1 = a1; // OK: copy-initialization selects A::operator bool()
    bool na2 = static_cast<bool>(a1); // OK: static_cast performs direct-initialization
 
//  B b1 = 1;      // error: copy-initialization does not consider B::B(int)
    B b2(2);       // OK: direct-initialization selects B::B(int)
    B b3 {4, 5};   // OK: direct-list-initialization selects B::B(int, int)
//  B b4 = {4, 5}; // error: copy-list-initialization does not consider B::B(int,int)
    B b5 = (B)1;   // OK: explicit cast performs static_cast
    if (b2) ;      // OK: B::operator bool()
//  bool nb1 = b2; // error: copy-initialization does not consider B::operator bool()
    bool nb2 = static_cast<bool>(b2); // OK: static_cast performs direct-initialization
}
```

- **override** Specifier
  - Refers to overriding a member function in a base class by the same function and signature in a derived class
  - Its use ensures that the function is cirtual and that it is overriding a virtual function from the base class
  - functions' constness and reference qualifiers must be identical
  - Example:
```c++
struct Base{
  virtual void draw(){}
  void print() {}
};

struct Derived : public Base{
  void draw() override{};
  // void draw() const override{} // No compile
  // void print() override{} // No compile
};
```
- Explicitly defaulted and deleted Functions
 - In C++11 we can control whetehr the special member functions are automatically generated
 - Deleted functions prevent problematic type promotions from occurring in arguments to functions of all types

- Special Member Function Generation
 - (defult ctor, destructor, copy ctor, copy assignment operator) are generated only if needed
 - In C++11, we now have in addition move ctor and move assignment operator to add to the list
 - Move operations are generated when three situations occur: 1) No copy operations are declared in the class 2) No move operations are declared in the class 3) No destructor is declared in the class
 - Example:
```c++
class F{
  public:
    F() = default;
    F(F&&) = default;
    F& operator = (F&&) = default;
    F(const F&) = default;
    //F& operator = (const F&) = default;
    F& operator = (const F&) = delete;
    virtual ~F = default;
};
```
  
   
   
 
   


 
 