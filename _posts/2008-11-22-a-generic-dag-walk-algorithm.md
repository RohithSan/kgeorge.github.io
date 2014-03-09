---
layout: post
title: "A generic DAG walk algorithm"
description: ""
category: 
tags: [generic programming, c++, design patterns, object oriented programming]
---
{% include JB/setup %}

This is an example of a simple walk through a dag data structure,  written in the paradigm of<span style="text-decoration:underline;"> </span><a href="http://en.wikipedia.org/wiki/Generic_programming">generic programming</a>. Since a tree is always a dag, this can be easily applied to a tree walk, with or without further simplification.
<blockquote>You can access this as a downloadable version <a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_11_22_08.rar"> here</a>. The core generic dag walk header file is <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_dag_walk/dag_walk.h"> here</a>.</blockquote>
<h4>Introduction</h4>
The contribution of this implementation here is two unrelated functionalities.
<ul>
	<li> Using generic programming, split data structure and algorithm, there by promoting code re-use</li>
	<li> Expose the api to preserve user defined stackable information during the walk, through program stack and recursion</li>
</ul>




We encounter these data structures in multiple occasions in our projects. In projects related to games programming, examples of tree traversal can be,  traversing a bounding volume hierarchy tree of a polygon soup, or, traversing the bone hierarchy of a character. An example of a DAG traversal would be, traversing the dependency DAG of resources. When we write code for each of these cases of traversals,  we usually succumb to using  a separate  pieces of code. This is because,  the data structures corresponding to the bounding hierarchy traversal  is not exactly the same as the data structures for the traversal of animation hierarchy.

One might be tempted to use object oriented programming here, and make base and derived classes, so that generic code can be hoisted to the base class. But sadly OOP, demands unrelated classes to be put into hierarchies, just for the sake of code re-use.

Generic  programming doesn't demand such hierarchies. Generic programming aims to bring re-usabilty by  splitting the algorithm and the data structure.  The traditional way to implement a generic tree walk  or dag walk is through an iterator. But such an implementation, necessitates the iterator class to maintain a stack of where it was, so that it can find the next node to visit, which is kind of unweildy. Our implementation doesnt need an explicit stack of remembering where it is, for the purpose of traversing.

The generic <tt>dag_walk</tt> algorithm, takes a dag <tt>Node</tt> class  and a <tt>Walk</tt> class as  template arguments.

Eventhough our algorithm doesnt need a stack to keep track of where the current position is, for utility purposes, we expose the abilty to preserve stackable information in the program stack.  Many times when you walk a tree or dag, you often feel the necessity to maintain a stack for storing <tt>Walk</tt> related data. For eg: suppose if you are traversing a scene graph, you may need to store the cumulative transformation matrix. This implementation of the <tt>dag_walk</tt>, in adition to splitting the algorithm and data structure, allows the walk-implementer to designate  an explicit <tt>stack_type</tt> and  a stack variable. The dag walk is done in a recursive way, which means that the program stack can be used to store the stack variable designated by the <tt>Walk</tt> class.
<h4>User's point of view</h4>
<ul>
	<li>The user has to supply a dag <tt>Node</tt> class  and a <tt>Walk</tt> class  as  template arguments.</li>
	<li>Additionally the user has to specialize a <tt>node_traits</tt> struct  and <tt>walk_traits</tt> struct with the <tt>Node</tt> and <tt>Walk</tt> classes that he is supplying.</li>
</ul>
The folowing requirements need be met for the DAG <tt>Node</tt> class.

<em>Absolute requirements are requirements necessary for the dag_walk to compile.
Functional requirements are requirements necessary for the dag_walk to function.</em>

Absolute requirements of <tt>Node</tt> class:
<ul>
	<li> None</li>
</ul>
Functional Requirements  of the <tt>Node</tt> class:
<ul>
	<li> The <tt>Node</tt> should have the provision of setting and unsetting a <tt>visit</tt> flag,  which signifies whether the <tt>Node</tt> has already been visited or not.</li>
	<li> The requirement can be relaxed, if we anticipate only tree traversals on a dag (ie visiting many times <span style="font-family:monospace;">a </span><tt>Node</tt> with multiple parents) , or if the dag is guaranteed to be a tree.</li>
</ul>
Here is an example for a typical user supplied <tt>Node</tt>
<tt></tt>
<pre><tt>  struct Node {
      Node( int data):_data(data),_visited(false){}
      int _data;
      list&lt; Node *&gt; _children;
      bool _visited;
  };
