---
layout: post
title: "facial deformation transfer"
description: ""
category: 
tags: [scattered data interpolation, facial deformation, deformation transfer, facial animation, radial basis funcions]
---
{% include JB/setup %}

We discuss a scheme by which we can transfer the deformation of a source face
to a target face, using a given set of correspondence pair of points.
The target face could be of different, geometry, or transformation.




#### acknowledgement

We would like to thank Lazlo Vespermi of [abalone llc](http://www.abalonellc.com/) for providing us with the source material.

First let us familiarize with some terms.

#### source mesh
![source mesh]({{ site.url }}/assets/content/source_neutral.jpg)

A well defined regular facial mesh, resembling a mannequin's face.

#### target mesh

![target mesh]({{ site.url }}/assets/content/target_neutral2.jpg)

Target mesh is obtained from real people, using commercial computational photography techniques.

#### neutral state of a mesh

is the un-deformed state of the mesh.

#### feature points set

![source mesh]({{ site.url }}/assets/content/source_feature_points.jpg)

A set of points in the mesh which denote predetermined features.
Shown are the 28 feature points in the source mesh.
A feature point for a mesh is represented as an index to the triangles of the mesh,
and a three tuple bary-centric co-ordinates in the triangle.
The feature points are ordered. Eg: 20th feature point is the left end of the lips.

#### problem specification

The fixed inputs to the system are

	* neutral source mesh
	* neutral target mesh
	* set of 28 feature points for neutral source mesh
	* corresponding set of 28 feature points for target neutral mesh


Given the above fixed inputs, for each given deformation of the source mesh, compute the deformation of the target mesh.

#### results

Shown below is a deformation of the source mesh and a corresponding deformation of the target mesh.

![source mesh]({{ site.url }}/assets/content/deformed_source_mesh.jpg)
![source mesh]({{ site.url }}/assets/content/target_deformed.jpg)

#### method

We use the same technique of [scattered data interpolation](https://www.google.com/#q=scattered+data+interpolation),
that is described in
[Marco Fratarcangeli, Marco Schaerf and Robert Forchheimer:Facial Motion Cloning with Radial Basis Functions in MPEG-4 FBA](http://www.fratarcangeli.net/pubs/fratarcangeli_gmod07.pdf).




Let us talk about the neutral state.
Let $$S_j, j=0, 27$$ be the set of 28 feature points for the source mesh.
Let $$T_j, j=0, 27$$ be the corresponding set of target feature points.

For any point $$Q= [ x, y, z]$$ in the source mesh, $$G(Q)$$ will give the corresponding point in the target mesh.



$$
G(Q) = \sum_{j=0}^{27}( H_j . R( S_j, Q ) )
$$
Given two points $$P = [x,y,z]$$ and $$Q = [x',y'z']$$,
$$R(P, Q)$$ is a radial basis kernel, which evaluates to a scalar, .
$$H_j$$-s are vectors.
By equating $$T_i$$ with $$G(S_i)$$ we get a linear set of equations that can be solved to get the unknown $$H_j$$-s.


#### references

*  [Marco Fratarcangeli, Marco Schaerf and Robert Forchheimer:Facial Motion Cloning with Radial Basis Functions in MPEG-4 FBA](http://www.fratarcangeli.net/pubs/fratarcangeli_gmod07.pdf).


#### acknowledgement

We would like to thank Lazlo Vespermi of [abalone llc](http://www.abalonellc.com/) for providing us with the source material.