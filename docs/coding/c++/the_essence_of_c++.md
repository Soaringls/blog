---
title: Bjarne Strooustrup:The Essence of C++
date: 2020-10-25 16:11:12
mathjax: true
categories:
  - coding
---
This page is the note of Bjarne's speech at Edinburgh from youtobe.
<!-- more -->
## What did/do I want?
Type safety
- Encapsulate necessary unsafe operations

Resource safety
- it's not all memory

Performance
- For some parts of almost all systems,it's important

Predictability
- For hard and soft real time

Teachability
- Complexity of code should be proportional to the complexity of the task

Readability
- People and machines("analyzability")
## Who did/do I want it for?
Primary concerns
- Systems propramming
- Embedded systems
- Resource constrained systems
- Large systems

Experts
- "c++ is expert friendly"

Novices
- "c++ is not just expert friendly"

## C++ in two lines
###  What is C++?
Direct map to hardware
- of instructions and dundamental data types
- Initially from C

Zero-Overhead abstraction
- Classes with constructors and destructors,inheritance,generic programming,functional programming techniques
- Initially from Simula

Much of the inspiration came from operating systems
What does C++ wants to be when it grows up?
- See above
- And better at it for more modern hardware and techniques
- Compatibility/stability is a feature

## Cpp is a light-weight abstraction programming language
building and using efficient and elegant abstractions
## Key strengths:
- software infrastructure
- resource-constrained applications
### Resource Management
#### A resource is something that must be acquired and latter released
- Explicitly or implicitly
- Resource management should not be manual
  - we don't want leaks(泄露)

#### A resource should have an owner
- Usually a "handle"
- A "handle" should present a well-defined and useful abstraction

#### All the standard-library containers manage their elements
- vector
- list,forward_list(singly-linked list),...
- map,unordered_map(hash table),...
- set,multi_set,...
- string

#### Other standard-library classes manage other resoures
- Not just memory(Garbage collection is not sufficient)
- thread,lock_guard,...
- istream,fstream,...
- unique_ptr,shared_ptr,...

#### Use constructors and destructor
```cpp
template<typename T>
class Vector {
 public:
  Vector(initializer_list<T>);//vector of elements of type T
  ~Vector();
  //...
 private:
  T* elem; //pointer to elements
  int sz;  //number of  elements
};

void fct(){
    Vector<double> vd{1, 1.625, 3.14, 2.998e8};
    Vector<string> vs{"Strachey", "Richards"};
}
```
#### Pointer Misuse
>Many(most) uses of pointers in local scope are not exception safe
```cpp
void f(int n, int x){
    Gadget* p = new Gadget(n);
    //...
    if(x < 100) throw std::runtime_error("Weird!");//leak
    if(x < 200) return;                            //leak
    //...
    delete p; //I want my garbage collector!
}
```
```
But garbage collection would not release non-memory resources
Why use a "naked" pointer?
```
#### Resource Handles and Pointers
>A std::shared_ptr releases its object at when the last shared_ptr to it is destroyed
```cpp
void f(int n, int x){
    auto p = make_shared<Gadget>(n); //manage that pointer!
    //...
    if(x < 100) throw std::runtime_error("Weird!");//no leak
    if(x < 200) return;                            //no leak
    //...
}
```
**shared_ptr** provides a form of garbage collection(But I'm not sharing anything!)
>A **std::unique_ptr** releases its object at when it goes out of scope
```cpp
void f(int n, int x){
    auto p = make_unique<Gadget>(n); //manage that pointer!
    //...
    if(x < 100) throw std::runtime_error("Weird!");//no leak
    if(x < 200) return;                            //no leak
    //...
}
```
This is simple and cheap
No more expensive than a "plain old pointer"
#### Error Handling and Resources

"Resource Acquistion Is Intialization"(RAII)
1. Acquire during construction
2. Release in destructor

Throw exception in case of failure
>In particular, throw is a constructor cannot construct and object

Never throw while holding a resource **not** owned by a handle
>Never leak

In general
>Leave established invariants intact when leaving a scope

## Why do we use pointers?
And references,iterators,etc.
To represent ownership

- **Don't Stop!** Instead, use handles

To reference resources

- from within a handle

To represent positions

- Be careful

To pass large amounts of data(into a function)

- E.g. pass by **const** reference

To return large amount of data(out of a function)

- Don't! Instead use move operations
## How to get a lot of data cheaply out of a function?
Consider

- factory functions
- functions returning lots of objects

Return a pointer to a **new**'d object?

- M* operator+(const M&, const M&);
- M* pm = m1 + m2;  //ugly: and who does the delete?
- M* q = *pm + m3;  //ugly: and who does the delete?

Return a reference to a **new**'d object?

- M& operator+(const M&, const M&);
- M m = m1 + m2; //looks OK; but who does the delete? delete what?

Pass a target object?

- void operator+(const M&, const M&, M& result);
- M m;
- operator+(m1, m2, m); //ugly: We are regressing(退化,回归) towards assembly code(汇编代码)

**conslusion**
- **Consider**
factory functions
functions returning lots of objects(in containers)
- **Return an object!**
`M operator+(const M&, const M&);`
How? Becase copies are expensive
Tricks to avoid copying are brittle
Tricks to avoid copying are not general
- **Return a handle**
sample and cheap
### Move semantics
- Direct support in C++11:Move constructor
  ```cpp
  class  Matrix{
    Representation rep;
    //...
    Matrix(Matrix&& a) //move constructor
    {
      rep = a.rep; //*this gets a's elements
      a.rep = {};  //a becomes the empty Matrix
    }
  };
  Matrix res = a + b;
  ```
- Often, you can avoid writing copy and move operations
Easily avoid
  ```cpp
  class Matrix{
    vector<double> elem; //elements here
    //...matrix access...
  };
  ```
- Matrix just "inherit" resource management from vector
- Copy and a move operations can often be implicitly generated from members
Good copy and move operations, e.g from the standard library
