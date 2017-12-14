---
layout:		post
title:		Thoughts from a Rational class
subtitle:
date:		2017-12-11
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++11
    - rvalue
    - Rational
---

## Source Code

I wrote a Rational class for fun during weekends. Below is the header file:

```c++
/*************************************************************************
> File Name: Rational.h
> Author: Grant Liu
> Mail: ymliu6899@gmail.com
> Created Time: Sat Dec  9 23:14:28 2017
************************************************************************/

#ifndef _RATIONAL_H
#define _RATIONAL_H

#include<iostream>

using namespace std;

class Rational {
private:
	int p_, q_;
	void Standardize();
	int gcd(int m, int n);

public:
	Rational(int p = 0, int q = 1) : p_(p), q_(q) {
		Standardize();
	};

	Rational(const Rational &r) : p_(r.p_), q_(r.q_) {
		Standardize();
	};

	Rational(double ra);

	virtual ~Rational() = default;
	void Reduce();

	int GetNumerator() const {
		return p_;
	}

	int GetDenominator() const {
		return q_;
	}

	Rational& operator=(const Rational& rhs);
	Rational& operator=(double rhs);

	// overloading -
	Rational operator-() {
		Rational temp(-p_, q_);
		return temp;
	}

	// overloading ==
	int operator==(const Rational& rhs) const {
		return (p_ * rhs.GetDenominator() == q_ * rhs.GetNumerator());
	}

	int operator==(double ra) const {
		Rational temp(ra);
		return operator==(temp);
	}

	// overloading !=
	int operator!=(const Rational& rhs) const {
		return !operator==(rhs);
	}

	int operator!=(double ra) const {
		Rational temp(ra);
		return !operator==(temp);
	}

	// overloading >
	int operator>(const Rational& rhs) const {
		return (p_ * rhs.GetDenominator() > q_ * rhs.GetNumerator());
	}

	int operator>(double ra) const {
		Rational temp(ra);
		return operator>(temp);
	}

	// overloading >=
	int operator>=(const Rational& rhs) {
		return (operator==(rhs) || operator>(rhs));
	}

	int operator>=(double ra) {
		return (operator==(ra) || operator>(ra));
	}

	// overloading <
	int operator<(const Rational& rhs) {
		return (p_ * rhs.GetDenominator() < q_ * rhs.GetNumerator());
	}

	int operator<(double ra) {
		Rational temp(ra);
		return operator<(temp);
	}

	// overloading <=
	int operator<=(const Rational& rhs) {
		return (operator==(rhs) || operator<(rhs));
	}

	int operator<=(double ra) {
		return (operator==(ra) || operator<(ra));
	}

	// overloading +
	Rational operator+(const Rational& rhs) const {
		Rational temp(p_*rhs.GetDenominator() + rhs.GetNumerator() * q_, rhs.GetDenominator() * q_);
		return temp;
	}

	Rational operator+(double ra) const {
		Rational temp(ra);
		return operator+(temp);
	}

	// overloading -
	Rational operator-(const Rational& rhs) const {
		Rational temp(p_*rhs.GetDenominator() - rhs.GetNumerator() * q_, rhs.GetDenominator() * q_);
		return temp;
	}

	Rational operator-(double ra) const {
		Rational temp(ra);
		return operator-(temp);
	}

	// overloading *
	Rational operator*(const Rational& rhs) const {
		Rational temp(p_ * rhs.GetNumerator(), rhs.GetDenominator() * q_);
		return temp;
	}

	Rational operator*(double ra) const {
		Rational temp(ra);
		return operator*(temp);
	}

	// overloading /
	Rational operator/(const Rational& rhs) const {
		if (rhs.GetNumerator() == 0) {
			cout << "Error: Cannot be divided by zero!\n";
			exit(1);
		}
		Rational temp(p_*rhs.GetDenominator(), rhs.GetNumerator() * q_);
		return temp;
	}

	Rational operator/(double ra) const {
		Rational temp(ra);
		return operator/(temp);
	}

	// Convert a rational to double by force
	friend double Float(Rational &r);

	friend ostream& operator<<(ostream &os, Rational & obj);
	friend ostream& operator<<(ostream &os, Rational && obj);

};


// overloading binary operators
int operator==(double ra, Rational& rhs);

int operator!=(double ra, Rational& rhs);

int operator>(double ra, Rational& rhs);

int operator>=(double ra, Rational& rhs);

int operator<(double ra, Rational& rhs);

int operator<=(double ra, Rational& rhs);

Rational operator+(double ra, Rational rhs);

Rational operator-(double ra, Rational rhs);

Rational operator*(double ra, Rational rhs);

Rational operator/(double ra, Rational rhs);

#endif
```

