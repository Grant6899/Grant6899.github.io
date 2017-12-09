---
layout:		post
title:		Effective STL Notes
subtitle:
date:		2017-11-21
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - STL
    - Functional programming
---

What written here is a brief summary for what I've learnd from [Effective STL](https://www.amazon.com/Effective-STL-Specific-Standard-Template/dp/0201749629) by Scott Meyers. The purpose of doing this is to remind myself every now and then in case these tricks get rusty.


## Item 1: Choose your containers with care

Here's just an overview of how the author categorizes well-known STL containers:

* **standard** STL **sequence** containers: vector, string, deque, list
* **standard** STL **associative** containers: set, multiset, map, multimap
* **nonstandard** **sequence** containers: slist, rope
* **nonstandard associative** containers: hash_set, hash_multiset, hash_map, hash_multimap
* **contiguous-memory** containers (array-based containers): vector, string, deque, rope
* **node-based** containers: list, slist and all standard associative containers


## Item 2: Beware the illusion of container-independent code

only sequence containers support push_front or push_back

only associative containers support count and lower_bound

```c++
class Widget {
private:
	int _data;
public:
	Widget(int x) : _data(x) {}
	bool operator==(const Widget& rhs) { return this->data() == rhs.data(); }
	int data() const { return _data; }
};

typedef vector<Widget> WidgetContainer;
typedef WidgetContainer::iterator WCIterator;
		
WidgetContainer vw{ Widget(1), Widget(2), Widget(3) };

Widget bestWidget(2);

WCIterator i = find(vw.begin(), vw.end(), bestWidget);

WidgetContainer::difference_type distance = i - vw.cbegin(); // distance will be 1
```

## Item 3: Make copying cheap and correct for objects in containers

Containers are used to store objects, but the stored ones are not exactly what you give but a copy of it. The operations later on objects in containers are also mostly based on copy. This copy is implemented by object's copy member constructor or copy assignment constructor.

```c++
class Widget {
private:
	int _data;
public:
	Widget(int x) : _data(x) {}
    
	// take good care of cp ctor and cp assiganment definition, because they may get called	frequently in STL functions
	Widget(const Widget& rhs) :_data(rhs.data()) {} 
	Widget& operator=(const Widget& rhs) {
		this->_data = rhs.data();
		return *this;
	}
	bool operator==(const Widget& rhs) { return this->data() == rhs.data(); }
	int data() const { return _data; }
};
```

A better approach to fix this is to hold pointers to objects instead objects themselves.

## Item 4: Call empty instead of checking size() against zero

Use container.empty() to check if container is empty instead of container.size() == 0.

Reason: empty is a constant-time operation for all standard containers, but for some list implementations, size takes linear time.

## Item 5: Prefer range member functions to their single-element counterparts

Let's say we have three versions of range insertion.

Common initialization:
```c++
vector<int> v1{ 99,99 };
const int numValues = 10;
int data[numValues] = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
```


* Version 1:
	```c++
	v1.insert(v1.begin(), data, data + numValues);
	```

* Version 2:
	```c++
	vector<int>::iterator insertLoc(v1.begin());
	for (int i = 0; i < numValues; ++i)
		insertLoc = v1.insert(insertLoc, data[i]);
	```

* Version 3:
	```c++
	v1.resize(20); // copy cannot assign values to the place where nothing's there before
    std::copy(data, data + numValues, v1.begin());
	```

---

* resize() vs reserve():
  
  resize() method (and passing argument to constructor is equivalent to that) will insert or delete appropriate number of elements to the vector to make it given size (it has optional second argument to specify their value). It will affect the size(), iteration will go over all those elements, push_back will insert after them and you can directly access them using the operator[].
  
  reserve() method only allocates memory, but leaves it uninitialized. It only affects capacity(), but size() will be unchanged. There is no value for the objects, because nothing is added to the vector. If you then insert the elements, no reallocation will happen, because it was done in advance, but that's the only effect.
  
  So it depends on what you want. If you want an array of 1000 default items, use resize(). If you want an array to which you expect to insert 1000 items and want to avoid a couple of allocations, use reserve()
  
---

From perspective of efficiency, Version 2 and 3 are almost same except when Version 3 may need less memory rellocations because resize() is called. Compare with Version 1, they have below disadvantages:

* More times of unnecessary function calls
* More times of elements shift
* More frequently memory rellocations

Best practice:

* Container construction:
  All standard containers provide method below.
  ```c++
  container::container(InputIterator begin, InputIterator end);
  ```

* Range insertion:
  All standard containers provide method below.
  ```c++
  container::insert(iterator position, InputIterator begin, InputIterator end);
  ```
  Since associative containers have their own mehtod to calculate insertion position, usually the first argument can be omitted.
  ```c++
  void container::container(InputIterator begin, InputIterator end);
  ```
* Range erasure:
  All standard containers provide range erasure functions, but return time are different:
  * **Sequence Container**:
    ```c++
    iterator container::erase(iterator begin, iterator end);
    ```
    It returns Iterator following the last removed element.
  * **Associative Container**:
    ```c++
    void container::erase(iterator begin, iterator end);
    ```  
* Range assignment:
  All containers provide method below
  ```c++
  void container::assign(lnputIterator begin, InputIterator end);
  ```

## Item 6: Be alert for C++'s most vexing parse

complier tends to reconize everything as function declaration.

```c++
class Widget {
private:
	int _data;
public:
	Widget() { std::cout << "default ctor\n"; }
	Widget(int x) : _data(x) {}
	Widget(const Widget& rhs) :_data(rhs.data()) {}
	Widget& operator=(const Widget& rhs) {
		this->_data = rhs.data();
		return *this;
	}
	bool operator==(const Widget& rhs) { return this->data() == rhs.data(); }
	int data() const { return _data; }
};
		
	Widget w(); // this will be regarded as a function declaration , which returns Widget and takes no arguments instead of default ctor.
```

Another example:
```c++
ifstream dataFile("ints.dat");
list<int> data(istream_iterator<int>(dataFile), istream_iterator<int>()); // warning! this doesn't do what you think it does
```

## Item 7: When using containers of newed pointers, remember to delete the pointers before the container is destroyed

For most vanila containers, when it's out of scope, elements can be destroyed by its destructor so we only focus on containers of pointers here.

```c++
class Widget {
private:
	int _data;
public:
	Widget(int x) : _data(x) {}
	Widget(const Widget& rhs) :_data(rhs.data()) {}
	Widget& operator=(const Widget& rhs) {
		this->_data = rhs.data();
		return *this;
	}
	bool operator==(const Widget& rhs) { return this->data() == rhs.data(); }
	int data() const { return _data; }
};
		
vector<Widget*> vwp;
```
* Worst method:
  ```c++
  for (int i = 0; i < 10; ++i) {
    	vwp.push_back(new Widget(i));
  }

  for (vector<Widget*>::iterator it = vwp.begin(); it != vwp.end(); ++it) {
    	delete *it;
  }
  ```
this works sometimes, but it's neither exception-safe nor clear.

* Better method:
  ```c++
  struct DeleteObject{
  	  void operator()(const Widget* ptr) const {
		  delete ptr;
	  }
  };

  for (int i = 0; i < 10; ++i) {
    	vwp.push_back(new Widget(i));
  }
  for_each(vwp.begin(), vwp.end(), DeleteObject());
  vwp.clear();
  ```
this is clear and type safe, but still not exception-safe

* Best method: use shared_ptr for everything
  ```c++
  typedef std::shared_ptr<Widget> SPW;
  vector<SPW> svwp;

  for (int i = 0; i < 10; ++i) {
	  svwp.push_back(make_shared<Widget>(i));
  }
  ```
No need to delete them explicitly, shared pointer will deal with it cleverly.


## Item 8: Never create containers of auto_ptrs. - COAP

When auto_ptr is copied, it's actually got transferred. Means the original auto_ptr will become null, this attribute make it difficult to use when certain algo called where lots of copy actions happen. As mentioned in item 7, shared_ptr should always be preferred.

## Item 9: Choose carefully among erasing options

The reason why erasure is so subtle is it may invalidate iterator, so we can never erase elements withour consideration.

* To eliminate all objects in a container that have a particular value:
  * For contiguous-memory container, use erase-remove idiom, remove will move elements behind to the position equal to removed ones, erase destroy the locations saved by remove.
    ```c++
    vector<int> vec1{ 0,1,2,3,4,5,6,7,8,9 };
    vec1.erase(std::remove(vec1.begin(), vec1.end(), 3), vec1.end());   // work for lists too, but not as efficient as below
    ```
  * For list
	```c++
    list<int> list1{ 0,1,2,3,4,5,6,7,8,9 };
	list1.remove(3);
    ```

  * For associative container, it doesn't make any sense to remove, instead, use erase
    ```c++
	set<int> set1{ 1,2,3,4,5 };
	set1.erase(3); // takes logarithmic time
    ```
		
* To eliminate all objects in a container that satisfy a particular predicate:
  * For contiguous-memory container
	```c++
    vector<int> vec2{ 0,1,2,3,4,5,6,7,8,9 };
	vec2.erase(std::remove_if(vec2.begin(), vec2.end(), badValue), vec2.end());// work for lists too, but not as efficient as below
    ``` 
  * For list
    ```c++
	list<int> list2{ 0,1,2,3,4,5,6,7,8,9 };
	list2.remove_if(badValue);
    ```
  		
  * For associative container
	* Easier version: involves copying all the elements that aren't being removed
	```c++
	set<int> set2{ 1,2,3,4,5 }, set2_goodvalues;
	remove_copy_if(set2.begin(), set2.end(), inserter(set2_goodvalues, set2_goodvalues.end()), badValue);
	set2.swap(set2_goodvalues);
    ```
		
	* Efficient version: being sure to postincrement your iterator when you pass it to erase.
	```c++
    set<int> set3{ 1,2,3,4,5 };
	for (set<int>::iterator it = set3.begin(); it != set3.end();) {
		if (badValue(*it))
			set3.erase(it++); // to avoid invalidation of it once erased
		else
			++it;
	}
    ```
