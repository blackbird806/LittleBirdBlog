---
layout: post
title: "[WIP] A short introduction to L-Systems"
date: 2020-02-28
comments: false
---

# Introduction

A L-system aka Lindenmayer system is a rewriting system and a type of formal grammar, that can be used to create and generate a bunch of things. I will not dig to deep in the theorical definition of L-Systems since [wikipedia](https://en.wikipedia.org/wiki/L-system) already does it very well.

# What's a L-System ?

Let _V_ be an alphabet and _V+_ be the ensemble of valid words from _V_.  
Let _ω_ a starting __axiom__ from _V+_.  
Let _S_ be a set of constants.  
Let _P_ be a ensemble of __rewriting rules__, that reproduce elements of _V_.

A L-Systems is the aggregate : _L_ = { _V_, _ω_, _S_, _P_ }

A __rewriting rule__ (we will shorten that one in simply rule) is defined like this:  
 _Axiom_ -> _Production_  
 Where _Axiom_ is a element from _V+_ and _Production_ is a valid sequence of elements from _V+_.

# L-Systems from scratch

So let's precise more concretly.

Let _V_ = { _F_, _A_ }  
Let _S_ = { }  
Let _P_ = {  
	_F_ -> _FA_  
	_A_ -> _F_    
}  
Let _ω_ = _F_

Let _n_ = 0 .. 5 be the iteration number.

_n_ = 0 : _F_  
_n_ = 1 : _FA_  
_n_ = 2 : _FAF_  
_n_ = 3 : _FAFFA_  
_n_ = 4 : _FAFFAFAF_  
_n_ = 5 : _FAFFAFAFFAFFA_  

At each iteration each Axiom that match a rule is replaced/rewritted by its production. This is this behavior allow us to express easly recusive patterns like fractals for example.

# A practical Case

In this blog post we will mostly talk about using L-Systems for generate fractals and rendering stuff but the core of this system is a lot more generic than that and can be used for a lot of other uses cases.

We can easly interpret an L-Systems into graphics using technics such as [turtle graphics](https://en.wikipedia.org/wiki/Turtle_graphics).
All we have to do is define a subset of _V+_ that will compose an ensemble of commands wich will be interpreted by the turtle.
So let define them:

_F_ : moving forward  
_+_ : rotate right  
_-_ : rotate left  

And only with this we can start drawing our firsts fractals !  
(For convenience purposes I will define L-Systems as follow, starting from now)

_ω_ = _F_  
_F_ -> _F+F-F-F+F_  

_n_ = 0 : _F_  
_n_ = 1 : _F+F-F-F+F_  
_n_ = 2 : _F+F-F-F+F+F+F-F-F+F-F+F-F-F+F-F+F-F-F+F+F+F-F-F+F_  

![L-System fractal example](images/fract.PNG)

