---
layout: post
title: "observation: maintaining self pointers in the boost way"
description: ""
category: 
tags: []
---
{% include JB/setup %}

<b>A novice programmer once asked a master programmer,
"Oh venerable master,  can an instance of a class  keep a shared_ptr to itself?
If so will that class be  ever deleted? Why would somebody want to do something like that?"

The master smiled and the following explanation happened.</b>



### Experiment-1, dumb design and leaky code
Consider two classes  <code>Engine</code> and <code>Renderer</code>.
Design-wise they can be related by composition.
<code>Engine</code> should have a <code>Renderer</code>.
Since we are speaking of shared_pointer-s, an instance of <code>Engine</code> should maintain
a <code>shared_pointer</code> to the instance of <code>Renderer</code> that it encompasses.
Also, the instance of  <code>Renderer</code> needs to have a back-pointer to the engine that houses it,
because the <code>Renderer</code> need access to a number of characteristics of the engine
(viz: current camera, current scene, texture manager etc).
If the <code>Renderer</code> implements the back pointer via a <code>shared_pointer</code> to the <code>Engine</code>,
then we have a dead lock of destroying both the <code>Engine</code> and the <code>Renderer</code>. <code>Engine</code> has a <code>shared_ptr</code> to the <code>Renderer</code>
and <code>Renderer</code> has a <code>shared_ptr</code> to the <code>Engine</code>.


<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
using namespace std;
using namespace boost;
namespace
{
    namespace Experiment1
    {
        class R;
	class E
	{
	    public:
	    E( const std::string &amp;data):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "E:constructorn" );
	    }
	    virtual ~E()
	    {
	        _RPT0(_CRT_WARN, "E:destructorn" );
	    }
	    void SetRenderer(
                shared_ptr &lt;R&gt; &amp; renderer
            )
	    {
	        m_renderer = renderer;
	    }
	    protected:
	    const std::string m_data;
	    shared_ptr &lt;R&gt; m_renderer;
	};


    class R
        {
            public:
	    R(
	        const std::string &amp;data
	    ):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "R:constructorn" );
	    }
	    virtual ~R()
	    {
	         _RPT0(_CRT_WARN, "R:destructorn" );
	    }
	    void RegisterEngine(
                shared_ptr &lt;E&gt; &amp; engine
            )
	    {
	        m_engine = engine;
	    }
	    protected:
	    const std::string m_data;
	    shared_ptr &lt;E&gt; m_engine;
    };
}
</pre>

If the above two classes are used in the following manner,
<code>Engine</code> has a <code>shared_ptr</code> to the <code>Renderer</code> and <code>Renderer</code> has a <code>shared_ptr</code> to the <code>Engine</code>.
Consequently the destructors of the  instances pointed to by <code>sR</code> and <code>sE</code> wont be called,
which results in a memory leak.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
/*
  namespace Experiment1
{
   // E and R classes defined as above
}
*/
using namespace Experiment1;
bool test()
{
    //sR and sE refer to each other mutually
    //consequently they are not deleted
    //which results in memory leak
    _CrtSetDbgFlag (
    _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF
    );
    shared_ptr sE( new E("theEngine") );
    shared_ptr sR( new R("cartoonRenderer"));
    sE-&gt;SetRenderer( sR );
    sR-&gt;RegisterEngine( sE );
    return true;
}
</pre>


### Experiment2, leak proof code, but dumb design

