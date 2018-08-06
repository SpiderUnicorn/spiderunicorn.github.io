---
layout: single
title: "Writing a filter function"
tags:
   - .NET
   - Functional programming
---

**How to make a filter method to work with lists from the ground up.**

> This is part of a series on refactoring towards functional programming.

---
[Introduction]({% post_url 2018-07-25-refactoring-towards-functional-programming-introduction %})  
&nbsp;&nbsp;[Imperative versus declarative programming]({% post_url 2018-07-26-imperative-versus-declarative-programming %})  
&nbsp;&nbsp;&nbsp;&nbsp;[Functional principles for better code]({% post_url 2018-07-27-functional-principles-for-better-code %})  
[Designing a functional list API]({% post_url 2018-08-04-designing-a-functional-list-api %})  
&nbsp;&nbsp;&nbsp;&nbsp;**[Writing a filter function]({% post_url 2018-08-06-writing-a-filter-function %})**  
&nbsp;&nbsp;&nbsp;&nbsp;Transforming with Map  
&nbsp;&nbsp;&nbsp;&nbsp;Isolating side effects with ForEach  
Functional composition  
&nbsp;&nbsp;&nbsp;&nbsp;Functional composition through extension methods  
&nbsp;&nbsp;&nbsp;&nbsp;Refactoring to point free  
Functional refactoring conclusions  

---

Now it’s time to split functionality up into the three parts previously mentioned - filtering, transforming and performing the side-effect of printing to the console. In this step we’ll be doing the filtering. First we need to pull the if statement out, and keeping with immutability, we need to make a new menu that is filtered. This -

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

Is refactored into this -

```csharp
var lowCalorieMenu = CaloriesLowerThan(menu, 400);

foreach (var lowCalorieDish in lowCalorieMenu)
{
    Console.WriteLine(lowCalorieDish.Name);
}
```

As a first step, we opted to go for a helper method that is named appropriately for our domain. As you can see, the work done in the foreach loop is suddenly much simpler. The implementation of the ```CaloriesLowerThan``` method looks something like this.

```csharp
public IEnumerable<Dish> CaloriesLowerThan(IEnumerable<Dish> menu, int limit)
{
    List<Dish> lowCalorieDishes = new List<Dish>();

    foreach (var dish in menu)
    {
        if (dish.Calories < limit)
        {
            lowCalorieDishes.Add(dish);
        }
    }
    return lowCalorieDishes;
}
```

As you can see, it contains much the same structure that was removed from the original code. The method does the job of filtering out the high calorie dishes, but what if we wanted to filter based on another condition? What if we wanted all the dishes with short names?

```csharp
public IEnumerable<Dish> NamesShorterThan(IEnumerable<Dish> menu, int limit)
{
    List<Dish> shortNamedDishes = new List<Dish>();

    foreach (var dish in menu)
    {
        if (dish.Name.Length < limit)
        {
            shortNamedDishes.Add(dish);
        }
    }
    return shortNamedDishes;
}
```

The similarity is striking. The only thing that differs is the names of some variables and the condition. Can we generalize it?

The way to generalize filtering is to plug the ```if``` statement by using what is known as a predicate function. We pass a predicate function into the filtering function, and each element that satisfies the predicate is added to the list. Since we are making it more general, we also change the name to ```filter``` to better capture the concept. It looks like this.

```csharp
public IEnumerable<Dish> Filter(IEnumerable<Dish> menu, Func<Dish, bool> predicate)
{
    List<Dish> filteredDishes = new List<Dish>();

    foreach (var dish in menu)
    {
        if (predicate(dish))
        {
            filteredDishes.Add(dish);
        }
    }
    return filteredDishes;
}
```

We can go one step further by utilizing [C# generics](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/generics/) to make the filter work on any type, not just dishes.

```csharp
public IEnumerable<T> Filter<T>(IEnumerable<T> ts, Func<T, bool> predicate)
{
    List<T> filtered = new List<T>();

    foreach (var t in ts)
    {
        if (predicate(t))
        {
            filtered.Add(t);
        }
    }
    return filtered;
}
```

Here it is used in the menu application.

```csharp
var lowCalorieMenu = Filter(menu, dish => dish.Calories < 400);

foreach (var lowCalorieDish in lowCalorieMenu)
{
    Console.WriteLine(lowCalorieDish.Name);
}
````

As you can see, we now have to pass a function to the filter method as a predicate to plug in the filter condition. It works wonderfully, and it’s a general enough function that we can use it for any type, on any [IEnumerable](https://msdn.microsoft.com/en-us/library/system.collections.ienumerable(v=vs.110).aspx).

The possible downside is that we lose out on the descriptive name. We’ll look at ways to increase readability later, but first, we’ll define our next functional abstraction - the map function.
