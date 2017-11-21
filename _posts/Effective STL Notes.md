# Effective STL Notes

What written here is a brief summary for what I've learnd from [Effective STL](https://www.amazon.com/Effective-STL-Specific-Standard-Template/dp/0201749629) by Scott Meyers. The purpose of doing this is to remind myself every now and then in case these tricks get rusty.


## Item 1: Choose your containers with care

Here's just an overview of how the author categorizes well-known STL containers:

* **standard** STL **sequence** containers: vector, string, deque, list
* **standard** STL **associative** containers: set, multiset, map, multimap
* **nonstandard** **sequence** containers: slist, rope
* **nonstandard associative** containers: hash_set, hash_multiset, hash_map, hash_multimap
* **contiguous-memory** containers (array-based containers): vector, string, deque | rope
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
vector<int>::iterator first5(find(v.begin(), v.end(), 5)); // Line 1
	if (first5 != v.end()) { //Line 2
		*first5 = 0; // Line 3
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

# Item 16: Know how to pass vector and string data to legacy APIs

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

## Item 18: Avoid using vector<bool>
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