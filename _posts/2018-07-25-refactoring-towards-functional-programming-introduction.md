---
layout: single
title: "Refactoring towards functional programming - Introduction"
tags:
   - .NET
   - Functional programming
---

How to refactor code towards functional programming from the ground up.

## Introduction

This series of blog posts is meant to give you, the inquisitive reader, a practical introduction to functional programming through the use of common list operations. No prior knowledge of functional programming is required but some familiarity with programming in general is assumed. The language used in the example code is C# while the principles taught are language agnostic. The motivation behind these posts is to introduce functional programming through concrete examples with clear benefits. I’ll skip most FP jargon but won’t shy away from complexity. Functional programming has a high barrier of entry, not because it’s inherently harder than object-oriented programming, but because it is unfamiliar. We’ll jump off the deep end with higher order functions and functional composition, but in the end, we’ll have made an API you might be familiar with. You may be surprised to find that you have been doing some functional programming all along.

### Imperative versus declarative programming

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

Yes and no. Writing a specific function is often a good way to replace a comment with a descriptive function name, but it’s not always preferred. Luckily for us, functional programming at its core is about making general functions and combining them to solve specific problems. Follow along and I’ll show you how, and in the end, we’ll get back to this very example. But first, let’s learn more about what functional programming is all about and why it’s worth your time.

## Functional programming heuristics

Functional programming is a *paradigm* and is not tied to any specific language. Then what are functional languages? They are simply languages which make it easy to follow the paradigm and hard to go against it. Few if any high-level languages are strictly object-oriented or procedural and many are multi-paradigm. C# is such a language. It leans towards the object-oriented side but enables functional programming. Then what does it mean to be functional?

Somewhat simplified, object-oriented programs couples data and behavior in objects. Functional programming separates the two. While object-oriented programs often model the world as a set of *things*, functional programming concerns *processes*. Data flows through functions which are composed into larger flows.

Does it sound awfully vague? Let’s break it down into two practical concepts which are core ideas in functional programming but are generally useful. These are **pure functions** and **immutability**.

### Pure functions

In order to build larger workflows out of smaller functions, we need functions we can *trust* to only do what they claim to do, and to not have any side effects. Side effects are anything that affects the world around the function in any way - such as performing network requests, IO, or writing to the console. Modifying state outside the function is also a side effect. Functions without side effects are known as *pure functions*. Here’s an impure function that writes a name to the console.

```csharp
static void WriteFullName(string firstName, string lastName)
{
    var fullName = $"{firstName} {lastName}”;
    Console.WriteLine(fullName);
}
```

Why is it impure? Concatenating the first and last names is ok but writing to the console is a side effect. You might scoff and say that this isn’t that big of a deal, but there are real issues with the code.

The first thing that comes to mind is **how would you write a test for this function?** There is no return value, and no simple way to observe what was written out. You can test it manually easily enough but writing an automated test would be hard. The clue is the ```void``` keyword. A function without a return value cannot be a pure function.

Furthermore, **how would you use this function in another context?** What if you want to write the name to a file or send it in response to an HTTP-request. You’d either have to copy-paste the concatenating code, or refactor it towards [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)er, modular code. Truth be told, this function is simply doing *too much work*. As it turns out, we’re not adhering to the [single responsibility principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html). What if separate the concatenation from the printing to console?

```csharp
static string FullName(string firstName, string lastName) =>
    "{firstName} {lastName}”;
```

We’ve made the function so simple we can write it as an expression bodied method. Now, it’s up the consumer of the function to decide how to use it. It can either be composed into a larger flow or be used to produce a side effect. To write the full name to the console we pass the output of the ```FullName``` function as input to ```Console.WriteLine```.

```csharp
Console.WriteLine(FullName("Willy", "Wonka"));
```

I hope this convinced you that there’s a benefit to converting your functions into pure functions, even if they may seem benign.

You should also be aware that we cannot rid ourselves completely of side effects, after all – they are what actually *does* something in the program. If we never write any output or read any input, the program would be useless. What’s important is to *limit the scope of side effects* or *isolate* them from the pure functions. The reward is testable, more reusable code.

### Immutability

If you’re used to object-oriented code, you’re probably used to mutating state. You create an object and call methods that modify state on the object. In functional programming, data is sparsely if ever modified.

When working with objects, this means we create new objects instead of modifying the existing ones. The same goes for lists. When we add an object to a list, we create a new list with the new item added. You’ll see many examples of this throughout this series.

The reward for immutability is code that offers fewer surprises since don’t have to wonder where updates come from if nothing ever updates. Immutability also enables code to run in parallel. Every aspect of immutability is a big topic of its own and won’t be covered in detail. Again, the important thing is to change your mindset and strive for it where possible.

In the beginning of this post I stated that C# is a multi-paradigm language that leans towards the object-oriented side. C# is built to make mutation easy and immutability often requires special syntax and specialized data structures. Even variables are meant to be, well, variable.

```csharp
var age = 42;
Thread.Sleep(new TimeSpan(365, 0, 0, 0));
age = 43;
```

C# offers no language support to make them immutable. We can turn the age variable into a field and mark it as ```readonly``` but that increases its scope. Unfortunately, sometimes our best bet to achieve immutability by *convention* - we simple don’t change things.

## Introduction to the example problem

In the following posts we’ll take some run-of-the-mill imperative code and refactor it towards a functional style. Our example is a short snippet to help us choose the healthy alternative from a diners menu.

```csharp
foreach (var dish in menu)
{
    var calories = dish.Calories;

    if (calories < 400)
    {
        Console.WriteLine(dish.Name);
    }
}
```

The dish class is just a simple bag of data. The getter only fields prevent modification after creation.

```csharp
public class Dish
{
    public string Name { get; }
    public int Calories { get; }

    public Dish(string name, int calories)
    {
        Name = name;
        Calories = calories;
    }
}
```

The menu is just a few dishes with a couple of healthy options.

```csharp
var menu = new List<Dish>
    {
        new Dish("Pasta", 400),
        new Dish("Sallad", 200),
        new Dish("Pizza", 1600),
        new Dish("Onion soup", 300)
    };
```

The output to console should be:
```
Sallad
Onion soup
```

There are no hidden errors in the code – it really is as straight forward as it seems! I hope that you’ve written similar code before. Now let’s make it functional!