---
layout: single
title: "Refactoring towards functional programming"
tags:
   - .NET
   - Functional programming
---

> This is part of a series on refactoring towards functional programming.

---
**[Introduction]({% post_url 2018-07-25-refactoring-towards-functional-programming-introduction %})**  
&nbsp;&nbsp;[Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})  
&nbsp;&nbsp;&nbsp;&nbsp;[Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})  
Creating a functional API  
&nbsp;&nbsp;&nbsp;&nbsp;Transforming with Map  
&nbsp;&nbsp;&nbsp;&nbsp;Excluding with Filter  
&nbsp;&nbsp;&nbsp;&nbsp;Isolating side effects with ForEach  
Functional composition  
&nbsp;&nbsp;&nbsp;&nbsp;Functional composition through extension methods  
&nbsp;&nbsp;&nbsp;&nbsp;Refactoring to point free  
Functional refactoring conclusions  

---

## Introduction

If know the basics of functional programming, [click here to skip ahead to the code]().

This series of blog posts is meant to give you a practical introduction to functional programming through the use of common list operations. No prior knowledge of functional programming is required but some familiarity with programming in general is assumed. The language used in the example code is C# while the principles taught are language agnostic.

The motivation behind these posts is to introduce functional programming through concrete examples with clear benefits. I’ll skip most FP jargon but won’t shy away from complexity. Functional programming has a high barrier of entry, not because it’s inherently harder than object-oriented programming, but because it is unfamiliar. We’ll jump off the deep end with higher order functions and functional composition, but in the end, we’ll have made an API you might be familiar with. You may be surprised to find that you have been doing some functional programming all along.

Next up, [Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})