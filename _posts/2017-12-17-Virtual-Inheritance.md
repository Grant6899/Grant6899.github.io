---
layout:		post
title:		Virtual Inheritance
subtitle:
date:		2017-12-17
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Multiple Inheritance
---

## Diamond Inheritance Problem

As we all know, when we're trying to create classes in the below structure:

![Diamond](../_post_img/MI.jpg)

A natural way comes to our mind is
```c++
class Singer : public Worker {...};
class Waiter : public Worker {...};
class SingingWaiter : public Waiter, public Singer {...};
```

But it causes problems since ed contains two worker objects:
```c++
SingingWaiter ed;
Worker * pw = &ed;   // ambiguous
```

Although it can be resolved by manual casting:
```c++
Worker * pw1 = (Waiter *) &ed;   // the Worker in Waiter
Worker * pw2 = (Singer *) &ed;   // the Worker in Singer
```
it's not what expected since we'd like to see dynamic binding here.


## Base classes

Below are the base classes we discussed in this blog.

```c++
class Worker   // an abstract base class
{
private:
    std::string fullname;
    long id;
public:
    Worker() : fullname("no one"), id(0L) {}
    Worker(const std::string & s, long n)
            : fullname(s), id(n) {}
    virtual ~Worker() = 0;   // pure virtual destructor
    virtual void Set();
    virtual void Show() const;
};

class Waiter : public Worker
{
private:
    int panache;
public:
    Waiter() : Worker(), panache(0) {}
    Waiter(const std::string & s, long n, int p = 0)
            : Worker(s, n), panache(p) {}
    Waiter(const Worker & wk, int p = 0)
            : Worker(wk), panache(p) {}
    void Set();
    void Show() const;
};

class Singer : public Worker
{
protected:
    enum {other, alto, contralto, soprano,
                    bass, baritone, tenor};
    enum {Vtypes = 7};
private:
    static char *pv[Vtypes];    // string equivs of voice types
    int voice;
public:
    Singer() : Worker(), voice(other) {}
    Singer(const std::string & s, long n, int v = other)
            : Worker(s, n), voice(v) {}
    Singer(const Worker & wk, int v = other)
            : Worker(wk), voice(v) {}
    void Set();
    void Show() const;
};
```

## Virtual Base Classes

**Virtual base classes allow an object derived from multiple bases that themselves share a common base to inherit just one object of that shared base class.**

```c++
class Singer : virtual public Worker {...};
class Waiter : virtual public Worker {...};
class SingingWaiter : public Waiter, public Singer {...};
```


### Constructor Rule

***One Line Rule: If the virtual base class is not constructed explicitly and separtely, then it's created by its default constructor.***

The problem is that automatic passing of information would pass wk to the Worker object via two separate paths (Waiter and Singer). **To avoid this potential conflict, C++ disables the automatic passing of information through an intermediate class to a base class if the base class is virtual.** 


```c++
SingingWaiter(const Worker & wk, int p = 0, int v = Singer::other)
                  : Waiter(wk,p), Singer(wk,v) {}  // flawed
```

Thus, the constructor above will initialize the panache and voice members, but the information in the wk argument won’t get to the Waiter subobject. However, the compiler must construct a base object component before constructing derived objects; **in this case, it will use the default Worker constructor.**

If you want to use something other than the default constructor for a virtual base class, you need to invoke the appropriate base constructor explicitly. Thus, the constructor should look like this:

```c++
SingingWaiter(const Worker & wk, int p = 0, int v = Singer::other)
                  : Worker(wk), Waiter(wk,p), Singer(wk,v) {}
```
Here the code explicitly invokes the Worker(const Worker &) constructor. **Note that this usage is legal and often necessary for virtual base classes, and it is illegal for nonvirtual base classes.**

### Method Distinguish

To explicitly tell complier which method is used, "::" and the namepace where it is declared will work. Below is an example if you want to overwrite SingingWaiter's Show method by Showing both Singer part and Waiter part.

```c++
void SingingWaiter::Show()
{
      Singer::Show();
      Waiter::Show();
}
```

### Example

```c++
// multiple inheritance
class SingingWaiter : public Singer, public Waiter
{
protected:
    void Data() const;
    void Get();
public:
    SingingWaiter()  {}
    SingingWaiter(const std::string & s, long n, int p = 0,
                            int v = other)
            : Worker(s,n), Waiter(s, n, p), Singer(s, n, v) {}
    SingingWaiter(const Worker & wk, int p = 0, int v = other)
            : Worker(wk), Waiter(wk,p), Singer(wk,v) {}
    SingingWaiter(const Waiter & wt, int v = other)
            : Worker(wt),Waiter(wt), Singer(wt,v) {}
    SingingWaiter(const Singer & wt, int p = 0)
            : Worker(wt),Waiter(wt,p), Singer(wt) {}
    void Set();
    void Show() const;
};
```

### Memory Layout and Low-level realization

[Reference](http://www.jianshu.com/p/4a65e84559db)