To fix the memory leaks, R should hold a weak pointer to E.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
#include &lt;boost/weak_ptr.hpp&gt;
using namespace std;
using namespace boost;
namespace
{
    namespace Experiment2
    {
        class R;
	class E
	{
	    public:
	    E( const std::string &amp;data):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "E:constructorn" );
	    }
	    virtual ~E()
	    {
	        _RPT0(_CRT_WARN, "E:destructorn" );
	    }
	    void SetRenderer(
                shared_ptr &lt;R&gt; &amp; renderer
            )
	    {
	        m_renderer = renderer;
	    }
	    protected:
	    const std::string m_data;
	    shared_ptr &lt;R&gt; m_renderer;
	};


    class R
        {
            public:
	    R(
	        const std::string &amp;data
	    ):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "R:constructorn" );
	    }
	    virtual ~R()
	    {
	         _RPT0(_CRT_WARN, "R:destructorn" );
	    }
	    void RegisterEngine(
                shared_ptr &lt;E&gt; &amp; engine
            )
	    {
	        m_engine = engine;
	    }
	    protected:
	    const std::string m_data;
	    weak_ptr &lt;E&gt; m_engine;
    };

}
</pre>

The following use of  the <code>Engine</code> and <code>Renderer</code> wont cause any memory-leaks.
But it has a design problem as explained below.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
/*
  namespace Experiment2
{
   // E and R classes defined as above
}
*/
using namespace Experiment2;
bool test()
{
    //Memory leak free
    //But not a good design.
    _CrtSetDbgFlag (
    _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF
    );
    shared_ptr sE( new E("theEngine") );
    shared_ptr sR( new R("cartoonRenderer"));
    sE-&gt;SetRenderer( sR );
    sR-&gt;RegisterEngine( sE );
    return true;
}
</pre>


### Experiment3, smart design, but leaky code
The requirement that the user has to explicitly have <code>Renderer</code> call
<code>RegisterEngine</code> after the <code>Engine</code> calls <code>SetRenderer</code> is called, is not a sound design.
<code>Engine::SetRenderer</code> should automatically call <code>Renderer::RegisterEngine</code>.
But inside the code of <code>Engine::SetRenderer</code>, for  <code>Engine</code> to call <code>Renderer::RegisterEngine</code>,
how can the <code>Engine</code> pass in a <code>shared_ptr</code> or <code>weak_ptr</code> of itself to  <code>Renderer::RegisterEngine</code>?
How about every  instance of <code>Engine</code>, keeping a self <code>shared_ptr</code> of itself?  <b>This is a perfect example
of the necessity  of keeping a self <code>shared_ptr</code> </b>.


<b>
The novice programmer nodded in perfect agreement. But the master continued.</b>
<p>
But the  concept of <code>shared_ptr</code> is that each instance of a class
subjected to <code>shared_ptr</code>-ing should have only one shared reference structure.
If such a self shared_ptr is kept, then we should have any other <code>shared_ptr</code> of <code>Engine</code>
initialized by copy-constructing or assignment from the self <code>shared_ptr</code>
stored in <code>Engine</code>.
Otherwise we should have the same instance of <code>Engine</code>,
referenced by two or more <code>shared_ptr</code>s of <code>Engine</code> who don't share the same  reference count structure.
Consequently we should 'private'-ise the constructor of <code>Engine</code>, and have <code>Engine</code>
created through  a <code>Create</code> function.

This will again result in memory leak, since if you  blindly keep a self shared_ptr to the class itself,
the class-s destructor will never be called. Consequently <code>Engine</code>'s destructor will never be called.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
#include &lt;boost/weak_ptr.hpp&gt;
using namespace std;
using namespace boost;
namespace
{
    namespace Experiment3
    {
        class R;
	class E
	{
	    public:
            shared_ptr &amp;Create(
                 const std::string &amp;i_data
            )
            {
                  E *pE = new E(i_data);
                  return pE-&gt;m_self;
            }
	    virtual ~E()
	    {
	        _RPT0(_CRT_WARN, "E:destructorn" );
	    }
	    void SetRenderer(
                shared_ptr &lt;R&gt; &amp; renderer
            );
            protected:
	    E( const std::string &amp;data):m_data( data)
	    {
                m_self.reset( this );
	        _RPT0(_CRT_WARN, "E:constructorn" );
	    }
	    protected:
	    const std::string m_data;
	    shared_ptr &lt;R&gt; m_renderer;
            shared_ptr &lt;E&gt; m_self;
	};


