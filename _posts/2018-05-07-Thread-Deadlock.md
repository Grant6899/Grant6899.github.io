---
layout:		post
title:		Thread Deadlock and Deadlock Prevention & Avoidance
subtitle:
date:		2018-05-07
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Multi-thread
---

## Deadlock Characteristics

- Mutual Exclusion
	
	A mutual exclusion(mutex) is a program object that prevents simultaneous access to a shared resource.
- Hold and Wait
	
    A thread is holding at least one resource and waiting for resources.
- No preemption
	
    A resource cannot be taken from a thread unless the thread releases the resource.
- Circular wait

	A set of threads are waiting for each other in circular form.
    
## Deadlock Prevention

### Eliminate Mutual Exclusion
It's impossible to dis-satisfy the mutual exclusion because some resources, such as the tap drive and printer, are inherently non-shareable.

### Eliminate Hold and Wait
1. Allocate all required resources to the thread before start of its execution, this way hold and wait condition is eliminated but **it will lead to low device utilization**. For example, if a thread requires printer at a later time and we have allocated printer before the start of its execution printer will remained blocked till it has completed its execution.
2. Thread will make new request for resources **only after** releasing the current set of resources.

### Eliminate No Preemption

Preempt resources from thread when resources required by other high priority thread.

### Eliminate Circular Wait

Each resource will be assigned with a numerical number. A thread can request for the resource only in increasing order of numbering.

For example, if T1 is allocated R5, now next time if T1 asks for R3, R4 lesser than R5, such request will not be granted. Only request for resources more than R5 will be granted.

## Deadlock Avoidance

### Banker's Algorithm

Banker's Algorithm is resources allocation and deadlock avoidance algorithm which test all the request made by threads for resources, it checks for safe state. If after granting request system remains in the safe state, it allows the request and if there's no safe state, it won't allow the request made by the thread.


#### Inputs to Banker's Algorithm

1. Max need of resources by each thread.
2. Currently allocated resources by each thread.
3. Max free available resources in the system.

#### Request will only be granted under below condition:
1. If request made by thread is less than or equal to max need to that thread.
2. If request made by is less than or equal to freely available resource in the system.


#### Example
	Total resouces in system:
    A B C D
    6 5 7 6
    
    Available system resources are:
    A B C D 
	3 1 1 2
    
    Threads(currently allocated resources):
        A B C D
    T1  1 2 2 1
	T2  1 0 3 3
	T3  1 2 1 0
    
    Threads(maximum resources):
        A B C D
    T1  3 3 2 2
	T2  1 2 3 4
	T3  1 3 5 0
    
    Threads(need resources):
        A B C D
    T1  2 1 0 1
	T2  0 2 0 1
	T3  0 1 4 0