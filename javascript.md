---
layout: page
title: Javascript for Pythonistas
description: Notes while learning Javascript
published: false
---

{% include JB/setup %}

I. Introduction
===============
Being a long-time Python user, I started learning Javascript and decided to make some notes along the way to more easily memorize the similarities and the differences. Here they are...

Mostly C-like syntax - think curly braces, semicolons etc.

    var a = [1, 2, 3];
    for (var i = 0; i < a.length; i++){
        console.log(a[i]);
    }

You get the picture. [Language reference][jsref] at MDN.

II. Javascript itself
=====================
Language-specific details.

Hoisting
--------
Variables have function-level scope (unlike C which is block-level). This means
that wherever a variable is defined, its variable declaration is hoisted up to
the beginning of the function. Function declaration statements are also hoisted
as is their initialisation.

Functions
---------

### Declaration
Function declaration statement or function declaration expression.

### Clojures
Nested functions where the inner function accesses the outer function's
variables.


Statements
----------
A nice [summary][statements] is given on MDN.

### Loops
The for-in loop "enumerates object properties" - it iterates over properties
of an object and as a special case this also works for arrays.

    a = [1, 2, 3]
    for (el in a)
        console.log(a)

### Labels
Similarly to GOTOs in C, labels can be added to any command in JS. Commands that
can jump to these labels are:

 - break
 - continue

### With
Deprecated. Unlike Python's with, in JS it simply brings an expression into
scope - equivalent to `from bla import *`.

### Strict mode
It throws errors and forbids bad things more rigorously. You should use it. 

    "use strict"

Older JS versions will simply ignore it.

Objects
-------

[jsref]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference
[statements]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements