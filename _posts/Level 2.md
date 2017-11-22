# Level 2 Notes  

标签（空格分隔）： C++ Adv_C++_Notes

---
## Type Traits
- provide a mechanism to define behavior based on types
- shared_ptr is not a pointer
- Example:
```c++
template <typename T> void IsPointer(const T& t){
if (std::is_pointer<T>::value) {
  std::cout << "This is a pointer type argument\n"; }
else {
  std::cout << "This is _not_ a pointer type argument\n";
} 
```
- For operator ?:, B and C must be the same type
```c++
A? B:C 
```
---
## Advanced Type Traits

- Pointer Traits

 - add_pointer: 
 ```c++
 template <class T> struct add_pointer;
 ```
Obtains the pointer type that points to T. The transformed type is aliased as member type add_pointer::type.
**If T is a reference type, this is remove_reference<T>::type*, otherwise it is T*.**
Example:
 ```c++
 // add_pointer
 #include <iostream>
 #include <type_traits>

 typedef std::add_pointer<int>::type A;        // int*
typedef std::add_pointer<const int>::type B;  // const int*
typedef std::add_pointer<int&>::type C;       // int*
typedef std::add_pointer<int*>::type D;       // int**
typedef std::add_pointer<int(int)>::type E;   // int(*)(int)
 ```
 - remove_pointer:
 ```c++
 template <class T> struct remove_pointer;
 ```
Obtains the type pointed by T (if T is a pointer). The transformed type is aliased as member type remove_pointer::type.
**If T is a pointer type, this is the type to which it points. Otherwise, it is the same as T, unchanged.**
Example:
 ```c++
 // remove_pointer
 #include <iostream>
 #include <type_traits>

 typedef std::remove_pointer<int>::type A;          // int
typedef std::remove_pointer<int*>::type B;         // int
typedef std::remove_pointer<int**>::type C;        // int*
typedef std::remove_pointer<const int*>::type D;   // const int
typedef std::remove_pointer<int* const>::type E;   // int
``` 

- Array traits
 - alignment_of:
 ```c++
template <class T> struct alignment_of;
 ```
Trait class to obtain the alignment requirement of type T.
**If T is a reference type, the alignment obtained is the one required by the referenced type.
If T is an array type, the alignment obtained is the one required by the element type.**

 - rank:
 ```c++
 template <class T> struct rank;
 ```
Trait class to obtain the rank of type T. The rank of an array type is its number of dimensions. For other types is zero.
Example:
 ```c++
 int main() {
   std::cout << "rank:" << std::endl;
   std::cout << "int: " << std::rank<int>::value <<    std::endl;
   std::cout << "int[]: " << std::rank<int[]>::value << std::endl;
   std::cout << "int[][10]: " << std::rank<int[][10]>::value << std::endl;
   std::cout << "int[10][10]: " <<   std::rank<int[10][10]>::value << std::endl;
  return 0;
 }
 ```
 - extent:
 Trait class to obtain the extent of the Ith dimension of type T.
**If T is an array that has a rank greater than I, the extent is the bound (i.e., the number of elements) of the Ith dimension.
In all other cases, and in the case that I is zero and T is an array of unknown bound, the extent value is zero.**
Example:
 ```c++
int main() {
  typedef int mytype[][24][60];
  std::cout << "mytype array (int[][24][60]):" << std::endl;
  std::cout << "rank: " << std::rank<mytype>::value << std::endl;
  std::cout << "extent (0th dimension): " << std::extent<mytype,0>::value << std::endl;
  std::cout << "extent (1st dimension): " << std::extent<mytype,1>::value << std::endl;
  std::cout << "extent (2nd dimension): " << std::extent<mytype,2>::value << std::endl;
  std::cout << "extent (3rd dimension): " << std::extent<mytype,3>::value << std::endl;
  return 0;
}
 ```
