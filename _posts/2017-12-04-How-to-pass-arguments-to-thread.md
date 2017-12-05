---
layout:		post
title:		How to pass arguments to thread
subtitle:
date:		2017-12-04
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Boost
    - Multi-thread
---

## Thread function must return void

The function used to initialize threads cannot retrun anything but void.
```c++
void partial_sum(int st, int ed, boost::uint64_t& total) {
	for (int i = st; i < ed; ++i) 
		total += i;
}
```

In the funciton above, partial_sum is adding the sum of [st, ed) on total. 


## Boost Reference need to be used with "&" in function head


However, only using **boost::uint64_t&** will not make your program work in the way expected. When initializing the thread, we need to add **boost::ref()** as well:
```c++
boost::thread t1(partial_sum, 0, 500000000,  boost::ref(sum_1));
```

## Alternative way: pointer

Thread function definition:
```c++
void partial_sum(int st, int ed, boost::shared_ptr<boost::uint64_t> total) {
	for (int i = st; i < ed; ++i) 
		*total += i;
}
```

To initialize the thread:
```c++
auto sum_1 = boost::make_shared<boost::uint64_t>(0);
boost::thread t1(partial_sum, 0, 500000000,  sum_1);
```
After this function, *sum_1 will become the sum value.

