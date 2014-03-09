---
layout: post
title: "small experiments with  r value references in c++11"
description: ""
category: 
tags: [c++11, r-value references]
---
{% include JB/setup %}

This is a small experiment which I did, which shows how move constructor and r-value references in c++11
can avoid unnecessary calls to the copy constructor. I used mingw and gcc 4.7.





Let us consider a <code>class A</code>, which holds an expensive resource. For the sake of simplicity,
let us assume that the member <code>string A::sA</code> is in fact an expensive resource.


<pre>
class A {
  public:
  A(const string &sA):sA(sA){
    const char *pz = sA.c_str();
    cout &lt;&lt; "expensive constructor: " &lt;&lt; pz;
    cout &lt;&lt;  endl;
  }

  A(const A &a):sA(a.sA){
    const char *pz = a.sA.c_str();
    cout &lt;&lt; "expensive copy constructor:" &lt;&lt; pz;
    cout &lt;&lt; endl;
  }

  A & operator=(const A &other){
    sA = other.sA; 
    const char*pz = sA.c_str();
    cout &lt;&lt; "assignment operator:" &lt;&lt; pz;
    cout &lt;&lt; endl;
    return *this;
  }

  ~A(){}
  string sA; //stand in for the expensive resource
};
</pre>

Let us assume that it  is expensive to construct <code>A</code>. 
Also assume that calling copy constructor of <code>A</code> is also prohibitively expensive,
since this requires cloning the resource from the source argument.

We want to have a bunch of <code>A</code>-s created.
I tried the usual c++ style, where the target is passed in as a reference.
The following code fills the target vector with 5 <code>A</code> objects.

### Trial -1

<pre>
void bunchOfAs(vector&lt;A&gt; &tgt) {
  insert_iterator&lt;vector&lt;A&gt; &gt; ait(tgt, tgt.end());
  cout &lt;&lt; "start filling vector" &lt;&lt; endl;
  for(int i=0; i &lt; 5; ++i) {
     stringstream ss;
     ss &lt;&lt; i;
    *ait++ = A(ss.str());
  }
  cout &lt;&lt; "end filling vector" &lt;&lt; endl;
}
</pre>

This was called as,

<pre>
vector&lt;A&gt; myCollection;
bunchOfAs(myCollection);
</pre>

This resulted in the following output.

<pre>
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
</pre>

The expensive constructors associated with the creation of the 5 <code>A</code>-s cannot be avoided.
No worries there. But I was amazed to find such an innocent code resulting in this many number of copy constructors.


Part of the reason is that, the capacity of the vector myCollection is initially zero,
and as we add items to the end, the capacity increases to accommodate more.
When the capacity increases, this results in re-allocation of storage,
which results in copying already created A-s to a fresh location.
This re-allocation and copying repeats for a few times.

### Trial 2
So this time I reserved enough storage for myCollection to begin with, as in,

<pre>
vector&lt;A&gt; myCollection;
myCollection.reserve(5);
bunchOfAs(myCollection);
</pre>

which resulted in the following output.

<pre>
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
</pre>

This is much better. Still, for each <code>A</code> which is constructed,
we call another copy constructor also.
How can we eliminate this unnecessary call to the copy constructor?
If we go back, our original code to make a bunch of <code>A</code>-s was

<pre>
void bunchOfAs(vector&lt;A&gt; &amp;tgt) {
  insert_iterator&lt;vector&lt;a&gt; &gt; ait(tgt, tgt.end());
  cout &lt;&lt; "start filling vector" &lt;&lt; endl;
  for(int i=0; i &lt; 5; ++i) {
    stringsteam ss;
    ss &lt;&lt; i;
    *ait++ = A(ss.str());
  }
  cout &lt;&lt; "end filling vector" &lt;&lt; endl;
}
</pre>


Let us take a look at the line <code> *ait++ = A(ss.str()); </code>.
(Ignore for the moment the <code>stringstream</code> that is used).
On the right hand side, <code>A</code> is constructed in an r-value,
and it is then copied to the next location pointed by the insert iterator.

### Trial-3

To eliminate the unnecessary call to the copy constructor,
I amended the definition of <code>class A</code> with a move constructor also, as in.

<pre>
class A {
  public:
  A(const string &amp;sA):sA(sA){
    const char *pz = sA.c_str();
    cout &lt;&lt; "expensive constructor: " &lt;&lt; pz;
    cout &lt;&lt;  endl;
  }

  A(A&amp;&amp; a):sA(std::move(a.sA)){
    const char *pz = sA.c_str();
    cout &lt;&lt; "cheap move constructor:" &lt;&lt; pz;
    cout &lt;&lt; endl;
  }

  A(const A &amp;a):sA(a.sA){
    const char *pz = a.sA.c_str();
    cout &lt;&lt; "expensive copy constructor:" &lt;&lt; pz;
    cout &lt;&lt; endl;
  }

  A &amp; operator=(const A &amp;other){
    sA = other.sA;
    const char*pz = sA.c_str();
    cout &lt;&lt; "assignment operator:" &lt;&lt; pz;
    cout &lt;&lt; endl;
    return *this;
  }

  ~A(){}
  string sA; //stand in for expensive resource
}
</pre>

I called this as follows,

<pre>
vector&lt;A&gt; myCollection;
myCollection.reserve(5);
bunchOfAs(myCollection);
</pre>

which resulted in the following output.

<pre>
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
</pre>

### Trial-4
Since modern compilers (I used g++4.7) are good at return value optimization,
I could simplify this code as follows without any additional calls to the copy constructor.

<pre>
vector&lt;A&gt; bunchOfAs() {
  vector&lt;A&gt; tgt;
  tgt.reserve(5);
  insert_iterator&lt;vector&lt;A&gt; &gt; ait(tgt, tgt.end());
  cout &lt;&lt; "start filling vector" &lt;&lt; endl;
  for(int i=0; i &lt; 5; ++i) {
    stringstream ss;
    ss &lt;&lt; i;
    *ait++ = A(ss.str());
  }
  cout &lt;&lt; "end filling vector" &lt;&lt; endl;
  return tgt;
}
</pre>

I invoke this as

<pre>
vector&lt;A&gt; myCollection = bunchOfAs();
</pre>
