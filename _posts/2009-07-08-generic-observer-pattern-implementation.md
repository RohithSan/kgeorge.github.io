---
layout: post
title: "Generic observer pattern implementation"
description: ""
category: 
tags: []
---
{% include JB/setup %}

#### Introduction
We implement a generic <a href="http://en.wikipedia.org/wiki/Observer_pattern">observer</a> pattern framework below. The observer design pattern can be employed as a core architectural element of a software package. In this pattern, objects called subscribers can arrange to have notified of any state change from an object called subject. GUI systems use this pattern to implement distributed event handling.

In our terminology the the object observed is called <em>Observed</em> and the object who wants to get notified is called a <em>Subscriber</em>. The subscription function which a <em>Subscriber</em> wants to get notified through, is housed in a lightweight class called <em>Subscription</em>.

We use a mixture of dynamic polymorphism and generic programming to connect the observer functions to the object observed. The user of this framework is hidden from much of the template complexity that happens internally. A good sample of test cases are also provided which covers most of the ways subscribers are hitched to an observed object.
<blockquote>An illustrative example can be downloaded <a href="http://kgeorge-lib.googlecode.com/files/kg_observer_071209.rar">here</a>.
<a href="http://code.google.com/p/kgeorge-lib/source/browse/#svn/trunk/kg_observer">Here</a> is another viewable version of the code.</blockquote>



Consider the following button class.
<pre>class IObserved
{
 public:
        typedef void  (*TFun) ();
        virtual ~IObserved(){}
        virtual void ClickMe()=0;
};

class ExampleButton : public IObserved
{

    public:
      typedef std::vector &lt; TFun &gt; TFunContainer;
       void ClickMe()
      {
          TFunContainer::iterator tit;
          for( tit = m_subscriptions.begin();
                tit != m_subscriptions.end();
               ++tit
              )
             {
                  TFun fun = *tit;
                  (*fun)();
             }
      }
      protected:
           TFunContainer m_subscriptions;
};</pre>
The <em>ExampleButton</em> is the Signal. When it is clicked, the GUI frame work ensures that <em>ClickMe</em> is invoked. That much is given. Now we want a few subscribers to be notified when the <em>ClickMe</em> is executed. Note that, if the signature of the function subscribed or unsubscribed is anything other than <em>void (*fun) ()</em> this will result in a compilation error. This ensures <strong>type safety</strong>.

We should be able to connect and disconnect subscribers at any part of the program.

Let us consider a subscriber, which is a global static function of signature <em>void (*fun)()</em>.

Let us modify our class to include the subscribing an unsubscribing of the observer.
<pre>class IObserved
{
    public:
        typedef void (*TFun) ();
        virtual ~IObserved(){}
        virtual void ClickMe()=0;
        virtual void Subscribe( TFun fun )=0;
        virtual void Unsubcribe( TFun fun )=0;
};

class ExampleButton : public IObserved
{
    public:
        typedef std::vector TFunContainer;
        void ClickMe()
        {
          TFunContainer::iterator tit;
          for( tit = m_subscriptions.begin();
                tit != m_subscriptions.end();
               ++tit
              )
             {
                  TFun fun = *tit;
                  (*fun)();
             }
      }

      void Subscribe( TFun fun )
      {
          m_subscriptions.push_back( fun );
      }

      void Unsubscribe( TFun fun )
      {
          TFunContainer::iterator lit;
          lit = find( m_subscriptions.begin(),
                        m_subscriptions.end(),
                        fun );
          m_subscriptions.erase( lit );
       }
  protected:
       TFunContainer  m_subscriptions;
};</pre>
We would like to generalize this.
<ul>
	<li>We should be able to add subscriptions which are either class member functions , member static functions or global static functions.</li>
	<li>The signature of the subscription functions that are added, should be general enough and should be specified by the <em>IObserved</em> interface at compile time.</li>
</ul>
<h5>Observed and abstract ISubscription-s</h5>
To achieve the above generalizations, we come up with
an interface for the subscription, which is template-ized by the
arguments and return value of the subscription function.
Based on the number of the arguments, we come up with
N different interfaces. N is limited to 3 in the current implementation.
<pre> // R is the return type of the subscription function
 // Ai is the ith argument of the subscription function

class ISubscription
{
    public:
        ISubscription(){}
        virtual ~ISubscription(){}
        void DeleteMe()
        {
             delete this;
        }
};

template &lt; typename R &gt;
class ISubscription0 : public ISubscription
{
    public:
        ISubscription0(){}
        virtual ~ISubscription0(){}
        R operator()()=0;
};

