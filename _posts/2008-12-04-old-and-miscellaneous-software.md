---
layout: post
title: "old and miscellaneous software"
description: ""
category: 
tags: []
---
{% include JB/setup %}

This is a collection of assorted software which I had written way back in 2003. Please note that these are 5 years old, so they may have lost some topical value now.




<h4>maya manipulator plugin</h4>
David Gould's <a href="http://www.davidgould.com/"><tt>Complete Maya Programming </tt></a>, provided some insights into how to make  a manipluator plugin. Here is one of my attempts.
Please find the code <a href="http://kgeorge-lib.googlecode.com/files/kgeorge-lib_maya_manipulator_12_03_08.rar">here</a>, that should work for Maya8.5.
<ol>
	<li>After you compile the code, please load in the plugin kgManip.mll.</li>
	<li>Source the mel script kgGenerateSlabs.mel.</li>
	<li>Run 'kgGenerateSlabs()' in the script window. This should create a grid of slabs and kgLocators
on top of the slabs.</li>
	<li>Select any kgLocator</li>
	<li>Modify/Transformation Tools/Show Maniplulator Tools. THis makes the corresponding manipulator visible. Trypulling the manipulator. This changes the slab height.</li>
</ol>
<h4>Environment mapping using renderman</h4>
I created the  following maya <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/vases_on_ground.mb"> scene </a> , using NURBS modeling in Maya. For aligning them on the floor, I dropped 10 or so vases from different heights,
and let them bounce and settle on the ground, utilizing a gravity field.
I had used the u-v texture editor to texture map the sky-dome using the following  spherical texture map <img class="alignnone size-full wp-image-98" title="sph_sky" src="http://kogocher.wordpress.com/files/2008/12/sph_sky.jpg" alt="sph_sky" width="377" height="381" /> which I found in the web. I used the maya-rib exporter that came with my Maya Unlimited 4.5, and since it didnt  export texture coordinates, I wrote  a mel
<a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/kgRibExportGeometry.mel">script </a> to export a selected geometry and texture coordinate as a rib file.
Next I had to generate the environment maps from the scene, so that they can be used in the environment map shaders for the balls. For this I created this mel <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/kgComputeEnvmap.mel"> script </a>. With these useful mel scripts, I could create the rib files<a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/vases_on_ground.rib"> 1</a>, and
<a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/vases_on_ground_geom.rib">2 </a>. As I said before, I used the standard environment mapping <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_env/kgEnv.sl"> shader </a> which was already available. Here is the result.<img class="alignnone size-full wp-image-99" title="shiny_balls_final" src="http://kogocher.wordpress.com/files/2008/12/shiny_balls_final.jpg" alt="shiny_balls_final" width="320" height="240" />

Two things that I have used  for this project have since then been discontinued.
<ul>
	<li>BMRT renderer for renderman</li>
	<li>Rib exporter, bundled with  Maya 4.5</li>
</ul>
<h4>Ambient Occlusion Shader for Renderman</h4>
<a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_amb_occ/occsurf.sl">Here</a> is an ambient occlusion shader for renderman. I have used <a href="http://en.wikipedia.org/wiki/Stratified_sampling">stratified sampling</a> of the hemisphere surrounding a point to get the ambient occlusion value.   Here <img class="aligncenter size-thumbnail wp-image-108" title="amb_occ" src="http://kogocher.wordpress.com/files/2008/12/amb_occ.jpg?w=128" alt="amb_occ" width="128" height="96" />is the result of ambient occlusion. Here <img class="aligncenter size-thumbnail wp-image-111" title="directional_lighting" src="http://kogocher.wordpress.com/files/2008/12/directional_lighting.jpg?w=128" alt="directional_lighting" width="128" height="96" />is the result of lighting the same scene with directional lighting. Here is the composited image.<img class="aligncenter size-thumbnail wp-image-110" title="composited_amb_occ_dirlight" src="http://kogocher.wordpress.com/files/2008/12/composited_amb_occ_dirlight.jpg?w=128" alt="composited_amb_occ_dirlight" width="128" height="96" />
<h4>Shader evaluation using python script</h4>
This is a small set of python scripts that renders a rib file multiple times, eachtime with different shader parameters. The scripts are
<ul>
	<li> <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_shader_python/myUtils.py">myUtils.py</a></li>
	<li><a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_shader_python/ribmodify.py">ribmodify.py</a></li>
	<li><a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_shader_python/shaderStat.py">shaderStat.py</a></li>
</ul>
First, edit the shaderParamDict in the script shaderStat.py to specify the maximum and minimum values, of the shader parameters to be tested, and a way to iterate them. These are parameters that you can specify in a declarative manner.

Then do

<tt>shaderStat.py &lt;input rib file name&gt;  &lt;shader name&gt;
&lt;sub sequence of shader parameter names that need be verified&gt;</tt>
I have used the following python features in this script.
<ul>
	<li> currying</li>
	<li>OO design</li>
	<li> iterators, generators</li>
	<li> dynamic changing of the functions of a module</li>
</ul>
Here are some resulting images for a simple <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_shader_python/morange.sl">noise </a>shader
on a simple <a href="http://code.google.com/p/kgeorge-lib/source/browse/trunk/kg_renderman_shader_python/vase.rib">vase</a> scene.
<img class="aligncenter size-full wp-image-122" title="shader_stat_result" src="http://kogocher.wordpress.com/files/2008/12/shader_stat_result.jpg" alt="shader_stat_result" width="460" height="403" />