---
layout: single
title: "Imperative versus declarative programming"
tags:
   - .NET
   - Functional programming
author_profile: false
---

**What the difference is between imperative and declarative, and why you should care.**

> This is part of a series on refactoring towards functional programming.

---
[Introduction]({% post_url 2018-07-25-refactoring-towards-functional-programming-introduction %})  
&nbsp;&nbsp;**[Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})**  
&nbsp;&nbsp;&nbsp;&nbsp;[Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})  
[Designing a functional list API]({% post_url 2018-08-04-designing-a-functional-list-api %})  
&nbsp;&nbsp;&nbsp;&nbsp;[Writing a filter function]({% post_url 2018-08-06-writing-a-filter-function %})  
&nbsp;&nbsp;&nbsp;&nbsp;Transforming with Map  
&nbsp;&nbsp;&nbsp;&nbsp;Isolating side effects with ForEach  
Functional composition  
&nbsp;&nbsp;&nbsp;&nbsp;Functional composition through extension methods  
&nbsp;&nbsp;&nbsp;&nbsp;Refactoring to point free  
Functional refactoring conclusions  

---

Programming can be roughly categorized into two types – imperative and declarative. Imperative programming is telling the program *how* to give you a result. Declarative programming is telling it *what* you want. Broadly speaking, object-oriented programming falls under the imperative style while functional programming is declarative. Imperative programs are written using statements such as ```if```, ```while``` and ```for```.

Consider the following iterative code. To get the sum of a list of values we traverse the list and add the values.

```csharp
int sum = 0;
for (int i = 0; i < values.Length(); i += 1)
{
    sum += values[i];
}
Console.WriteLine(sum);
```

As you can see, we are very specific as to *how* to perform the task. We have to create an index value (```int i = 0```), make sure to update it (```i += 1```) and have a valid terminating condition (```i < values.Length()```). We also need to declare a variable (```int sum = 0```) and make sure to increment it (```sum += values[i]```). Finally, we write the resulting value to the console (```Console.WriteLine(sum)```). So, what is so bad about this? Sure, more code usually implies more bugs, and it does seem that we’re doing a lot of work to get the sum of a list, but the big problems are subtler.

The major fault is that we have to know an awful lot about the underlying data structure in order to iterate over it. We must know there is a ```Length``` method exposed on the values object. We must also know that it exposes an indexer that allows us to extract the individual ```values[i]```. We have become dependent on the structure of the data.

We also carry the mental burden of dealing with the incrementation at a low level. Are we *sure* we should be iterating until the index is *less-than* the length, or should it be *less-than-or-equal*? Do we know that the list of values starts at zero? We are fairly confident we got it right, but it would be much preferred to not have deal with these questions at all. What if we raise the level of abstraction?

C# has a high level way of iterating through lists called [enumerators](https://msdn.microsoft.com/en-us/library/78dfe2yb(v=vs.110).aspx) which are an implementation of the [iterator pattern](https://en.wikipedia.org/wiki/Iterator_pattern). I won’t go into any detail of iterators here. It’s enough to know that iterators are built into C# , are available on all types of lists, and benefit from a specialized loop syntax known as ```foreach```. It looks like this.

```csharp
int sum = 0;
foreach (var value in values)
{
    sum += value;
}
Console.WriteLine(sum);
```

As you can see, we no longer need to know *anything* about the values object. How do we get the value out of the list? We don’t care! Is this declarative code? Well, the foreach statement is declarative, but we are still very specific as to how we want to sum the list by adding a value to the sum one at a time. We can surely do better. How’s this?

```csharp
Console.WriteLine(Sum(values));
```

But that’s cheating, right? All we are doing is calling a function called ```Sum``` and using its result as input to the ```WriteLine``` function. We don’t even know how ```Sum``` is implemented! True. But do we care? What matters is that this particular code has gone from imperative to declarative. We don’t need to know how to sum a list, just that we want it summed. If this feels too much like cheating, it’s probably because the ```Sum``` function is very specific. Does this mean that we need to write helper functions for everything just to make our code declarative?

Yes and no. Writing a specific function is often a good way to replace a comment with a descriptive function name, but it’s not always preferred. Luckily for us, functional programming at its core is about making general functions and combining them to solve specific problems. Follow along and I’ll show you how, and in the end, we’ll get back to this very example. But first, let’s learn more about what functional programming is all about and why it’s worth your time. [Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})