output:
 ```c++
 mytype array (int[][24][60]):
rank: 3
extent (0th dimension): 0
extent (1st dimension): 24
extent (2nd dimension): 60
extent (3rd dimension): 0
 ```
 - remove_extent:
 Obtains the type of the elements in T (if T is an array type).
 The transformed type is aliased as member type remove_extent::type. 
 **If T is an array type, this is the same type as its elements. Otherwise, member type is the same as T.**
Notice that, for multidimensional arrays, only the first array dimension is removed (see remove_all_extents to obtain the type of the elements in the deepest dimension).
This class merely obtains a type using another type as model, but it does not transform values or objects between those types.
Example:
 ```c++
int main() {
  typedef std::remove_extent<int>::type A;                // int
  typedef std::remove_extent<int[24]>::type B;            // int
  typedef std::remove_extent<int[24][60]>::type C;        // int[60]
  typedef std::remove_extent<int[][60]>::type D;          // int[60]
  return 0;
}
 ```

 - remove_all_extents:
 Obtains the type of the elements in the deepest dimension of T (if T is an array type). The transformed type is aliased as member type remove_all_extents::type.
**If T is an array type, this is the same type as the elements in its deepest dimension. Otherwise, member type is the same as T.**
 Example:
 ```c++
 int main() {
  typedef std::remove_all_extents<int>::type A;                // int
  typedef std::remove_all_extents<int[24]>::type B;            // int
  typedef std::remove_all_extents<int[24][60]>::type C;        // int
  typedef std::remove_all_extents<int[][60]>::type D;          // int
  typedef std::remove_all_extents<const int[10]>::type E;      // const int
  return 0;
}
 ```

## Advanced Lambda Programming
- init Capture(C++14)
 - Example
 ```c++
 // C++14 init capture, local ‘data’
auto fun2 = [data = std::move(data)] () mutable {
      for (std::size_t i = 0; i < data.size(); ++i) {
data[i] = 3.14; std::cout << data[i] << ",";} 
 std::cout << '\n';
}; 
fun2();
// The data has no elements, as can be seen
// if (data.size() == 0), YES
 ```

- Forward
 - std::forward
 ```c++
 std::forward<T>(u)
 ```
 1) Forwards lvalues as either lvalues or as rvalues, depending on T.
When t is a forwarding reference (a function argument that is an rvalue reference to a cv-unqualified function template parameter), this overload forwards the argument to another function with the value category it had when passed to the calling function.
For example, if used in wrapper such as the following, the template behaves as described below:
 ```c++
 template<class T>
void wrapper(T&& arg) 
{
    // arg is always lvalue
    foo(std::forward<T>(arg)); // Forward as lvalue or as rvalue, depending on T
}
 ```
*If a call to wrapper() passes an rvalue std::string, then T is deduced to std::string (not std::string&, const std::string&, or std::string&&), and std::forward ensures that an rvalue reference is passed to foo.
If a call to wrapper() passes a const lvalue std::string, then T is deduced to const std::string&, and std::forward ensures that a const lvalue reference is passed to foo.
If a call to wrapper() passes a non-const lvalue std::string, then T is deduced to std::string&, and std::forward ensures that a non-const lvalue reference is passed to foo.*
2) Forwards rvalues as rvalues and prohibits forwarding of rvalues as lvalues.
This overload makes it possible to forward a result of an expression (such as function call), which may be rvalue or lvalue, as the original value category of a forwarding reference argument.
For example, if a wrapper does not just forward its argument, but calls a member function on the argument, and forwards its result:
 ```c++
// transforming wrapper 
template<class T>
void wrapper(T&& arg)
{
    foo(forward<decltype(forward<T>(arg).get())>(forward<T>(arg).get()));
}
 ```
where the type of arg may be
 ```c++
struct Arg
{
    int i = 1;
    int  get() && { return i; } // call to this overload is rvalue
    int& get() &  { return i; } // call to this overload is lvalue
};
 ```
