---
layout: single
title: "Designing a functional list API"
tags:
   - .NET
   - Functional programming
---

**The basics of functional pipelines.**

> This is part of a series on refactoring towards functional programming.

---
[Introduction]({% post_url 2018-07-25-refactoring-towards-functional-programming-introduction %})  
&nbsp;&nbsp;[Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})  
&nbsp;&nbsp;&nbsp;&nbsp;[Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})  
**[Designing a functional list API]({% post_url 2018-08-04-designing-a-functional-list-api %})**  
&nbsp;&nbsp;&nbsp;&nbsp;[Writing a filter function]({% post_url 2018-08-06-writing-a-filter-function %})  
&nbsp;&nbsp;&nbsp;&nbsp;Transforming with Map  
&nbsp;&nbsp;&nbsp;&nbsp;Isolating side effects with ForEach  
Functional composition  
&nbsp;&nbsp;&nbsp;&nbsp;Functional composition through extension methods  
&nbsp;&nbsp;&nbsp;&nbsp;Refactoring to point free  
Functional refactoring conclusions  

---

Let’s look at the example problem again.

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

If we break it down, we’re doing four things inside the loop. We pick the calories, check if they are lower than the calorie limit (400), pick out the name and we write it out. It seems that there are three types of operations happening. *Checking a condition*, *getting something* and doing a *side-effect*. What we want to do is find suitable functional abstractions that work with these three operations. We also need them to be *immutable*, which means we need to create a new list after every operation since we cannot modify the original in any way.

When thinking of list-operations in a functional way, we often picture iteration as values flowing down a stream, changing along the way into a new form. This is often called a *pipeline*.

![Pipeline]({{ "/assets/calorie_pipeline.png" | absolute_url }})

The first two steps in the pipeline are common operations in functional programming known as filtering and mapping. The third step is out of bounds of functional programming, but we’ll still implement it in the pipeline in a function, as a place to put side-effects. If we can’t rid ourselves of side-effects, we *isolate* them.

Here is the pipeline again with the three parts categorized.

![Pipeline]({{ "/assets/calorie_pipeline_described.png" | absolute_url }})

Now it’s time to create a small class of higher-order functions to help us refactor the code towards functional thinking. We’ll stick to our principles - the functions should be pure, and they should be immutable. First, we’ll do the [filtering]({% post_url 2018-08-06-writing-a-filter-function %}).