</tt></pre>
For the second template argument, the <tt>Walk</tt> class embodies the work done on the <tt>Node</tt>. Often the<tt> Node</tt> class is a rigid, predetermined class supplied by a thrid party, where as the second template argument, the <tt>Walk</tt> class is  a glue class written by the person who uses the <tt>Node</tt> class. The <tt>Walk</tt> class must meet the following requirements.

Absolute rquirements of the <tt> Walk </tt> class:
<ul>
	<li>The <tt>Walk</tt> class must define a <tt>stack_type</tt>, which should be copy_constructible and assignable.</li>
	<li>The <tt>Walk</tt> class should have a member variable of the type <tt>stack_type</tt>.</li>
</ul>
If the walk doesnt need to use stackable information, then a dummy <tt>stack_type</tt>, and a dummy  <tt>stack_type</tt> variable should  be defined.

Functional requirements of the <tt> Walk </tt>class:
<ul>
	<li> The <tt>Walk</tt> class should store the stackable information during the dag traversal in the stack variable.Eg: If the dag was a scene  graph, the <tt>stack_type</tt> of the <tt>Walk</tt> could contain the current cumulative matrix, at each <tt>Node</tt>. This trick, enables us to use the program stack to store the <tt>stack_type</tt> variable while we do recursive traversal of the DAG. Even though the existance of the<tt> stack_type</tt> and the <tt>stack_tpe</tt> variable are a compilation necessity, if we dont need them, you can just use dummy values here.</li>
	<li> The <tt>Walk</tt> class should have a  <tt> pre( Node *) </tt> and a <tt>post (Node *)</tt> function which does the main work on the <tt>Node</tt>, before and after traversing the children of the <tt>Node</tt>.</li>
</ul>
Here is an example of the Walk data structure, which prints the dag depth at each Node.

<tt></tt>
<pre><tt>struct Walk {
  Walk():_level(0){}
  typedef int stack_type;
  stack_type _level;
  stack_type &amp; get_stack() { return _level;}
  const stack_type &amp; get_stack() const { return _level;}
  void print( Node * pNode ) {
      cout &lt;&lt; pNode-&gt;_data &lt;&lt; endl;             }
  bool pre( Node * p){
      inc_level();
      print(p);
      return true;
 }
 bool post( Node *) {
      dec_level();
      return true;
 }
 void inc_level() {
      ++_level;
 }
 void dec_level() {
      --_level;
 }
}; </tt></pre>
We have  defined  the following return values for the algorithm to return.
<tt></tt>
<pre><tt>  typedef enum { eOK, ePrune, eStop, eError } walk_result;
</tt></pre>
In order to  facilitate generic access of the <tt>Node</tt> and <tt>Walk</tt> classes, one should specialize the node_traits and walk_traits structs with the <tt>Node</tt> and the<tt> Walk</tt> classes.

The <tt>node_traits</tt> struct which takes the <tt>Node</tt> as a  template parameter should define
<ul>
	<li> the set and get functions to access the  visit flag of the <tt>Node</tt></li>
	<li> get the begin and end iterators of the container that hold references to the children of a <tt>Node</tt>. ( If the child container of the <tt>Node</tt> deosnt have an iterator class,
one should define one )</li>
</ul>
Here is a specialization of the <tt>node_traits</tt> struct.
<tt></tt>
<pre><tt>template &lt; &gt;
struct node_traits &lt; Node &gt; {
  typedef Node N;

  //get the visit tag from the node
  static bool get_visit_tag(const N * pNode ) { return pNode-&gt;_visited;}
  //set the visit flag
  static void set_visit_tag( N *pNode, bool b ) { pNode-&gt;_visited= b; }

  //how the children of the nodes are defined
  typedef list&lt;Node*&gt; node_container;
  typedef list::iterator node_iterator;
  typedef list::const_iterator node_const_iterator;

  //get the begin and end iterators for the children
  static node_iterator get_node_container_begin( N *pNode ) { return pNode-&gt;_children.begin();}
  static node_const_iterator get_node_container_begin(const  N *pNode ) { return pNode-&gt;_children.begin();}
  static node_iterator get_node_container_end( N *pNode ) { return pNode-&gt;_children.end();}
  static node_const_iterator get_node_container_end(const N *pNode ) { return pNode-&gt;_children.end();}

};
</tt></pre>
The <tt>walk_traits class</tt>, which takes both the <tt>Node</tt> and the <tt>Walk</tt> class as template arguments should