Attempting to forward an rvalue as an lvalue, such as by instantiating the form (2) with lvalue reference type T, is a compile-time error.
 - Passing rule:
T&  + &  -> T& （lvalue + &  -> lvalue）
T&& + &  -> T& （rvalue + &  -> lvalue）
T&  + && -> T& （lvalue + && -> lvalue）
T&& + && -> T&& (rvalue + && -> rvalue)

---

## Boost Memory

- Smart Pointers
 - Auto pointer(auto_ptr)
 auto_ptr is a smart pointer that manages an object obtained via new expression and deletes that object when auto_ptr itself is destroyed. It may be used to provide exception safety for dynamically-allocated objects, for passing ownership of dynamically-allocated objects into functions and for returning dynamically-allocated objects from functions
 
 - Scoped Pointer
 Similar to auto_ptr but no transer of ownership
 Cannot be copied or assigned, ownership never surrendered
  The primary reason to use scoped_ptr rather than auto_ptr is to let readers of your code know that you intend "resource acquisition is initialization" to be applied only for the current scope, and have no intent to transfer ownership.
  A secondary reason to use scoped_ptr is to prevent a later maintenance programmer from adding a function that transfers ownership by returning the auto_ptr, because the maintenance programmer saw auto_ptr, and assumed ownership could safely be transferred.
Think of bool vs int. We all know that under the covers bool is usually just an int. Indeed, some argued against including bool in the C++ standard because of that. But by coding bool rather than int, you tell your readers what your intent is. Same with scoped_ptr; by using it you are signaling intent.
It has been suggested that scoped_ptr<T> is equivalent to std::auto_ptr<T> const. Ed Brey pointed out, however, that reset will not work on a std::auto_ptr<T> const.
Example:
 ```c++
 //Create dynamic memory
 boost::scoped_ptr<Point> myPoint (new Point(1.0, 23.3));
 
 //Scoped pointer has same syntax as a raw pointer
 if (myPoint != 0){
    cout<< *myPoint;
    }
//Assign to another point
 Point yourPoint (7.3, -9,9);
 
 *myPoint = yourPoint;
 cout<<*myPoint
 ```
  - Scoped Arrays(scoped_array<T>)
  Overloaded[] to mimic raw arrays
  Better than allocating ordinary dynamically allocated arrays
  Useful when the size of the array remains constant; otherwise use vector
  
  Example:
  ```c++
  int main(){
  const int N = 4;
  Point * pointArray = new Point [N};
  
  for (int i = 0; i < N; ++i){
  cout<<pointArray[i]<<endl;
  }
  
  {
  boost::scoped_array<Point> sharedPointArray (pointArray);
  for( int i = 0; i < N; ++i){
  sharedPointArray[i] = Point(double(i), double(i));
  cout<< sharedPointArray[i] <<endl;
  }
  }
  return 0;
  }
  ```
  
 - Shared Pointer(shared_ptr)
 Automatic lifetime control of dynamic objects
 Reference is incremented each time object is referenced
 When Reference Count == 0, object is automatically deleted
 Non-intrusive
  
 - Shared Array(shared_array)
 The object pointed to is guaranteed to be deleted when the last shared_array pointing to it is deleted
 Plan B: define a shared pointer to an STL vector
 
 - Weak Pointer(weak_ptr)
 weak_ptr is used to solve the "dead lock" problem that 
 share pointers referencing each other, where the use_count of these two pointers will never be zero, then resource is never released. 
 It's a weak reference of an object and will not increase the count. It can be converted to shared_ptr: shared_ptr can be directly assigned to it and we can also get the corresponding shared_ptr by calling .lock().
 
 - [Intrusive Pointer(intrusive_ptr)](https://my.oschina.net/cppblog/blog/9260)
 
 ---
 
## IEEE 754 - Techinical standard for floating point computation

- quiet NaN vs signalling NaN
 - Operations involving qNaNs do not result in exceptions; processing continues
 - sNaN are special NaNs; should throw an invalid operation exception when consumed by most operations
 
- Classifying Numbers
 - Zero
 - Subnormal/denormal (smaller than the smallest representable number for the given type)
 - Infinite
 - NaN
 - Normal (one that is not one of the other four)
 
 std::fpclassify() example:
 ```c++
 const char* show_classification(double x) {
    switch(std::fpclassify(x)) {
        case FP_INFINITE:  return "Inf";
        case FP_NAN:       return "NaN";
        case FP_NORMAL:    return "normal";
        case FP_SUBNORMAL: return "subnormal";
        case FP_ZERO:      return "zero";
        default:           return "unknown";
    }
} 
 ```
- Querying Numbers 
- std::numeric_limitt&lt;T&gt;
 - Provides information about the properties of arithmetic types
 - Class template is specialised for each fundamental arithmetic type, should not be specialised for other types
 - Minimum and maximum finite value min() and max()
 - lowest(); same as min() for integral types; for floating point types it is the negative of max()
 - Maximum rounding error round_error()
 - epsilon() all machine epsilon(difference between 1 and the least value > 1 that is representable)
 - round_style() based on IEEE754 rounding style
- STL Errors
 - Logic errors)in theory can be avoided, e.g. by testing arguments)
 - Runtime errors (events that are beyond the scope of the program cannot be avoided)
 - 'bad' errors: cast, typeid, (unexpected) exception, alloc, weak_ptr, function_call
