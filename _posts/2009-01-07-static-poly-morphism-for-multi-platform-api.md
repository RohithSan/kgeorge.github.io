---
layout: post
title: "static poly morphism for multi platform api"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Suppose if we are developing games for multiple platforms with the same code base, often we need to use some amount of poly-morphism to capture the platform-generic nature of components. Let us say that we are developing for the PC and PS2 platform. It is quite natural for us to have a base class called <tt>Texture</tt>, and derived classes called <tt>PCTexture</tt> and <tt>PS2Texture</tt>. The derived classes would contain the platform specific implementation of the <tt>Texture</tt> component.  But in a single build configuration, we would be having only one kind of these derived classes. If the build configuration is <tt>PS2_Debug</tt> or <tt>PS2_Release</tt>, then the compiled code would not contain any instance of the <tt>PCTexture</tt> class. In fact, at compile time, we make the decision of which of the derived class is going to be used. So, the kind of runtime poly-morphism or dynamic poly-morphism using base-derived classes and v-tables is not a necessity.



This is a simple technique to express the same kind of polymorphic behavior of platform abstraction, using static poly-morphism.
<blockquote>An illustrative example can be downloaded <a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_static_polymorphism_multi_platform_010909.rar">here</a>.
<a href="http://code.google.com/p/kgeorge-lib/source/browse/#svn/trunk/kg_static_polymorphism_multi_platform">Here </a>is another viewable version of the code.</blockquote>
<h5>dynamic poly-morphism</h5>
The following code shows an implementation of the use of dynamic poly-morphism.
<pre>  //This is widget.h

#if defined(PC_PLATFORM)
#define KGPLATFORM_PC  1
#define KGPLATFORM_PS2 0
#elif defined(PS2_PLATFORM)
#define KGPLATFORM_PC  0
#define KGPLATFORM_PS2 1
#endif

  class Widget
  {
    public:
    Widget();
    virtual ~Widget();
    void CommonMethod();
    virtual void PlatformSpecificMethod()=0;
  };</pre>
<pre>  //This is widget.cpp
  //Which holds the implementation of
 //non-virtual methods of the base class.

  Widget::Widget()
  {
     //platform-generic widget implementation
  }

  Widget:~Widget()
 {
    //platform-generic widget implementation
 }

  void Widget::CommonMethod()
 {
   //platform generic implementation of common method
 }</pre>
Now  the platform specific implementation for the PC platform would be
<pre>  //This is pc_widget.h
  class PCWidget :public Widget
  {
    public:
    PCWidget();
    ~PCWidget();
    void PlatformSpecificMethod();
  };</pre>
<pre>  //This is pc_widget.cpp
  PCWidget:: PCWidget()
 {
   //platform specific contrcuctor
 }
   PCWidget::~PCWidget()
 {
  //platform specific destructor
 }
  void PCWidget:: PlatformSpecificMethod()
 {
  //platform specific implementation
 }</pre>
The main file that would be calling widget methods would be,
<pre>//This is main.cpp or some .cpp file that uses widget

#include "widget.h"
#if KGPLATFORM_PC
#include "pc_widget.h"
#elif KGPLATFORM_PS2
#include "ps2_widget.h"
#endif
int main( int argc, char **argv)
{
   Widget *pw = NULL;
#if KGPLATFORM_PC
 pw = new PCWidget();
#elif KGPLATFORM_PS2
 pw = new  PS2Widget();
#endif

  pw-&gt;CommonMethod();
  pw-&gt;PlatformSpecificMethod();

  delete pw;
  return 0;
}</pre>
<h5>static poly-morphism</h5>
Now let us implement the same functionality using static poly-morphism.
<pre>//this is widget.h
#if defined(PC_PLATFORM)
#define KGPLATFORM_PC  1
#define KGPLATFORM_PS2 0
#elif defined(PS2_PLATFORM)
#define KGPLATFORM_PC  0
#define KGPLATFORM_PS2 1
#endif

   template&lt; int iPlat&gt;
   class Widget
   {
      public:
        Widget();
	~Widget();
	void CommonMethod()
       {
	 //platform generic method
       }
	void PlatformSpecificMethod();
   };</pre>
And the platform specific implementation  for PC would be
<pre>  //this is  pcWidget.cpp
   template&lt;&gt; Widget&lt;KGPLATFORM_PC&gt;::Widget()
  {
    // platform specific implementation of Widget
  }

   template&lt;&gt; Widget&lt;KGPLATFORM_PC&gt;::~Widget();
  {
    //platform specific implementatin of ~Widget
  }

  template&lt;&gt; void Widget&lt;KGPLATFORM_PC&gt;::
PlatformSpecificMethod()
  {
	 //platform specific method
  }</pre>
The widget methods can be called as  follows.
<pre>#include "widget.h"

int main( int argc, char **argv)
{
   //Always the current platform to which the
   //code has ben compiled for will have the
   //template parameter 1
   Widget&lt;1&gt; *pw = new Widget&lt;1&gt;();
   pw-&gt;CommonMethod();
   pw-&gt;PlatformSpecificMethod();
   delete pw;

   return 0;
}</pre>
This is much simpler and cleaner and more efficient too, since we are getting rid of runtime dispatching using v-tables.

The code can be seen <a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_static_polymorphism_multi_platform_010909.rar">here</a>.

<h5> issues </h5>
<ul>
<li>code bloat: One of the problems with using templates in c++ language is that upon compiling, each template instantiation, tends to generate its own copy of the code. However this is not an issue here, since we don't instantiate anything other than the code for the current platform for each build configuration. Eg: <tt> PS2_Debug </tt> configuration, when compiled wont be having any <tt>Widget&lt;KGPLATFORM_PC&gt;</tt> instantiations.   </li>
<li> Readability: Templates often result in unreadable code. But in this case, we could use <tt>typedef</tt>  for easy readable synonyms which doesnt have the template construct. <tt>typedef Widget&lt;1&gt; CurrentWidget</tt></li>

<li>Link issues involving compilers: Although C++ standard doesn't have any such requirement, some compilers require that,
bodies of template functions or member functions must be in the header file. Otherwise the translation units (.cpp files) using the template instantiations, wont find them and cause link errors. In this case, any such generic code (Eg: Widget::CommonMethod), should indeed be completely defined in the header file. But at least for visual studio compilers, the specialization, which is were the platform specific code resides, can be defined in the .cpp files.
<li>Tool Build Configurations: In game development, usually the runtime engine have  platform specific build configurations (Eg: PS2_Debug) and for asset building and authoring purposes the tools are built with a different configuration, which would be mostly equivalent to <tt>PC_Debug</tt> or <tt>PC_Release</tt>. For such configurations, there maybe a need where, all of the different platform specific instantiations may occur in the same configuration. Imagine a texture utility tool, which based upon a runtime parameter makes either <tt>PS2TExture</tt> or <tt>PCTexture</tt> in runtime. We suggest using enumeration template parameters for a tool build configuration and using  the platform specific instantiations, based on  that.
<tt>
<pre>
#if defined(TOOL_BUILD)
#define KGPLATFORM_PC    1
#define KGPLATFORM_PS2   2
#define KGPLATFORM_X360 3
#define KGPLATFORM_PSP  4
#endif
</pre>
</tt>
</li>
</ul>