---
layout:		post
title:		Lambda Function
subtitle:
date:		2016-12-20
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Lambda expression
---

Lambda function: Replacement for global functions or function objects(functors), can generate unnamed function objects.

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

  Variables captured by value, can't be changed because the lambda function is const in the generated function object. To make the   lambda function non-const, define it as mutable. The original variable stays unchanged, but within the function body, the captured variable can be modified.

- Capture-default in class
  |Syntax     |         Description |
  |:--------: |:------------------- |
  |[this]| captures only the this pointer|
  |[=]|captures this pointer + local variable by value|
  |[&]|captures this pointer + local variable by reference|


Inside lambda function can access member by:
this->member or member.

Sample code:
```c++
// Class to calculate the sine value of a vector with input values.
class CalculateSine {
private: 
	double m_amplitude; 
public:
	CalculateSine(double amplitude): m_amplitude(amplitude) {}
	// Calculate sine.
	vector<double> Calculate(vector<double>& input) {
	vector<double> result(input.size()); // Create result vector.
	// Transform the input vector using a lambda function.
	// The lambda function must capture the m_amplitude member variable.
	// [m_amplitude] or [this->m_amplitude] does not work.
	// Use [this] instead. Using [=] or [&] will also capture the this pointer. 
    // Now you can use the members implicitly(m_amplitude)
	// or explicitly (this->m_amplitude).
	transform(input.begin(), input.end(), result.begin(), 
	[this](double v) -> double
	{ return m_amplitude*sin(6.28319*v/360.0); } );
return result; // Return the result. }
};
```