template &lt; typename R, typename A1 &gt;
class ISubscription1 : public ISubscription
{
    public:
        ISubscription1(){}
        virtual ~ISubscription1(){}
        R operator()( A1 a1 )=0;
};

template &lt; typename R, typename A1, typename A2 &gt;
class ISubscription2 : public ISubscription
{
    public:
        ISubscription2(){}
        virtual ~ISubscription2(){}
        R operator()( A1 a1, A2 a2 )=0;
};

template &lt;
       typename R,
       typename A1,
       typename A2,
       typename A3  &gt;
class ISubscription3 : public ISubscription
{
    public:
        ISubscription3(){}
        virtual ~ISubscription3(){}
        R operator()( A1 a1, A2 a2, A3 a3 )=0;
};</pre>
The <em>ISubscription-n</em> classes abstract away whether the actual subscription function is a class member function, member static function or a global static function. The user of the <em>ISubscription</em>-n subscription, which according to the above example would be a <em>ExampleButton</em> class, does not need to know
<ul>
	<li>whether the subscription was through the member function of an instance of any particular class, and if so of what class</li>
	<li>whether the subscription was through the constant member function of an instance of any particular class, and if so of what class</li>
	<li>whether the subscription was through the static member function of any particular class and if so of what class</li>
	<li>whether the subscription was through a global static function.</li>
</ul>
The <em>ExampleButton</em> class provided was a concrete example of a <em>Observed</em>. To play the role of the <em>ExampleButton</em> class, we introduce the <em>Observed</em> class. The <em>ClickMe</em> function of the <em>ExampleButton</em> is replaced by <em>Fire</em> function in the <em>Observed</em> class. We give the capability to <em>Observed</em> to decide at compile time to accept any of the above 4 <em>ISubscription</em>-s.
<pre>template &lt;
    typename R,
    typename A1=DummyType,
    typename A2=DummyType,
    typename A3=DummyType
    &gt;
class Observed : public IObserved
{
public:
/*-------------------------------------------------/
//  Determine at compile time
//  the type value of TSubscription
//------------------------------------------------*/
    typedef ISubscription0  &lt;
                                    R
                                    &gt;  ISubscription_0;
    typedef ISubscription1  &lt;
                                    R, A1
                                    &gt; ISubscription_1;
    typedef ISubscription2  &lt;
                                    R, A1, A2
                                    &gt; ISubscription_2;
    typedef ISubscription3  &lt;
                                    R, A1, A2, A3
                                    &gt; ISubscription_3;
    //Now based on the template parameter,
    //choose the correct
    //function signature as 'TSubscription'
    typedef typename TSwitch
                         &lt;
                   TIsDummy&lt;A1&gt;::m_value,
                   ISubscription_0,
                   ISubscription_1
                         &gt;::TValue TSubscription_01 ;
    typedef typename TSwitch
                         &lt;
                   TIsDummy&lt;A2 &gt;::m_value,
                   TSubscription_01,
                   ISubscription_2
                         &gt;::TValue TSubscription_012;
    typedef typename TSwitch
                         &lt;
                   TIsDummy&lt;A3&gt;::m_value,
                   TSubscription_012,
                   ISubscription_3
                         &gt;::TValue TSubscription;

     typedef std::vector
                         &lt;
                    TSubscription
                         &gt; TSubscriptionContainer;
public:
     Observed()
     {}

     ~Observed()
     {
	//Signal owns the subscriptions
        //and are deleted
        //upon the observed's destruction
	for_each
           (
           m_slots.begin(),
	   m_slots.end(),
	   std::mem_fun( &amp;ISubscription::DeleteMe )
	   );
     }

/*-------------------------------------------------/
//  Fire functions of all possible signatures are
//  provided as member template functions.
//  Only the Fire function that matches with
//  the TSubscription can be used by the Observed.
//  Any other use will result in compilation error.
//  Either the constant or the non constant variant
//  of the Fire can be used by the Observed.
//------------------------------------------------*/
     void Fire(  )
     {
	TSubscriptionContainer::iterator tit;
        cout &lt;&lt;"calling 'void Fire()'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
		tit !=  m_subscriptions.end();
	   	++tit
	   )
	{
		TSubscription *psub = *tit;
		(*psub)( );
	}
     }
