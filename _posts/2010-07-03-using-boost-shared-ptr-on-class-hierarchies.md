---
layout: post
title: "Using boost::shared_ptr on class hierarchies"
description: ""
category: 
tags: [generic programming, boost, shared_ptr, object oriented programming]
---
{% include JB/setup %}

<code>boost::shared_ptr</code> has become one of the most versatile smart_pointer structures used in C++.
If you consistently use <code>shared_ptr</code> to refer to objects,
you don't have to worry when to destroy them.
We would like to see how <code>boost::shared_ptr</code> can be used amidst up-casting and down-casting across class hierarchies.



Consider the following class.

<pre>
    class A
     {
     public:
             A():m_i(0){}
             virtual ~A(){}
             int m_i;
     };
</pre>

If we always use <code>boost::shared_ptr</code>-s to refer to an instance of A, we
can use them across scopes, worry free of when to destroy them as shown.

<pre>
    using namespace boost;

    shared_ptr  &lt;A&gt;   sA1;
    {
         shared_ptr &lt;A&gt; sA2( new A);
         sA1 = sA2;
         //sA1 and sA2 shares the same pointer to A
         //and the same reference counting structure
         assert( sA1.use_count() == sA2.use_count() );
         assert( sA1.use_count() == 2);
    }
    assert( sA1.use_count() == 1);
</pre>

Now, let <code>B</code>, be a derived class of <code>A</code> as shown below.

</pre>
     class B : public A
     {
     public:
             B():m_j(0){}
             ~B(){}
             int m_j;
     };
</pre>

#### shared_ptr and up-casting

If we assign a <code>shared_ptr</code> <code>B</code> to a <code>shared_ptr</code> <code>A</code>,
the two shared pointers still share the same pointer and the reference count. So <code>shared_pointer</code> respects up-casting. Please see the code below.

<pre>
    using namespace boost;

    shared_ptr  &lt;A&gt;   sA;
    {
         shared_ptr &lt;B&gt; sB( new B);
         sA = sB;
         //sA and sB shares the same pointer to B
         //and the same reference counting structure
         assert( sA.use_count() == sB.use_count() );
         assert( sA.use_count() == 2);
    }
    assert( sA.use_count() == 1);
</pre>

#### shared_ptr and down-casting

Now consider the problem of down-casting.

<pre>
    using namespace boost;

    shared_ptr  &lt;B&gt;   sB;
    {
         shared_ptr &lt;A&gt; sA( new B);
         //how can we reset sB with the pointer
         //now owned by sA.?
    }
</pre>

The following wont compile.

<pre>
    using namespace boost;

    shared_ptr  &lt;B&gt;   sB;
    {
         shared_ptr &lt;A&gt; sA( new B);
         //compilation error in the line below
         sB = sA;
    }
</pre>

Here is another wrong way, that results in run-time error.
<pre>
    using namespace boost;

    shared_ptr  &lt;B&gt;   sB;
    {
         shared_ptr &lt;A&gt; sA( new B);
         B * pB = dynamic_cast &lt;B*&gt; (sA.get());
         sB.reset( pB );
         //though sA and sB now points to the same
         //instance of B, they are reference counted
         //using different structures.
         assert( sA.use_count() == 1);
         assert( sB.use_count() ==1);
    }
    //Instance of B pointed to by sA is destroyed.
    //But hey, sB still refers to it.
    //Now, the following will cause a runtime error
    //due to double deletion
    sB.reset();
</pre>
The correct way is shown below.

<pre>
    using namespace boost;

    shared_ptr  &lt;B&gt;   sB;
    {
         shared_ptr &lt;A&gt; sA( new B);
         sB = dynamic_pointer_cast &lt;B,A&gt; ( sA ) ;
         //sA and sB now shares the same instance of B.
         //Further sA and sB shares the same reference
         //counting structure.
         assert( sA.use_count() == 2);
         assert( sB.use_count() == 2);
    }
    assert( sB.use_count() == 1);
</pre>