* To loop in containers
  * To loop in contiguous-memory container: loop to erase with log, make sure to update your iterator with erase's return value each time we call it.
    ```c++
	vector<int> vec3{ 0,1,2,3,4,5,6,7,8,9 };
	for (vector<int>::iterator it = vec3.begin(); it != vec3.end();)
		if (badValue(*it)) {
			std::cout << "Erasing:" << *it << std::endl;
			it = vec3.erase(it);
		}
		else
			++it;
    ```
  * No worries to loop in list because it never invalidates iterators

  * To loop in associative containers, just use the same way above as efficient version for erasure

## Item 10: Be aware of allocator conventions and restrictions
- Make your allocator a template, with the template parameter T representing the type of objects for which you are allocating memory.
- Provide the typedefs pointer and reference, but always have pointer be T* and reference be T&.
- Never give your allocators per - object state. In general, allocators should have no nonstatic data members.
- Remember that an allocator's allocate member functions are passed the number of objects for which memory is required, not the number of bytes needed.Also remember that these functions return T* pointers Ma the pointer typedef), even though no T objects have yet been constructed.
- Be sure to provide the nested rebind template on which standard containers depend.

## Item 11: Understand the legitimate uses of custom allocators

A good example can be found from [here](http://www.josuttis.com/cppcode/myalloc.hpp).

Allocatoer definition:
```c++
// customized allocator
namespace MyLib {
	template <class T>
	class MyAlloc {
	public:
		// type definitions
		typedef T        value_type;
		typedef T*       pointer;
		typedef const T* const_pointer;
		typedef T&       reference;
		typedef const T& const_reference;
		typedef std::size_t    size_type;
		typedef std::ptrdiff_t difference_type;

		// rebind allocator to type U
		template <class U>
		struct rebind {
			typedef MyAlloc<U> other;
		};

		// return address of values
		pointer address(reference value) const {
			return &value;
		}
		const_pointer address(const_reference value) const {
			return &value;
		}

		/* constructors and destructor
		* - nothing to do because the allocator has no state
		*/
		MyAlloc() throw() {
		}
		MyAlloc(const MyAlloc&) throw() {
		}
		template <class U>
		MyAlloc(const MyAlloc<U>&) throw() {
		}
		~MyAlloc() throw() {
		}

		// return maximum number of elements that can be allocated
		size_type max_size() const throw() {
			return std::numeric_limits<std::size_t>::max() / sizeof(T);
		}

		// allocate but don't initialize num elements of type T
		pointer allocate(size_type num, const void* = 0) {
			// print message and allocate memory with global new
			std::cerr << "allocate " << num << " element(s)"
				<< " of size " << sizeof(T) << std::endl;
			pointer ret = (pointer)(::operator new(num * sizeof(T)));
			std::cerr << " allocated at: " << (void*)ret << std::endl;
			return ret;
		}

		// initialize elements of allocated storage p with value value
		void construct(pointer p, const T& value) {
			// initialize memory with placement new
			new((void*)p)T(value);
		}

		// destroy elements of initialized storage p
		void destroy(pointer p) {
			// destroy objects by calling their destructor
			p->~T();
		}

		// deallocate storage p of deleted elements
		void deallocate(pointer p, size_type num) {
			// print message and deallocate memory with global delete
			std::cerr << "deallocate " << num << " element(s)"
				<< " of size " << sizeof(T)
				<< " at: " << (void*)p << std::endl;
			::operator delete((void*)p);
		}
	};

	// return that all specializations of this allocator are interchangeable
	template <class T1, class T2>
	bool operator== (const MyAlloc<T1>&,
		const MyAlloc<T2>&) throw() {
		return true;
	}
	template <class T1, class T2>
	bool operator!= (const MyAlloc<T1>&,
		const MyAlloc<T2>&) throw() {
		return false;
	}
}
```

Example to use:
```c++
// create a vector, using MyAlloc<> as allocator
std::vector<int, MyLib::MyAlloc<int> > v;

// insert elements
// - causes reallocations
v.push_back(42);
v.push_back(56);
v.push_back(11);
v.push_back(22);
v.push_back(33);
v.push_back(44);
```

Output:
```
allocate 1 element(s) of size 8
 allocated at: 008AE378
allocate 1 element(s) of size 4
 allocated at: 008ACA30
allocate 2 element(s) of size 4
 allocated at: 008AE2D0
deallocate 1 element(s) of size 4 at: 008ACA30
allocate 3 element(s) of size 4
 allocated at: 008AE228
deallocate 2 element(s) of size 4 at: 008AE2D0
allocate 4 element(s) of size 4
 allocated at: 008B21C0
deallocate 3 element(s) of size 4 at: 008AE228
allocate 6 element(s) of size 4
 allocated at: 008ACA30
deallocate 4 element(s) of size 4 at: 008B21C0
deallocate 6 element(s) of size 4 at: 008ACA30
deallocate 1 element(s) of size 8 at: 008AE378
```

## Item 12: Have realistic expectations about the thread safety of STL containers

Safe points:
- Multiple readers are safe. 
  Multiple threads may simultaneously read the contents of a single container, and this will work correctly. Naturally, there must not be any writers acting on the container during the reads.

- Multiple writers to different containers are safe. 
  Multiple threads may simultaneously write to different containers.

What we need to do:
- Lock a container for the duration of each call to its member functions.
- Lock a container for the lifetime of each iterator it returns(via, e.g.., calls to begin or end).
- Lock a container for the duration of each algorithm invoked on that container. (This actually makes no sense, because, as Item 32 explains, algorithms have no way to identify the container on which they are operating.Nevertheless, we'll examine this option here, because it's instructive to see why it wouldn't work even if it were possible.)

```c++
vector<int> v;
// Lock aquired here
vector<int>::iterator first5(find(v.begin(), v.end(), 5));
	if (first5 != v.end()) {
		*first5 = 0;
	}
// Lock released here
```

## Item 13: Prefer vector and string to dynamically allocated arrays.

Shortcomings of dynamically allocated arrays:
1. You must make sure that somebody will later delete the allocation. Without a subsequent delete, your new will yield a resource leak.
2. You must ensure that the correct form of delete is used.For an allocation of a single object, "delete" must be used.For an array allocation, "delete []" is required. If the wrong form of delete is used, results will be undefined. On some platforms, the program will crash at runtime. On others, it will silently blunder forward, sometimes leaking resources and corrupting memory' as it goes.
3. You must make sure that delete is used exactly once. If an allocation is deleted more than once, results are again undefined.

## Item 14: Use reserve to avoid unnecessary reallocations
- size() tells you how many elements are in the container. It does not tell you how much memory the container has allocated for the elements it holds.
- capacity() tells you how many elements the container can hold in the memory it has already allocated.This is how many total elements the container can hold in that memory, not how many more elements it can hold. If you'd like to find out how much unoccupied memory a vector or string has, you must subtract size() from capacity(). If size and capacity return the same value, there is no empty space in the container, and the next insertion (via insert or push_back. etc.) will trigger the reallocation steps above.
- resize(size_t n) forces the container to change to n the number of elements it holds.After the call to resize, size will return n. If n is smaller than the current size, elements at the end of the container will be destroyed. If n is larger than the current size, new default - constructed elements will be added to the end of the container. If n is larger than the current capacity, a reallocation will take place before the elements are added.
- reserve(size_t n) forces the container to change its capacity to at least n.provided n is no less than the current size.This typically forces a reallocation, because the capacity needs to be increased. (If n is less than the current capacity, vector ignores the call and does nothing, string may reduce its capacity to the maximum of size() and n.but the string's size definitely remains unchanged. Using reserve to trim the excess capacity from a string is generally less successful than using "the swap trick." which is the topic of Item 17.)

Example:
```c++
vector<int> v;
v.reserve(1000); // this reserve can save you from 10 times of reacllocation 
for (int i = 1; i <= 1000; ++i) 
	v.push_back(i);
```

## Item 15: Be aware of variations in string implementations
		
- string values may or may not be reference counted.By default, many implementations do use reference counting, but they usually offer a way to turn it off, often via a preprocessor macro.Item 13 gives an example of specific conditions under which you might want to turn it off, but you might want to do so for other reasons, too.For example, reference counting helps only when strings are frequently copied, and some applications just don't copy strings often enough to justify the overhead.
- string objects may range in size from one to at least seven times the size of char* pointers.
- Creation of a new string value may require zero, one, or two dynamic allocations.
- string objects may or may not share information on the string's size and capacity.
- strings may or may not support per - object allocators. 
- Different implementations have different policies regarding minimum allocations for character buffers.

## Item 16: Know how to pass vector and string data to legacy APIs

```c++
vector<int> v;
for (int i = 1; i <= 10; ++i)
	v.push_back(i);

print(&v[0], 3); // works
print(&*v.begin(), 3); // also works
//print(v.begin(), 3); // doesn't work

std::string str = "abc";
print(str.c_str(), 3);
//print(&str[0], 3); // sometimes doesn't work because value is not contagious allocated.
```

## Item 17: Use "the swap trick" to trim excess capacity

vector's copy constructor allocates only as much memory as is needed for the elements being copied, so this temporary vector has no excess capacity. We then swap the data in the temporary vector with that in contestants, and by the time we're done, contestants has the trimmed capacity of the temporary, and the temporary holds the bloated capacity that used to be in contestants

Example:
```c++
const int MAX = 1024;

#define show(v, msg)    printf ("%s -- %s: size: %lu, capacity: %lu\n", #v, msg, v.size(), v.capacity());
```


```c++
srand(time(NULL));

printf("Testing vector...\n");
vector<int> v;
show(v, "After init");

v.reserve(MAX);
show(v, "After reserve()");

for (int i = 0; i < MAX; ++i)
	v.push_back(rand() % 1000);

show(v, "After Filling");

v.erase(remove_if(v.begin(), v.end(), [](int x) {return x > 100; }), v.end());
show(v, "After erase()");

vector<int>(v).swap(v);
show(v, "After swap()");

v.clear();
show(v, "after clear");

vector<int>().swap(v);
show(v, "after swap with empty vector");
```

Output:
```
Testing vector...
v -- After init: size: 0, capacity: 0
v -- After reserve(): size: 0, capacity: 1024
v -- After Filling: size: 1024, capacity: 1024
v -- After erase(): size: 107, capacity: 1024
v -- After swap(): size: 107, capacity: 107
v -- after clear: size: 0, capacity: 107
v -- after swap with empty vector: size: 0, capacity: 0
```

## Item 18: Avoid using vector of bool
Among the requirements is that if c is a container of objects of type T and c supports operator[], the following must compile:
```c++
T *p = &c[0];
```

This will not complie:
```c++
vector<bool> v{ true, false };
bool* p = &v[0];
```

Solutions: use deque<bool> or bitset instead.

## Item 19: Understand the difference between equality and equivalence

Equality: based on operator ==. We say A is euqal to B if "A == B" is true. Algorithms such as std::find use equality to find expected elements.

Equivalence: based on "Compare" operator, which is Less<> by default. We say A and B have equivalent values if expression below is true:
```c++
!(A < B) && !(B < A)
```
Standard associative containers is always sorted respect to equivalence. In other words, if two elements are equivalent by "Compare", they cannot exist in same container.

ciStinrgCompare is a functor that do case-insensitive string compare.
```c++
set<string, ciStringCompare> s1;
s1.insert("Persephone");
s1.insert("persephone");

set<string> s2;
s2.insert("Persephone");
s2.insert("persephone");
std::cout << "element count in s1:" << s1.size() << std::endl; // print 1
std::cout << "element count in s2:" << s2.size() << std::endl; // print 2

std::cout << (std::find(s1.begin(),s1.end(), "persephone") == s1.end()); // print true because std::find use string::operator==
std::cout << (s1.find("persephone") == s1.end()); // print false, it will use equivalence
```


## Item 20: Specify comparison types for associative containers of pointers

Remember to define a comparison for associative containers of pointers, otherwise the elements will be sorted by the pointer values, which are the hexadecimal memory addresses.

## Item 21: Always have comparison functions return false for equal values

As mentioned in Item 19, by default, set<int> is set<int, less<int>>. If we use set<int,less_equal<int>> to declare a set, we will have duplicates elements inserted without knowing it:

```c++
set<int,less_equal<int>> s;
s.insert(10);
s.insert(10); // try inserting 10 again and will cause undefined behaviour, which is not expected at all!
```

The reason is obvious, container will check if we have "same" key value already, which is
```c++
!(10 <= 10) && !(10 <= 10)
```
and get a false value indicating 10 and 10 are not equivalent.

## Item 22: Avoid in-place key modification in set and multiset

Since elements in set and multiset are sorted by key value, key modification will make modified elements in wrong locations and would  break the sorted=ness then cause unproper behaviors.

When set non-key values, some STL implementation may reject it because find method returns a const pointer. You need to cast constness away before modification:
```c++
Employee selectedID; // a dummy employee with the selected ID number
EmplDSet::iterator i = se.find(selectedlD);
if (i != se.end()) { 
	const_cast<Employee&>(*i).setTitle("Corporate Deity"); // cast away constness
}
```

If you really want to change the key value, erase it and insert a new node like the way below:
```c++
EmplDSet::iterator i = se.find(selectedlD); // Step 1: find element to change
if(i!=se.end()){
	Employee e(*i); // Step 2: copy the element
	se.erase(i++); // Step 3: remove the element; increment the iterator to maintain its validity (see Item 9)
	e.setTitle("Corporate Deity"); // Step 4: modify the copy
	se.insert(i, e); // Step 5: insert new value; hint that its location is the same as that of the original element
}
```

## Item 23: Consider replacing associative containers with sorted vectors

Associative containers is more efficient at average level, and this is concluded when we have insertions, erasure, search actions mixed. If the behaviour follows pattern below, a sorted vector might be a better option.

1. **Setup**. Create a new data structure by inserting lots of elements into it. During this phase, almost all operations are insertions and erasures. Lookups are rare or nonexistent.
2. **Lookup**. Consult the data structure to find specific pieces of information. During this phase, almost all operations are lookups.. Insertions and erasures are rare or nonexistent.
3. **Reorganize**. Modify the contents of the data structure, perhaps by erasing all the current data and inserting new data in its place. Behaviorally, this phase is equivalent to phase 1. Once this phase is completed, the application returns to phase 2.


Sometimes you may want to replace map and multimap with vectors as well. Just notice to have pair<K,V> as your element in vectors instead of pair<const K, V>, because when you sort the vector, the values of its elements will get moved around via assignment, and that means that both components of the pair must be assignable.

## Item 24: Choose carefully between map::operator[] and map-insert when efficiency is important

map::operator[] returns a reference to the value object associated with k. v is then assigned to the object to which the reference (the one returned from operator[]) refers. But if k isn't yet in the map, there's no value object for operator[] to refer to. In that case, it creates one from scratch by **using the value type's default constructor, operator[] then returns a reference to this newly-created object**. Then assign value to this new object.

In short, map::operator[] is efficient to update existing nodes but not as good as insert when inserting new nodes.

A possible efficient way (use it only when this matters):
```c++
template< typename MapType, typename KeyArgType, typename ValueArgtype> 
typename MapType::iterator efficientAddOrUpdate(MapType& m, const KeyArgType& k, const ValueArgtype& v) {
	typename MapType::iterator Ib = m.lower_bound(k); 
	if(lb != m.end() && !(m.key_comp()(k, lb->first))) { // update
		lb->second = v;
		return Ib;
	}
	else{
	typedef typename MapType::value_type MVT;
	return m.insert(lb, MVT(k, v)); // insert
	}
}
```

## Item 25: Familiarize yourself with the nonstandard hashed containers

Understand how hashed containers are implemented and related algorithms complexcity.

## Item 26: Prefer iterator to const iterator, reverse_iterator, and const_reverse_iterator

Most functions support iterators as arguments and STL has implicit conversions from others to iterator. Refer the diagram below:

![iterator_conversion](../_post_img/iterator_conversion.PNG)

- Some versions of insert and erase require iterators.
- It's not possible to implicitly convert a const iterator to an iterator, and the technique described in Item 27 for generating an iterator from a const_iterator is neither universally applicable nor guaranteed to be efficient.
- Conversion from a reverse_iterator to an iterator may require iterator adjustment after the conversion.

## Item 27： Use distance and advance to convert a container's const_iterators to iterators

In order to convert a const_iterator to iterator, you can do something similar to this:

```c++
deque<int> de{ 1,2,3,4,5 };
deque<int>::iterator i = de.begin();
deque<int>::const_iterator cend = de.cend();
advance(i, distance<deque<int>::const_iterator>(i, cend)); // i will point to the place to which de.end() points to
```

## Item 28: Understand how to use a reverse_iterator's base iterator

Use a example to demonstrate:

```c++
vector<int> v{ 1,2,3,4,5 };
vector<int>::reverse_iterator ri = find(v.rbegin, v.rend, 3);
vector<int>::iterator i(ri.base());
```

After the execution, things will be like this:

![iterator_conversion](../_post_img/reverse_iterator.PNG)

- To emulate insertion at a position specified by a reverse_iterator ri, insert at the position ri.base() instead. For purposes of insertion, ri and ri.base() are equivalent, and ri.base() is truly the iterator corresponding to ri.

- To emulate erasure(modification) at a position specified by a reverse_iterator ri, erase at the position preceding ri.base() instead. For purposes of erasure, ri and ri.base() are nor equivalent, and ri.base() is nor the iterator corresponding to ri. Do something similar to

  ```c++
  v.erase(++ri).base());
  ```

## Item 29: Consider istreambuf_iterators for character-by-character input

Not used yet, will add in the future

## Item 30: Make sure destination ranges are big enough

Each algorithm that uses a destination range, writes its results by making **assignments** to the elements in the destination range. 

So below code will not work because results is empty.
```c++
int tran_add(int x);
vector<int> values {1, 2, 3, 4};
vector<int> results;

transform(values.begin(), values.end(), results.end(), tran_add); // error
```

Correct ways:
```c++
transform(values.begin(), values.end(), back_inserter(results), tran_add);
transform(values.begin(), values.end(), front_inserter(results), tran_add);
transform(values.begin(), values.end(), inserter(results, results.begin() + results.size()/2, tran_add);
```

## Item 31: Know your sorting options

Stable sort: In a stable sort, if two elements in a range have equivalent values, their relative positions are unchanged after sorting.

- If you need to perform a full sort on a vector, string, deque, or array, you can use sort or stable_sort
- If you have a vector, string, deque, or array and you need to put only the top n elements in order, partial_sort is available
- If you have a vector, string, deque, or array and you need to identify the element at position n or you need to identify the top n elements without putting them in order. nth_element is at your beck and call
- If you need to separate the elements of a standard sequence container or an array into those that do and do not satisfy some criterion, you're probably looking for partition or stable_partition
- If your data is in a list, you can use partition and stable_partition directly, and you can use list-sort in place of sort and stable_sort. If you need the effects offered by partial_sort or nth_element, you'll have to approach the task indirectly

Sort algorithms Efficiency comparision: Principle is that algorithms that do more work take longer to do it. and algorithms that must sort stably take longer than algorithms that can ignore stability.

**Efficiency: partition > stable_partition > nth_element > partial_sort > sort > stable_sort**

Indirect approach for list to use partial_sort or nth_element:
1. Copy the elements into a container with random access iterators, then apply the desired algorithm to that. 
2. Create a container of list::iterators, use the algorithm on that container, then access the list elements via the iterators. 
3. Use the information in an ordered container of iterators to iteratively splice the list's elements into the positions you'd like them to be in.

## Item 32: Follow remove-like algorithms by erase if you really want to remove something

Understand that remove algorithm is just moving elements behind to fill the location of elements to be removed. Its size will not even change.

```c++
vector<int> v_remove{ 1,2,3,99,5,99,7,8,9,99 };
remove(v_remove.begin(), v_remove.end(), 99);
for_each(v_remove.begin(), v_remove.end(), [](int it) {cout << it << " "; }); // print 1 2 3 5 7 8 9 8 9 99
```

**Always use erase to really remove elements after the new end iterator returned by remove!!!**

## Item 33: Be wary of remove-like algorithms on containers of pointers

For a vector of pointers of Widget:

![remove_if_pointer1](../_post_img/remove_if_pointer_1.PNG)

After remove_if: Memory leak already happens!

![remove_if_pointer2](../_post_img/remove_if_pointer_2.PNG)

After erase:

![remove_if_pointer3](../_post_img/remove_if_pointer_3.PNG)

Solution:

- Delete pointers first by the critiria then remove if. It may require more loops.
- Use shared_ptr.

## Item 34: Note which algorithms expect sorted ranges

Some algorithms expect ranges that are already sorted:
- binary_search
- lower_bound
- upper_bound
- equal_range
- set_union
- set_intersection
- set_difference
- set_symmetric_difference
- merge
- inplace_merge
- includes

Algorithms that are typically used with sorted ranges, though they don't require them:
- unique
- unique_copy

**Passing unsorted ranges may cause undefined behaviour!**

## Item 35: Implement simple case-insensitive string comparisons via mismatch or lexicographical compare

Below is the implementation via lexicographical compare
```c++
bool ciCharLess(char c1, char c2) {
	return tolower(static_cast<unsigned char>(c1)) < tolower(static_cast<unsigned char>(c2)); 
} 

bool ciStringCompare(const string& s1, const string& s2)
{
	return lexicographical_compare(s1.begin(), s1.end(), s2.begin(), s2.end(), ciCharLess);
}

int main() {
	
	std::cout << std::boolalpha << ciStringCompare("abc", "Abc") << std::endl;
	std::cout << std::boolalpha << ciStringCompare("Abc", "abc") << std::endl;

	return 0;
}
```

output:
```
false
false
```

This means "abc" and "Abc" will be regarded as equivalent by ciStringCompare.

## Item 36: Understand the proper implementation of copy_if

"copy" algorithms:
- copy
- copy_backward
- replace_copy
- reverse_copy
- replace_copy_if
- unique_copy
- remove_copy
- rotate_copy
- remove_copy_if
- partial_sort_copy_unintialized_copy
- unintialized_copy

Since we don't have a copy_if algorithm, we can do it in this way:
```c++
template<typename InputIterator,  typename OutputIterator, typename Predicate>
OutputIterator copy_if(InputIterator begin, InputIterator end, OutputIterator destBegin, Predicate p) {
	while (begin != end) {
 		if (p(*begin))*destBegin++ = *begin;
 		++begin;
 	}
 	return destBegin;
}
```

## Item 37: Use accumulate or for_each to summarize ranges

Make sure the start value in accumulate's type same as elements.

```c++
list<double> ld;
double sum = accumulate(ld.begin(), Id.end(), 0.0); // Do not put 0 here.
```

## Item 38: Make your functor pass-by-value

Two things you need to ensure:

1. Fucntor should be small so they can be copied with low cost
2. Your function objects must be monomorphic

A bridge design pattern might be needed to make it happen because it only needs to copy the pointer:
```c++
template<typename T>
class BPFCImpl : public unary_function<T, void> {
private:
 	Widget w;
 	int x;
 	//...
 	virtual ~BPFCImpl();
 	virtual void operator()(const T& val) const;
 	friend class BPFC<T>;
};

template<typename T>
class BPFC:
 	public unary_function<T, void> {
private:
 	BPFCImpl<T> *pImpl;
public:
 	void operator()(const T& val) const { 
 	pImpl->operator() (val);
 	}
    //...
};
```

## Item 39: Make predicates pure functions

Definition:

- A **predicate** is a function that returns bool

- A **pure function** is a function whose return value depends only on its parameters

A bad example:
```c++
bool anotherBadPredicate(const Widget&, const Widget&) {
	static int timesCalled = 0; 
 	return ++timesCalled == 3; // its return value depends on static variables
} 
```
## Item 40 ~ 41: Make functor classes adaptable, Understand the reasons for ptr_fun, mem_fun, and mem_fun_ref

Before C++11, functinal programming is limited within uniary_fucntion, binary_function. They have to be adapted by using ptr_fun, mem_fun, and mem_fun_ref. 

Nowadays, we prefer lambda functions and std::functions to define these functors by which we can avoid these adaption issues.

## Item 42: Make sure less<T> means operator<

It's natural to have this claim that less<T\> should call T's operator<. If you really want an associative container to be sorted by a special compare, define a seperate functor to realize it instead of calling it less which will make people confused.


## Item 43: Prefer algorithm calls to hand-written loops

There are three reasons:
- Efficiency: Algorithms are often more efficient than the loops programmers produce.
- Correctness: Writing loops is more subject to errors than is calling algorithms.
- Maintainability: Algorithm calls often yield code that is clearer and more straightforward than the corresponding explicit loops.

## Item 44: Prefer member functions to algorithms with the same names

When you're comparing std::find and std::set::find, there is no doubt that a specilized version works better.

std::set::find will utilize set's property of being sorted, to do the binary search whose complecity is O(logN). On the other hand, std::find is a generalized method which has a O(logN) complexcity.

## Item 45: Distinguish among count, find, binary search, lower_bound, upper_bound, and equal_range


|What You Want to Know   | On an Unsorted range  | On a Sorted Range  | With a set or map  | With a multiset or multimap  |
|---|---|---|---|---|
| Does the desired value exist?  | find  | binary_search  | count  | find  |
| Does the desired value exist? If so, where is the fisrt object with that value?  | find  | equal_range | find  | find/lower_bound   |
| Where is the first object value not preceding the desired value?  | find_if  | lower_bound  | lower_bound  | lower_bound  |
| Where is the first object value succeeding the desired value?  | find_if  | upper_bound  | upper_bound  | upper_bound  |
| How many objects have the desried value?  | count  | equal_range  | count  | count  |
| Where are all the objects with the desired value?  | find(iteratively)  | equal_range  | equal_range  | equal_range  |

Note: On an Unsorted range / On a Sorted Range are using general algorithms,  With a set or map / With a multiset or multimap are using member functions.

## Item 46: Consider function objects instead of functions as algorithm parameters

1. For higher efficiency, the reaosn is inline functions. greater<double>::operator() is an inline function, which is faster than user-defined functions.
2. It can avoid some unkonwn bugs on certain STL platforms.
```c++
// below will not complie
set<string> s;
//...
transform(s.begin(), s.end(), ostream_iterator<string::size_type>(cout, "\n"), mem_fun_ref(&string::size));

// below complies
struct StringSize:
 public unary_function<string, string::size_type>{ 
 string::size_type operator()(const string& s) const // remember to declare const
 {
 return s.size();
 }
};

transform(s.begin(), s.end(), ostream_iterator<string::size_type>(cout, "\n"), StringSize());
```
## Item 47: Avoid producing write-only code.

Increase the readablity of your code when using STL. Don't have somthing like this:
```c++
vector<int> v;
int x, y;
//...
v.erase(
 remove_if(find_if(v.rbegin(), v.rend(),
 bind2nd(greater_equal<int>(), y)).base(),
 v.end(),
 bind2nd(less<int>(), x)),
 v.end());
```

## Item 48: Always #include the proper headers

Tips for including:
1. Almost all the containers are declared in headers of the same name, i.e., vector is declared in <vector>. list is declared in <list>, etc. The exceptions are <set> and <map>. <set> declares both set and multiset, and <map> declares both map and multimap
2. All but four algorithms are declared in <algorithm>. The exceptions are accumulate (see Item 37), inner_product, adjacent_difference, and partial_sum. Those algorithms are declared in <numeric>
3. Special kinds of iterators, including istream_iterators and istreambuf_iterators (see Item 29), are declared in <iterator>.
4. Standard functors (e.g., less<T>) and functor adapters (e.g., not1, bind2nd) are declared in <functional>.

## Item 49: Learn to decipher STL-related compiler diagnostics

Here are some common mistakes you may make:
- For vector and string, iterators are usually pointers, so compiler diagnostics will likely refer to pointer types if you've made a mistake with an iterator. For example, if your source code refers to vector<double>::iterators, compiler messages will almost certainly mention double* pointers
- Messages mentioning back_insert_iterator, front_insert_iterator, or insert_iterator almost always mean you've made a mistake calling back_inserter, front_inserter, or inserter
- If you get a message mentioning binder1st or binder2nd, you've probably made a mistake using bind1st or bind2nd
- Output iterators (e.g., ostream_iterators, ostreambuf_iterators (see Item 29), and the iterators returned from back_inserter, front_inserter, and inserter) do their outputting or inserting work inside assignment operators, so if you've made a mistake with one of these iterator types, you're likely to get a message complaining about something inside an assignment operator you've never heard of
- If you get an error message originating from inside the implementation of an STL algorithm, there's probably something wrong with the types you're trying to use with that algorithm
- If you're using a common STL component like vector, string, or the for_each algorithm, and a compiler says it has no idea what you're talking about, you've probably failed to #include a required header file


## Item 50: Familiarize yourself with STL-related web sites
- The SGI STL site, http://www.sgi.com/tech/stl/.
- The STLport site, http://www.stlport.org/.
- The Boost site, http://www.boost.org/.