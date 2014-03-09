---
layout: post
title: "stl wrapper for maya container classes"
description: ""
category: 
tags: []
---
{% include JB/setup %}

In the <a href="http://www.google.com/search?client=firefox-a&amp;rls=org.mozilla%3Aen-US%3Aofficial&amp;channel=s&amp;hl=en&amp;q=autodesk+maya&amp;btnG=Google+Search"> Maya</a> SDK array container classes which are
<tt></tt>
<ul>
	<li><tt> MIntArray </tt></li>
	<li><tt> MFloatArray </tt></li>
	<li><tt> MDoubleArray </tt></li>
	<li><tt> MVectorArray </tt></li>
	<li><tt> MPointArray </tt></li>
	<li><tt> MFloatPointArray </tt></li>
	<li><tt> MFloatVectorArray</tt></li>
	<li><tt>MCallbackIdArray</tt></li>
	<li><tt>MAttributeSpecArray</tt></li>
	<li><tt>MColorArray</tt></li>
	<li><tt>MDagPathArray</tt></li>
	<li><tt>MMatrixArray</tt></li>
	<li><tt>MRenderLineArray</tt></li>
	<li><tt>MTimeArray</tt></li>
	<li><tt>MTrimBoundaryArray</tt></li>
	<li><tt>MUint64Array</tt></li>
</ul>
has got an entirely different interface compared to the stl conatiner classes. They dont expose any iterator interface. Consequently we cannot use the data contained in them in stl or generic algorithms.

So we have come up with a templatized wrapper class <tt>MArray_stl</tt>.  A wrapper class for <tt>MFloatVectorArray</tt> would be defined as

<tt> typedef kg::MArray_stl&lt; float, MFloatVector, MFloatVectorArray &gt; MArray_stl_fv;</tt>




<blockquote>The code can be found <a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_maya_stl_wrapper_11_25_08.rar"> here which is tested to work for Maya8.5.</a></blockquote>
Since the  container classes that we wrap are not light weight classes and carry heavy data, the wrapper class just maintains a pointer to the allready constructed maya container class. For example an instance of <tt>MArray_stl_fv</tt> would maintain a pointer to the <tt>MFloatVectorAray</tt> that it is pointing to.

<code>
MFloatVectorArray fa;
//populate this float vector array and use it for something.
typedef typename kg::MArray_stl kgWrapper;
kgWrapper kg_fa( fa );
//do some generic programming code with kg_fa
</code>
Here are the differences of the <tt> MArray_stl </tt> class compared with<tt> std::vector</tt>.
<ul>
	<li> The only constructors to the wrapper class include a copy constructor, and a constructor that accepts a reference to the maya container class. We don't allow a default constructor for <tt> MArray_stl</tt>, since it mean that the wrapper class  <tt>MArray_stl</tt> would have to  carry a NULL pointer to the corresponding container class. If it carries a NULL pointer, then what would <tt>begin</tt> and <tt>end</tt> iterators be?</li>
	<li> since Maya container classes doesn't expose any <tt>reserve</tt> functionality, the <tt>reserve</tt> function of the wrapper class doesn't do anything. Also the <tt>capacity</tt> function just returns the length of the container.</li>
	<li> The allocator member variable doesn't perform any functionality and is a dummy value.</li>
       <li> Also, use the copy constructor and assignment operator with caution, the use of which   leads to two wrappers pointing to the same wrapee.</li>
</ul>
The core header file where this is referenced can be found at <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_maya_stl_wrapper/inc/kg_marray_stl.h"> here</a>

