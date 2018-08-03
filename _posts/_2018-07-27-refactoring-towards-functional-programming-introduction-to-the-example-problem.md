---
layout: single
title: "Refactoring towards functional programming"
subtitle: "Introduction to the example problem"
tags:
   - .NET
   - Functional programming
---

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