/*------------------------------------------------*/
     void Fire(   ) const
     {
	TSubscriptionContainer::const_iterator tit;
        cout &lt;&lt; "calling 'void Fire() const'"
             &lt;&lt; endl;
	for(  tit = m_subscriptions.begin();
		tit !=  m_subscriptions.end();
		++tit
	   )
	{
		TSubscription  *  psub = *tit;
		(*psub) ( );
	}
     }
/*------------------------------------------------*/
     template &lt; typename A &gt;
     void Fire( A a )
     {
	TSubscriptionContainer::iterator tit;
        cout &lt;&lt; "calling 'void Fire(A a)'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
	     tit !=  m_subscriptions.end();
	     ++tit
           )
	{
		TSubscription *psub = *tit;
		(*psub)( a  );
	}
     }
/*------------------------------------------------*/
     template &lt; typename A &gt;
     void Fire( A a  ) const
     {
	TSubscriptionContainer::const_iterator tit;
        cout &lt;&lt; "calling 'void Fire(A a) const'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
             tit !=  m_subscriptions.end();
             ++tit
           )
	{
		TSubscription  *  psub = *tit;
		(*psub)( a  );
	}
     }
/*------------------------------------------------*/
     template &lt;
            typename A,
            typename B &gt;
     void Fire( A a , B b )
     {
	TSubscriptionContainer::iterator tit;
        cout &lt;&lt; "calling 'void Fire(A a, B b)'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
             tit !=  m_subscriptions.end();
             ++tit
           )
	{
		TSubscription *psub = *tit;
		(*psub) ( a, b );
        }
     }
/*------------------------------------------------*/
     template &lt;
            typename A,
            typename B &gt;
     void Fire( A a , B b ) const
     {
	TSubscriptionContainer::const_iterator tit;
        cout &lt;&lt; "calling 'void Fire(A a, B b)const'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
             tit !=  m_subscriptions.end();
             ++tit
           )
	{
		TSubscription  *  psub = *tit;
		(*psub) ( a, b );
	}
     }
/*------------------------------------------------*/
     template &lt;
            typename A,
            typename B,
            typename C &gt;
     void Fire( A a , B b , C c)
     {
	TSubscriptionContainer::iterator tit;
        cout &lt;&lt; "calling 'void Fire(
                                           A a,
                                           B b,
                                           C c
                                           )'"
             &lt;&lt; endl;
	for( tit = m_subscriptions.begin();
	     tit !=  m_subscriptions.end();
	     ++tit
           )
	{
		TSubscription *psub = *tit;
		(*psub) ( a, b, c );
	}
     }
/*------------------------------------------------*/
     template &lt;
            typename A,
            typename B,
            typename C &gt;
     void Fire( A a , B b, C c ) const
     {
	TSubscriptionContainer::const_iterator tit;
        cout &lt;&lt; "calling 'void Fire(
                                          A a,
                                          B b,
                                          Cc) const'"
             &lt;&lt; endl;
	for(  tit = m_subscriptions.begin();
		tit !=  m_subscriptions.end();
		++tit
	   )
	{
		TSupscription  *  psub = *tit;
		(*psub)( a, b, c );
	}
     }
/*------------------------------------------------*/
     void Subscribe( TSubscription &amp;sub )
     {
	m_subscriptions.push_back( &amp;sub );
     }

     void Unsubscribe( TSubscription &amp;sub )
     {
        TSubscriptionContainer::iterator lit;
        lit = find( m_subscriptions.begin(),
                      m_subscriptions.end(),
                      sub );
        m_subscriptions.erase( lit );
     }

     TSubscriptionContainer  m_subscriptions;
};</pre>
<em>DummyType</em> is an auxilliary structure used to pass default template arguments. With the <em>DummyType</em>, we could templatize <em>Observed</em> as <em>Observed &lt; int &gt;</em> or <em>Observed &lt; int, int &gt;</em> or <em>Observed &lt; int, int, int &gt;</em>. The missing template parameters will be of type <em>DummyType</em>. <em>TIsDummy</em> is a templatized structure that accepts one template parameter and the value of its enum <em>m_value</em> is 1 only if the template parameter is <em>DummyType</em>. <em>TSwitch</em> has three template parameters. If the first parameter, which is of int type is 0, its <em>TValue</em> will be the third template type argument. For all other cases, its <em>TValue</em> is by default the second template type argument.
<pre>struct DummyType{};

//template structure to detect DummyType
template &lt; typename T &gt;
struct TIsDummy
{
	enum { m_value=0};
};

template &lt; &gt;
struct TIsDummy &lt; DummyType &gt;
{
	enum { m_value=1};
};

//If the value of the template parameter i passed in
//is non-zero TValue is A, else TValue is B
template &lt;
            int i,
            typename A,
            typename B &gt;
struct TSwitch
{
	typedef typename  A TValue;
};

template &lt; typename A, typename B &gt;
struct TSwitch &lt; 0, A, B &gt;
{
	typedef typename B TValue;
};</pre>
Going back to our <em>Button</em> class, let us implement a <em>ExampleButton</em>. The <em>ExampleButton</em> class should derive from the <em>Observed</em> class of appropriate template parameters. The <em>ExampleButton</em>'s <em>ClickMe</em> function should call one of the Fire functions. In fact, the <em>ExampleButton</em>'s <em>ClickMe</em> can call only the <em>Fire</em> function of appropriate template parameters. Any other inappropriate <em>Fire</em> function called would result in a compilation error. Let us conceive a <em>ExampleButton</em> that notifies the subscribers, by calling a subscription function that accepts the id(<em>int</em>) of the <em>ExampleButton</em> as the only argument. Let the subscription function return <em>void</em>. Here is an example of such a <em>ExampleButton</em>.
<pre>class ExampleButton : public Observed&lt; void, int&gt;
{
    public:
       ExampleButton( int id):
           m_id(id),
           m_state(false){}
    public:
       void ClickMe()
      {
            m_state = (m_state) ? false : true;
            Fire( m_id );
      }
    protected:
        int m_id;
        int m_state;
};</pre>
When the <em>ExampleButton</em> is clicked, it should call the function
<em>template void Signal::Fire( A a )</em>. If the <em>ClickMe</em> function of the <em>ExampleButton</em> was of signature <em>void ClickMe() const</em> then the compiler would have automatically chosen the function <em>template &lt; typename A &gt; void Signal::Fire( A a ) const</em> for firing.

As we can see from the above example, we are more or less done on providing the frame work on the signal side. The signal doesnt know about the actual subscription nature, (ie is it a member function, or a constant member function of an instance of some particular class, a static member function of some particular class or a global static function). Also the Signal can be templatized at compile time to use Slots of widely varying subscription signitures which accepts upto 3 arguments.
<h5>Subscription implementation for class member functions</h5>
So far we have talked only about abstract interfaces to subscription-s.
We should provide a subscription framework that makes it easier for the user to construct a subscription by denoting one the following
<ul>
	<li>a class instance and a member function,</li>
	<li>a class instance and a constant member function</li>
	<li>a member static function</li>
	<li>a global static function</li>
</ul>
First let us come up with a concrete implementation of Subscription that encompass these generalizations. As an example we choose to implment <em>Subscription1</em> which derives from <em>ISubscription1</em>.
<pre>//IsConst = is the subscription function
//	a constant member function,
//	as in 'R (H::*Fun) (Arg1) const'
//R = return type of the member function
//H = holder class of the member function
//A1 = First Argument of the member function
template &lt; int IsConst,
	typename R,
	typename H,
	typename A1 &gt;
class Subscription1 : public ISubscription1
                                         &lt;R, A1&gt;
{
    public:
	//get all possible function signatures here
	typedef R  (H::*TSubscriptionFun1)(A1);
	typedef R  (H::*TSubscriptionFun1C)(A1)const;
	//Now based on the template parameter,
	//choose the correct
	//function signature as 'TSubscriptionFun'

	typedef typename TSwitch
                                     &lt;
                                IsConst,
				TSubscriptionFun1C,
				TSubscriptionFun1
                                     &gt;::TValue
				TSubscriptionFun;

    public:
        Subscription1 (
                          H &amp;holder,
                          TSubscriptionFun sFun
                           ):
            m_pHolder( &amp;holder ),
            m_subscriptionFun( sFun )
            {}

        //main execute function
        R operator() ( A1 a )
        {
            return( m_pHolder-&gt;*m_subscriptionFun)(a);
        }

        //trivial destructor
        ~Subscription1()
        {}
    protected:
        H *         m_pHolder;
        TSubscriptionFun  m_subscriptionFun;
};</pre>
So we have now a concrete implementation of a <em>Subscription</em>. As an example, we need a subscriber who is going to use this subscription to get notified. Consider the following <em>ExampleSubscriber</em>.
<pre>class  ExampleSubscriber
{
    public:
       ExampleSubscriber( int id ):
           m_id( id ){}
      //details of the class
    public:
        //This is the example subscription function
        void NotifyMe( int signalId )
        {
             std::cout &lt;&lt;  m_id &lt;&lt; " is notified by "
                           &lt;&lt; signalId &lt;&lt; std::endl;
         }
        int m_id;
};</pre>
Using the <em>ExampleSubscriber</em> we can make a <em>Subscription1</em> as follows
<pre>ExampleSubscriber subscriber(78);
//IsConst = 0, The signature of Notify denotes that
//                    it is a non-constant function
//R = void
//H = ExampleSubscriber
//A1 = int
Subscription1 &lt; 0, void, ExampleSubscriber, int &gt;
            subscription(
                   subscriber,
                   ExampleSubscriber::Notify
                        );</pre>