- Logic errors:
 - Domain error: situations where the input is outside the domain on which an operation is defined (Standard library doesn't use it but 3rd parties do)
 - Invalid_argument: an argument value to a function has not been accepted(Can be thrown by std::bitset)
 - Length error: event occurs when we attempt to exceed implementation-defined length limits for some object (thrown by std::basic_string, std::vector::reverse)
 - Out of range: event occurs if we attempt to access elements that are outside a defined range(thrown by std::vector::at and std::map::at)
 - Future error: thrown on failure by the function in the thread liabrary
- Runtime errors
 - system_error: can be thrown by various library functions, typically functions that interface to OS (e.g.boost::asio)
 - range_error: used to report errors concerning the results of a computation(the mathematical functions in the standard library do not throw this exception)
 - overflow_error: result of a computation that is too large for the destination type
 - underflow-error: reports arithmetic underflow errors(that is when the result of a computation is a subnormal floating-point number)(Standard library functions do not throw this exception but third party libraryies do)
 - bad_function_call: thrown by std::function::operator() if a function wrapper has no target method
 
---
##C++ System Errors
- Error classes are eventually inherited from std::exception
- Classes(in std namespace)
 - system_error: type of the exception thrown by various library functions, e.g. OS interfaces
 Example:
 ```c++
 #include<system_error>
 int main(){
  try{
    std::thread().detach(); // attemt to detach a non-thread
    }   
    catch (const std::system_error& e){
    std::cout<<"Caught system_error with code "<< e.code() <<"meaning "<<e.what()<<'\n';
    }
    
    return 0;
    }
 ```
 - error_category: base class for system category types. Instances are singletons
 - error_condition:platform-independent error code. Uniquely identified by an integer and an error_category
 - error_code: platform-dependent error code. Each error code holds a pair [OS error code, pointer to error_category]
 - future_error: exception object that is thrown on failure by functions in the thread library
 
- Specific Error Categories
 - generic_category: identifies the generic error category
 - system_category: identifies the operating system error category
 - iostream_category: identifies the iostream error category
 - future_category: identifies the future error category 
 
- std::errc
 - A scoped enumeration that defines the values of portable error conditions
 - These conditions corresponds to POSIX error codes

---
##STL Bitset and Boost Dynamic Bitset
- Wish to model fixed-size and dynamic arrays of bits(Boolean values)
- Useful way to manage sets of flags and corresponding bitwise operations
- Can specify at compile-time (STL) or run-time (Boost) 

 
 