    class R
        {
            public:
	    R(
	        const std::string &amp;data
	    ):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "R:constructorn" );
	    }
	    virtual ~R()
	    {
	         _RPT0(_CRT_WARN, "R:destructorn" );
	    }
	    void RegisterEngine(
                shared_ptr &lt;E&gt; &amp; engine
            );
	    protected:
	    const std::string m_data;
	    weak_ptr &lt;E&gt; m_engine;
    };



    void E::SetRenderer(
        shared_ptr &lt;R&gt; &amp; renderer
     )
    {
        m_renderer = renderer;
        m_renderer-&gt;RegisterEngine( m_self );
    }

    void R::RegisterEngine(
        shared_ptr &lt;E&gt; &amp; engine
     )
    {
	m_engine = engine;
    }
}
</pre>

The following use of te above design will still result in memory leak, since E is never deleted.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
/*
  namespace Experiment3
{
   // E and R classes defined as above
}
*/
using namespace Experiment3;
bool test()
{
   //A sound design but, instance pointed
   //to by SE is never deleted. So memory leaks!
    _CrtSetDbgFlag (
    _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF
    );
    shared_ptr sE( E::Create("theEngine") );
    shared_ptr sR( new R("cartoonRenderer"));
    sE-&gt;SetRenderer( sR );
    return true;
}
</pre>

=== Experiment4, smart design and leak proof code
The correct way to do this is using a <code>boost::enable_shared_from_this</code> construct.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
#include &lt;boost/weak_ptr.hpp&gt;
#include &lt;boost/enable_shared_from_this.hpp&gt;
using namespace std;
using namespace boost;
namespace
{
    namespace Experiment4
    {
        class R;
	class E :public enable_shared_from_this&lt;E&gt;
	{
	    public:
            shared_ptr &amp;Create(
                 const std::string &amp;i_data
            )
            {
                  shared_ptr temp( new E(i_data) );
                  return temp-&gt;shared_from_this();
            }
	    virtual ~E()
	    {
	        _RPT0(_CRT_WARN, "E:destructorn" );
	    }
	    void SetRenderer(
                shared_ptr &lt;R&gt; &amp; renderer
            );
            protected:
	    E( const std::string &amp;data):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "E:constructorn" );
	    }
	    protected:
	    const std::string m_data;
	    shared_ptr &lt;R&gt; m_renderer;
            //no need of maintaining a self shared_ptr
            //shared_ptr &lt;E&gt; m_self;
	};


    class R
        {
            public:
	    R(
	        const std::string &amp;data
	    ):m_data( data)
	    {
	        _RPT0(_CRT_WARN, "R:constructorn" );
	    }
	    virtual ~R()
	    {
	         _RPT0(_CRT_WARN, "R:destructorn" );
	    }
	    void RegisterEngine(
                shared_ptr &lt;E&gt; &amp; engine
            );
	    protected:
	    const std::string m_data;
	    weak_ptr &lt;E&gt; m_engine;
    };



    void E::SetRenderer(
        shared_ptr &lt;R&gt; &amp; renderer
     )
    {
        m_renderer = renderer;
        m_renderer-&gt;RegisterEngine( m_self );
    }

    void R::RegisterEngine(
        shared_ptr &lt;E&gt; &amp; engine
     )
    {
	m_engine = engine;
    }
}
</pre>

This is used as below.

<pre>
#include &lt;cstdlib&gt;
#include &lt;crtdbg.h&gt;
#include &lt;string&gt;
#include &lt;boost/shared_ptr.hpp&gt;
/*
  namespace Experiment4
{
   // E and R classes defined as above
}
*/
using namespace Experiment4;
bool test()
{
    //The correct design which doesnt result in memory leaks.
    _CrtSetDbgFlag (
    _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF
    );
    shared_ptr sE( E::Create("theEngine") );
    shared_ptr sR( new R("cartoonRenderer"));
    sE-&gt;SetRenderer( sR );
    return true;
}
</pre>