And corresponding implementation:
```c++
/*************************************************************************
> File Name: Rational.cpp
> Author: Grant Liu
> Mail: ymliu6899@gmail.com
> Created Time: Sat Dec  9 23:10:19 2017
************************************************************************/

#include<iostream>
#include<numeric>
#include"Rational.h"

using namespace std;

void Rational::Standardize() {
	if (q_ == 0) {
		cout << "Error: Denominator cannot be zero!\n";
		exit(1);
	}
	else if (q_<0) {
		p_ = -p_;
		q_ = -q_;
	}
}

int Rational::gcd(int m, int n) {
	int i, t;
	if (m == 0 || n == 0) {
		cout << "Error: m and n cannot be zero!\n";
		exit(1);
	}

	if (n > m || n<-m) {
		t = n;
		n = m;
		m = t;
	}
	i = m % n;
	if (i != 0)
		do {
			m = n;
			n = i;
			i = m % n;
		} while (m % n != 0);

		return ((n>0) ? n : -n);
}

Rational::Rational(double ra) {
	int base = 1;
	while (ra - (int)ra != 0) {
		ra *= 10;
		base *= 10;
	}
	p_ = ra;
	q_ = base;
}

void Rational::Reduce() {
	if (p_ == 0)
		return;
	int didr = gcd(p_, q_);
	p_ /= didr;
	q_ /= didr;
}

Rational& Rational::operator=(const Rational& rhs) {
	p_ = rhs.p_;
	q_ = rhs.q_;
	return *this;
}

Rational& Rational::operator=(double rhs) {
	Rational temp(rhs);
	// Make this point to temp, otherwise temp will get destructed automatically.
	*this = temp;
	return *this;
}

double Float(Rational &r) {
	return r.p_ / (double)r.q_;
}

ostream& operator<<(ostream &os, Rational& obj) {
	obj.Reduce();
	os << obj.GetNumerator() << "/" << obj.GetDenominator();
	return os;
}

ostream& operator<<(ostream &os, Rational&& obj) {
	obj.Reduce();
	os << obj.GetNumerator() << "/" << obj.GetDenominator();
	return os;
}

int operator==(double ra, Rational& rhs) {
	return (rhs == ra);
};

int operator!=(double ra, Rational& rhs) {
	return !(rhs == ra);
};

int operator>(double ra, Rational& rhs) {
	Rational temp(ra);
	return (ra > rhs);
};

int operator>=(double ra, Rational& rhs) {
	return (ra == rhs || ra > rhs);
};

int operator<(double ra, Rational& rhs) {
	Rational temp(ra);
	return (temp < rhs);
};

int operator<=(double ra, Rational& rhs) {
	return (ra == rhs || ra < rhs);
};

Rational operator+(double ra, Rational rhs) {
	return (ra + rhs);
}

Rational operator-(double ra, Rational rhs) {
	Rational temp(ra);
	return (temp - rhs);
}

Rational operator*(double ra, Rational rhs) {
	return (ra * rhs);
}

 Rational operator/(double ra, Rational rhs) {
	Rational temp(ra);
	return (temp / rhs);
}
```
To check out the most recent version, check [my github](https://github.com/Grant6899/Learning_Examples/tree/master/Primer/MyClass/Rational).


## Lessons

### 1. Overloading arithmetic operator

Take "+" as example, we need to cover three scenarios:
- Rational + Rational
- Rational + double
- double + Rational

First two functions should be declared as member functions in Rational class.

```c++
Rational operator+(const Rational& rhs) const;
Rational operator+(double ra) const;
```

For the third case, the first operand is not Rational, so it should be decalred outside of the class. **It could be either firend or not, depends on if public functions are enough to get the sum.** Here since we already have getters of both numerator and denominator, it's not necessary to make them friend.

### 2. gcd algorithm

The algorithm to calculate greatest common divisor(gcd) is usually called Euclidean algorithm.

pseudocode:
```
function gcd(a, b)
    if b = 0
       return a
    else
       return gcd(b, a mod b)
```


### 3. rvalue version of << overloading

In the main function, if you call something like this:
```c++
cout << "r0 + r1 = " << (r0 + r1) << endl;
```

You must define 
```c++
ostream& operator<<(ostream &os, Rational&& obj);
```

The reason is "(r0 + r1)" is a rvalue so it will look for << overlading with rvalue as argument to output.

According to [cppreference](http://en.cppreference.com/w/cpp/language/value_category):

**A function call or an overloaded operator expression, whose return type is non-reference, such as str.substr(1, 2), str1 + str2, or it++** is one of rvalue types.

As for accepting argument:

(1). Const lvalue reference is the most powerful parameter style, which can accept lvalue, rvalue, const lvalue, const rvalue.

(2). Normal lvalue reference cannot take rvalue. This is why 
```c++
ostream& operator<<(ostream &os, Rational& obj);
```
will not work if you have cout << r1 + r2.

(3). For constructors, compiler will generate them automatically for different scenarios. More details check [the reference](http://en.cppreference.com/w/cpp/language/move_constructor).
    

