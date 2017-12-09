---
layout:		post
title:		Storage Duration
subtitle:
date:		2017-12-09
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - c++
    - Storage Duration
---


## Scope and Linkage

***Scope*** describes how widely visible a name is in a file.

***Linkage*** describes how a name can be shared in different units. 
- A name with ***external linkage*** can be shared across files.
- A name with ***internal linkage*** can be shared by functions with in a single file. 
- Names of automatic varialbes have no linkage because they are not shared.


## Storage Duration

C++ uses three separate schemes (four under C++11) for storing data:

### Automatic storage duration(stored on stack)

Varialbes declared inside a function definition-including function parameters-have automatic storage duration.

Variable is allocated when program execution enters the innermost black containing the definition and each fades from existence when execution leaves that block.

```c++
int main(){
	int teledeli = 5;
    {						// websight allocated
    	cout << "Hello\n";
        int websight = -2;	// websight scope begins
        cout << wibsight << ' ' << teledeli << endl;
        
        int teledeli = 1;   // legal declaration, original teledeli will be hidden
    }						// websight and inner teledeli expires
    cout << teledeli << endl;
    //...
}   // teledeli expires
```

#### Keyword "auto" and "register"

- In C++11, the keyword auto is used for automatic type deduction.
- In C and in prior versions of C++, auto is used to explicitly identify a variable as having automatic storage.
```c++
auto int x = 1;
```

- In C++11, "register" is deprecated and can only explicitly indicate the variable is automatic storage duration.
- In C and in prior versions of C++, register suggests that the complier use a CPUregister to store an automatic variable to allow faster access to the variable.

### Static storage duration

Varialbes defined outside a function definition or else by using keyword "static" have static storage duration. They persist for the entire time a program is running.

The complier allocates a fixed block of memory to hold all the static variables, and those variables stay present as long as the program executes.

If there is no explicit initialization, the complier will set it to 0 by default.

```c++
//...
int global = 1000;			// static duration, external linkage
static int one_file = 50;	// static duration, internal linkage

int main(){
	//...
}

void funct1(int n){
	static int count = 0;	// static duration, no linkage
    int llama = 0;
}

void funct2(int q){
//...
}
```

The variable count has local scope and no linkage, which can only be used inside funct1(). Unlinke llama, count remains in memory even when the funct1() is not being executed.

We call variable like global ***external variable*** or ***global variable***.

#### One Definition Rule and Keyword "extern"

C++ only allows one definition of a variable. To satisfy the requirement, C++ has two kinds of variable declarations.

- ***defining declaration***

	Causes storage for the variable to be allocated.

- ***referencing declaration***
	
    Does not cause storage to be allocated because it refers to an existing variable.
    
Examples:
```c++
double up;				// definition
extern int blem;		// referencing declaration, blem defined elsewhere
extern char gr = 'z';   // definition because of initialization
```

If you use an external variable in several files, only one file can contain a definition for that varaible. But every other file using the variable needs to declare that variable using the keyword **extern**.

#### Five Kinds of Variable Storage

| **Storage Description** | **Duration**  | **Scope**| **Linkage** | **How Declared** |
|:---:|:-:|:-:|:-:|:-|
| Automatic   | Automatic | Block | None | In a block |
| Register    | Automatic | Block | None | In a block with the keyword "register" |
| Static with no linkage | Static | Block | None | In a block with the keyword "static" |
| Static with external linkage | Static | File | External | Outside all functions |
| Static with internal linkage | Static | File | Internal | Outside all functions with the keyword "static" |


### Dynamic Storage Duration

Memory  allocated by the new operator persists until it is freed with the delete operator or until the program ends, whichever comes first. This memory has dynamic storage duration and sometimes is termed the free store or the heap.

### Thread Storage Duration(C++11)
Variables declared with the "thread_local" keyword have storage that persists for as long as the containing thread lasts.













