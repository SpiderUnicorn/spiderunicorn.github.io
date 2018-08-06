---
layout: single
title: "Functional principles for better code"
tags:
   - .NET
   - Functional programming
---

**How using basic principles from functional programming helps write testable, reusable code.**

> This is part of a series on refactoring towards functional programming.

---
[Introduction]({% post_url 2018-07-25-refactoring-towards-functional-programming-introduction %})  
&nbsp;&nbsp;[Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})  
&nbsp;&nbsp;&nbsp;&nbsp;**[Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})**  
[Designing a functional list API]({% post_url 2018-08-04-designing-a-functional-list-api %})  
&nbsp;&nbsp;&nbsp;&nbsp;[Writing a filter function]({% post_url 2018-08-06-writing-a-filter-function %})  
&nbsp;&nbsp;&nbsp;&nbsp;Transforming with Map  
&nbsp;&nbsp;&nbsp;&nbsp;Isolating side effects with ForEach  
Functional composition  
&nbsp;&nbsp;&nbsp;&nbsp;Functional composition through extension methods  
&nbsp;&nbsp;&nbsp;&nbsp;Refactoring to point free  
Functional refactoring conclusions  

---

Functional programming is a *paradigm* and is not tied to any specific language. Then what are functional languages? They are simply languages which make it easy to follow the paradigm and hard to go against it. Few if any high-level languages are strictly object-oriented or procedural and many are multi-paradigm. C# is such a language. It leans towards the object-oriented side but enables functional programming. Then what does it mean to be functional?

Somewhat simplified, object-oriented programs couples data and behavior in objects. Functional programming separates the two. While object-oriented programs often model the world as a set of *things*, functional programming concerns *processes*. Data flows through functions which are composed into larger flows.

Does it sound awfully vague? Let’s break it down into two practical concepts which are core ideas in functional programming but are generally useful. These are **pure functions** and **immutability**.

### Pure functions

In order to build larger workflows out of smaller functions, we need functions we can *trust* to only do what they claim to do, and to not have any side effects. Side effects are anything that affects the world around the function in any way - such as performing network requests, IO, or writing to the console. Modifying state outside the function is also a side effect. Functions without side effects are known as *pure functions*. Here’s an impure function that writes a name to the console.

```csharp
static void WriteFullName(string firstName, string lastName)
{
    var fullName = $"{firstName} {lastName}";
    Console.WriteLine(fullName);
}
```

Why is it impure? Concatenating the first and last names is ok but writing to the console is a side effect. You might scoff and say that this isn’t that big of a deal, but there are real issues with the code.

The first thing that comes to mind is **how would you write a test for this function?** There is no return value, and no simple way to observe what was written out. You can test it manually easily enough but writing an automated test would be hard. The clue is the ```void``` keyword. A function without a return value cannot be a pure function.

Furthermore, **how would you use this function in another context?** What if you want to write the name to a file or send it in response to an HTTP-request. You’d either have to copy-paste the concatenating code, or refactor it towards [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)er, modular code. Truth be told, this function is simply doing *too much work*. As it turns out, we’re not adhering to the [single responsibility principle](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html). What if separate the concatenation from the printing to console?

```csharp
static string FullName(string firstName, string lastName) =>
    "{firstName} {lastName}";
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

Next, we'll look at the design considerations for [creating a functional list API]({% post_url 2018-08-04-designing-a-functional-list-api %}).