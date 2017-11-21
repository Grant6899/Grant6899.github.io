# Effective STL Notes

What written here is a brief summary for what I've learnd from [Effective STL](https://www.amazon.com/Effective-STL-Specific-Standard-Template/dp/0201749629) by Scott Meyers. The purpose of doing this is to remind myself every now and then in case these tricks get rusty.


## Item 1: Choose your containers with care

Here's just an overview of how the author categorizes well-known STL containers:

standard STL sequence containers: vector, string, deque, list

standard STL associative containers: set, multiset, map, multimap

nonstandard sequence containers: slist, rope

nonstandard associative containers: hash_set, hash_multiset, hash_map, hash_multimap

contiguous-memory containers (array-based containers): vector, string, deque | rope

node-based containers: list, slist and all standard associative containers


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