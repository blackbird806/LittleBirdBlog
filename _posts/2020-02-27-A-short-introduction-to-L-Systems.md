---
layout: post
title: "[WIP] A short introduction to L-Systems"
date: 2020-02-27
comments: false
---

# Introduction

L-systems aka Lindenmayer systems is a rewriting system and a type of formal grammar, that can be used to create and generate a bunch of things. I will not dig to deep in the theorical definition of L-Systems since [wikipedia](https://en.wikipedia.org/wiki/L-system) already does it very well.

## What's a L-Systems ?

Let _V_ be an alphabet and _V+_ be the ensemble of valid words from _V_.  
Let _ω_ a starting __axiom__ from _V+_.  
Let _S_ be a set of constants.  
Let _P_ be a ensemble of __rewriting rules__, that reproduce elements of _V_.

A L-Systems is the aggregate : L = {_V_, _ω_, _S_, _P_}

A __rewriting rule__ (we will shorten that one in simply rule) is defined like this:  
 _Axiom_ -> _Production_  
 Where _Axiom_ is a element from _V+_ and _Production_ is a valid sequence of elements from _V+_.

## L-Systems from scratch

So let's precise more concretly.

Let V = { F, A }  
Let S = { }  
Let P = {  
	F -> FA  
	A -> F    
}  
Let ω = F

Let n = 0 .. 5 be the iteration number.

n = 0 : F  
n = 1 : FA  
n = 2 : FAF  
n = 3 : FAFFA  
n = 4 : FAFFAFAF  
n = 5 : FAFFAFAFFAFFA  

At each iteration each Axiom that match a rule is replaced/rewritted by its production. This is this behavior allow us to express easly recusive things like fractals for example.

## A practical Case

In this blog post we will mostly talk about using L-Systems for generate fractals and rendering stuff but the core of this system is a lot more generic than that and can be used for a lot of other uses cases.

We can easly interpret an L-Systems into graphics using technichs such as [turtle graphics](https://en.wikipedia.org/wiki/Turtle_graphics).
All we have to do is define a subset of _V+_ that will compose an ensemble of commands wich will be interpreted by the turtle.
So let define them:

_F_ : moving forward  
_+_ : rotate right  
_-_ : rotate left  

And only with this we can start drawing our firsts fractals !  
(For convenience purposes I will define L-Systems as follow, starting from now)

ω = F  
F -> F+F-F-F+F  

n = 0 : F  
n = 1 : F+F-F-F+F  
n = 2 : F+F-F-F+F+F+F-F-F+F-F+F-F-F+F-F+F-F-F+F+F+F-F-F+F  

__TODO PICTURES__ 

