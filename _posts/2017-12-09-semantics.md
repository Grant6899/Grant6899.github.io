---
layout:		post
title:		Value Semantic vs Reference Semantic
subtitle:
date:		2017-12-08
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - java
	- python
	- deep copy
---


## Definition

Value semantic means the copy of objects has nothing to do with the original object. After the copy, the new object will be detached from the model one. They are independent and will not impact each other.


We say a type is reference semantic when either of below holds:

- Object of the type is non-copyable
- After the copy, both objects still share the underlying resource. Change of either will change the underlying (shallow copy).



## Example in Java, C++ and Python

In java, objects of primitive data types are value semantic.

```java
int a = 8;
int b = a;
```
Here a and b are unrelated after copy.

On the other hand, for a user-defined class Employee:
```java
Employee a1 = new Employee("zhangsan", 23);
Employee a2 = a1;
a2.setAge(30);
System.out.println(a1.getAge());
```
The output is 30. The reason is user-difined class Employee is reference semantic, a1 and a2 both point to the same actual object. Changing any of them will change the underlying value.


In C++, all the built-in types obviously behave like value. For user-defined classes, **by default** they are value semantic. However, C++ gives us the option to modify the copy constructor and override assignment operator to make it reference semantic.

```c++
class ItemBook{
private:
    string isbn_;
    double price_;
    int amount_;
};
```

```c++
ItemBook a1;
// set member of a1;
ItemBook a2 = a1;
```
After copy, a1 is independent of a2 since the copy is implemented by complier with default method.


In Python, we have two groups of types: 

1. immutable type, basic types like integer and string is value sematic
    ```python
    str1 = "abc"
    str2 = str1
    id(str1)
    id(str2)
    str2 = str2 + "d"
    id(str1)
    id(str2)
    ```
2. complex type/class like list is reference semantic
    ```python
	list1 = ['apple', 'mango', 'carrot', 'banana']
    list2 = list1
    del list2[0]
    print(list1)
    ```
   You will see both list1 and list2's first element is erased. This is like what we see in Java.
   
   **To create a copy, you can assign a slice to another list.**
   ```python
   list2 = list1[:]
   ```
   It will be an independent copy(like deep copy in C++, new memory will be allocated). For a general deep copy, you can do:
   ```python
   from copy import deepcopy

   lst1 = ['a','b',['ab','ba']]

   lst2 = deepcopy(lst1)

   lst2[2][1] = "d"
   lst2[0] = "c";
   print lst2
   print lst1
   ```


## How to make reference semantic type non-copyable

- 1. Make the parent abstract calss's copy consturcotr and assignment operator **private**, and do not provide implementation

- 2. Make the type derived from a NonCopyable class

```c++
class NonCopyable
{
protected: 
    NonCopyable() {}
    ~NonCopyable() {}
private:
    NonCopyable(const NonCopyable &);
    const NonCopyable &operator=(const NonCopyable &);
};

class Child : private NonCopyable
{
public:
    virtual double Calc() const = 0;
    virtual ~Child(void) {}
};
```


## Object Life Cycle

The life cycle of value semantic object is easy to control. A value semantic object is either an object on the stack or a data member of another object.

On the other hand, reference semantic objects cannot be copied, we must use pointer or reference to use it. Then it will be our concern if the object pointed is released properly.

A good solution to handle it is smart pointer. Essentially, a smart pointer convert reference semantic to value semantic. A local object(smart pointer)'s destruction is defined in the corresponding smart pointer class.






