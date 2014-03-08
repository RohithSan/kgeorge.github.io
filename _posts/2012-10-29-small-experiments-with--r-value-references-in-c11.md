---
layout: post
title: "small experiments with  r value references in c++11"
description: ""
category: 
tags: []
---
{% include JB/setup %}

This is a small experiment which I did, which shows how move constructor and r-value references in c++11
can avoid unnecessary calls to the copy constructor. I used mingw and gcc 4.7.





Let us consider a <code>class A</code>, which holds an expensive resource. For the sake of simplicity,
let us assume that the member <code>string A::sA</code> is in fact an expensive resource.

```c++
class A {
  public:
  A(const string &sA):sA(sA){
    const char *pz = sA.c_str();
    cout << "expensive constructor: " << pz;
    cout <<  endl;
  }

  A(const A &a):sA(a.sA){
    const char *pz = a.sA.c_str();
    cout << "expensive copy constructor:" << pz;
    cout << endl;
  }

  A & operator=(const A &other){
    sA = other.sA; 
    const char*pz = sA.c_str();
    cout << "assignment operator:" << pz;
    cout << endl;
    return *this;
  }

  ~A(){}
  string sA; //stand in for the expensive resource
};
```
Let us assume that it  is expensive to construct <code>A</code>. 
Also assume that calling copy constructor of <code>A</code> is also prohibitively expensive,
since this requires cloning the resource from the source argument.

We want to have a bunch of <code>A</code>-s created.
I tried the usual c++ style, where the target is passed in as a reference.
The following code fills the target vector with 5 <code>A</code> objects.

### Trial -1

```c++
void bunchOfAs(vector<A> &tgt) {
  insert_iterator<vector<A> > ait(tgt, tgt.end());
  cout << "start filling vector" << endl;
  for(int i=0; i < 5; ++i) {
     stringstream ss;
     ss << i;
    *ait++ = A(ss.str());
  }
  cout << "end filling vector" << endl;
}
```

This was called as,

```c++
vector<A> myCollection;
bunchOfAs(myCollection);
```

This resulted in the following output.

```c++
start filling vector
expensive constructor: 0
expensive copy constructor:0
expensive constructor: 1
expensive copy constructor:1
expensive copy constructor:0
expensive constructor: 2
expensive copy constructor:2
expensive copy constructor:0
expensive copy constructor:1
expensive constructor: 3
expensive copy constructor:3
expensive constructor: 4
expensive copy constructor:4
expensive copy constructor:0
expensive copy constructor:1
expensive copy constructor:2
expensive copy constructor:3
end filling vector
```

The expensive constructors associated with the creation of the 5 <code>A</code>-s cannot be avoided.
No worries there. But I was amazed to find such an innocent code resulting in this many number of copy constructors.


Part of the reason is that, the capacity of the vector myCollection is initially zero,
and as we add items to the end, the capacity increases to accommodate more.
When the capacity increases, this results in re-allocation of storage,
which results in copying already created A-s to a fresh location.
This re-allocation and copying repeats for a few times.

### Trial 2
So this time I reserved enough storage for myCollection to begin with, as in,

```c++
vector<A> myCollection;
myCollection.reserve(5);
bunchOfAs(myCollection);
```

which resulted in the following output.

```
start filling vector
expensive constructor: 0
expensive copy constructor:0
expensive constructor: 1
expensive copy constructor:1
expensive constructor: 2
expensive copy constructor:2
expensive constructor: 3
expensive copy constructor:3
expensive constructor: 4
expensive copy constructor:4
end filling vector
```

This is much better. Still, for each <code>A</code> which is constructed,
we call another copy constructor also.
How can we eliminate this unnecessary call to the copy constructor?
If we go back, our original code to make a bunch of <code>A</code>-s was

```c++
void bunchOfAs(vector<A> &amp;tgt) {
  insert_iterator<vector<a> > ait(tgt, tgt.end());
  cout << "start filling vector" << endl;
  for(int i=0; i < 5; ++i) {
    stringsteam ss;
    ss << i;
    *ait++ = A(ss.str());
  }
  cout << "end filling vector" << endl;
}
```


Let us take a look at the line <code> *ait++ = A(ss.str()); </code>.
(Ignore for the moment the <code>stringstream</code> that is used).
On the right hand side, <code>A</code> is constructed in an r-value,
and it is then copied to the next location pointed by the insert iterator.

### Trial-3

To eliminate the unnecessary call to the copy constructor,
I amended the definition of <code>class A</code> with a move constructor also, as in.

```c++
class A {
  public:
  A(const string &amp;sA):sA(sA){
    const char *pz = sA.c_str();
    cout << "expensive constructor: " << pz;
    cout <<  endl;
  }

  A(A&amp;&amp; a):sA(std::move(a.sA)){
    const char *pz = sA.c_str();
    cout << "cheap move constructor:" << pz;
    cout << endl;
  }

  A(const A &amp;a):sA(a.sA){
    const char *pz = a.sA.c_str();
    cout << "expensive copy constructor:" << pz;
    cout << endl;
  }

  A &amp; operator=(const A &amp;other){
    sA = other.sA;
    const char*pz = sA.c_str();
    cout << "assignment operator:" << pz;
    cout << endl;
    return *this;
  }

  ~A(){}
  string sA; //stand in for expensive resource
}
```

I called this as follows,

```c++
vector<A> myCollection;
myCollection.reserve(5);
bunchOfAs(myCollection);
```

which resulted in the following output.

```
start filling vector
expensive constructor: 0
cheap move constructor:0
expensive constructor: 1
cheap move constructor:1
expensive constructor: 2
cheap move constructor:2
expensive constructor: 3
cheap move constructor:3
expensive constructor: 4
cheap move constructor:4
end filling vector
```

### Trial-4
Since modern compilers (I used g++4.7) are good at return value optimization,
I could simplify this code as follows without any additional calls to the copy constructor.

```c++
vector<A> bunchOfAs() {
  vector<A> tgt;
  tgt.reserve(5);
  insert_iterator<vector<A> > ait(tgt, tgt.end());
  cout << "start filling vector" << endl;
  for(int i=0; i < 5; ++i) {
    stringstream ss;
    ss << i;
    *ait++ = A(ss.str());
  }
  cout << "end filling vector" << endl;
  return tgt;
}
```

I invoke this as

```
vector<A> myCollection = bunchOfAs();
```
