---
layout: page
title: "Javascript for Pythonistas"
description: "Notes while learning Javascript"
---
{% include JB/setup %}

Introduction
============
Mostly C-like syntax - think curly braces, semicolons etc.

    var a = [1, 2, 3];
    for (var i=0; i<a.length; i++){
        console.log(a[i]);
    }

You get the picture. [Language reference][jsref] at MDN.

Javascript itself
=================
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

[jsref]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference

Loops
-----
The for-in loop iterates over properties of an object and as a special case
this also works for arrays.

    a = [1, 2, 3]
    for (el in a)
        console.log(a)