define
<ul>
	<li>whether multiple visits to the <tt>Node</tt> are allowed (ie: it can behave like a tree or is it
strictly a dag)</li>
	<li>call the appropriate delegataion codes in the <tt>Walk</tt> class for the <tt>pre and post</tt> calls</li>
	<li>access functions for get-ing and set-ing the current stack</li>
</ul>
<tt></tt>
<pre><tt>template &lt; &gt;
struct walk_traits&lt;&gt; : public walk_traits_base {
  typedef Walk W;
  typedef Node N;
  typedef W::stack_type stack_type;

  //does this walk allows multiple visits for
  //the dag node. generally during a dag walk, the nodes
  // visisted once are not visited again
  static const bool is_multiple_visits_allowed = false;

  //main operation that need be performed on the node
  //This is called in dag_walk, for each node,
  //before we recurse into the dscendents
  static walk_result pre(  N *pNode, W &amp; w )
      { return (w.pre( pNode )) ? eOK : eStop; }

  //get the curent stack variable of W
  static stack_type &amp; get_stack( W &amp; w){ return w.get_stack(); }
  static const stack_type &amp;get_stack( const W &amp; w) { return w.get_stack();}

  //set the curent stack on W
  static void set_stack( W &amp;w, stack_type &amp;s ){ w._level = s; }

  //main operatrion that will be called after visiting the descendants
  //this function is called after visiting the children
  static walk_result post( N * pNode, W &amp;w, const walk_result &amp;res )
      { return (w.post( pNode) ) ?  eOk : eStop;}
};
</tt></pre>
<h4>Core algorithm</h4>
Finally here is the data structure indpendent algorithm  code for dag_walk.

<tt></tt>
<pre><tt>/**
*    Generic dag_walk procedure
*
*   - do the pre operation on the node
*   - visit the children and call dag_walk recursively
*   - restore the stack variable after each child visit.
*   - do the post operation on the node
*   @param pNode[in, out] pointer to node.
*          Note that pNode is a not a const pointer and is mutable
*
*   @param w [in, out] the walk operation on the node
*   @return walk_result
**/

template &lt;typename N, typename W&gt;
walk_traits_base::walk_result dag_walk( N *pNode, W &amp;w)
{
  typedef typename walk_traits&lt;W&gt; wtraits;
  typedef typename node_traits&lt;N&gt; ntraits;
  typedef walk_traits_base::walk_result walk_result;
  using walk_traits_base::eOK;
  using walk_traits_base::ePrune;
  using walk_traits_base::eStop;
  using walk_traits_base::eError;

  walk_result res = eOK;
  bool b_visit_children = true;

  //there is no need to visit this node
  //if multiple visits are not allowed and it has already been visited earlier
  bool b_should_do_main_operations = wtraits::is_multiple_visits_allowed || !ntraits::get_visit_tag( pNode);
  if( b_should_do_main_operations)
  {
      res = wtraits::pre( pNode, w );

      //set the visit flag for this node
      ntraits::set_visit_tag( pNode, true );
      //if the result of the main operation
      //is prune, stop or error, the not visit children
      if ( res &gt; eOK)
      {
          b_visit_children = false;
      }
  } else {
      //there is no need to continue
      //this  node has been visited once
      return res;
  }
  //if the children ought to be visited
  if ( b_visit_children )
  {
      //copy the current stack
      //we need to rstore it after visiting chilren
      wtraits::stack_type s_copy( wtraits::get_stack( w) );
      ntraits::node_iterator nit, nbegin, nend;
      nbegin = ntraits::get_node_container_begin( pNode );
      nend = ntraits::get_node_container_end( pNode );
      for( nit = nbegin; nit != nend; ++ nit )
      {
          N * pChild = *nit;
          assert( pChild );

              res = dag_walk( pChild, w );

          //restore the stack on w
          //so that the next child gets the same context
          wtraits::set_stack( w, s_copy);

          //if the rsult of this walk is more serious
          //than ePrune, let us quit
          if ( res &gt;= wtraits::eStop )
          {
              break;
          }
      }
  }
  //finally do the post operation
  //The post operation is the one done after vsisting the children.
  //It is passed the previous value of res.
  //So the first step in post, has to check the result so far,
  //and treat it accordingly
  if( res &lt;= ePrune )     {
      res = wtraits::post( pNode, w,  res );
 }
return res;
}  </tt></pre>
Here is the version of the above stored at  <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_dag_walk/dag_walk.h"> code/google </a> .<a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_11_22_08.rar"></a><a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_dag_walk/dag_walk.h"><ins datetime="00"></ins></a>