To be used by a <em>Observed</em>, the subscription has to be created in heap, because once subscribed, the subscription is owned by the <em>Observed</em>. This is because, <em>Subscription1</em> (or in general <em>Subscription-n</em> is a light weight class and it is better if the keeper of the class takes care of its deletion. Also, to aid in the creation of a subscription, we can come up with helper functions called <em>SubscriptionMaker</em>.
<pre> template &lt;
        typename R,
        typename H,
        typename A1
         &gt;
Subscription1 &lt; 0, R, H, A1 &gt;* SubscriptionMaker
                           (
                                   H &amp;h,
                                   R (*H::Fun)( A1 )
                           )
{
        return new Subscription1
                                   &lt; 0, R, H, A1
                                   &gt;( h, Fun);
}</pre>
Now we can use the framework to tie the <em>Notify</em> function of the <em>ExampleSubscriber</em> to the <em>ExampleToggleButton</em>'s <em>ClickMe</em> function
as follows.
<pre>{
    ExampleButton button( 96 );
    ExampleSubscriber sub1( 78) ;
    button.Subscribe(
      *SubscriptionMaker
                     (
                     sub1,
                     &amp;ExampleSUbscriber::Notify
                     );
    button.ClickMe();
}</pre>
As you can see, all the template ugliness is hidden away from the user, and as a result the following outputs should be produced.
<pre>calling void Fire( A1 a )
78 is notified by signal 96</pre>
<h5>Subscription implementation for static functions</h5>
The above <em>Subscription</em> class worked only for subscription functions which are constant or non-constant class member functions.
We now need to provide a solution that covers member static functions of some particular classes or global static functions.
We can form an equivalent <em>Subscription</em> class for these static functions by conveniently specializing the template parameter H of <em>Subscription</em> to <em>void</em>.
<pre>//IsConst = 0, since static functions
//                       cannot be constant.
//R = return type of the member function
//H = void
//A1 = First Argument of the member function
template               &lt;
        typename R,
	typename A1
                          &gt;
class Subscription1 &lt; 0, R, void, A1 &gt;
                     : public ISubscription1
                              &lt; R, A1 &gt;
{
    public:
	typedef R  (*TSubscriptionFun)(A1);

    public:
        Subscription1 (
                          TSubscriptionFun sFun
                           ):
            m_subscriptionFun( sFun )
            {}

        //main execute function
        R operator() ( A1 a )
        {
            return(*m_subscriptionFun)(a);
        }

        //trivial destructor
        ~Subscription1()
        {}
    protected:
        TSubscriptionFun  m_subscriptionFun;
};</pre>
The subscription maker corresponding to that would be
<pre> template &lt;
        typename R,
        typename A1
         &gt;
Subscription1 &lt; 0, R, void, A1 &gt;* SubscriptionMaker
                           (
                                   R (*Fun)( A1 )
                           )
{
        return new Subscription1
                                   &lt; 0, R, void, A1
                                   &gt;( Fun);
}</pre>
The following example code can be used for that.
<pre>   //notify function is defined
    void Notify( int signalId )
   {
        cout &lt;&lt;
             "Notify function notified by signal"
             &lt;&lt; signalId &lt;&lt;endl;
   }

   class NotifyClass
   {
       static void Notify( int signalId )
       {
             cout &lt;&lt; "NotifyClass::Notify notified by signal"
                    &lt;&lt; signalId &lt;&lt; endl;
       }
   } ;
  //else where in the code body
    ExampleToggleButton button( 96 );
    button.Subscribe(
      *SubscriptionMaker
                     ( &amp;Notify
                     );
    button.Subscribe(
      *SubscriptionMaker
                    ( &amp;NotifyClass::Notify
                    );
    button.ClickMe();
}</pre>
which produces the output
<pre>calling void Fire( A1 a )
Notify function is notified by signal 96
NotifyClass::Notify notified by signal 96</pre>