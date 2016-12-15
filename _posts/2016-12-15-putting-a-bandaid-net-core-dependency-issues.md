---
layout: single
title: "Putting a band aid on .NET Core dependency issues"
tags: 
   - .NET 
   - .NET Core
   - Powershell
---

>**TL;DR** Problems with .NET Core? [Download this script](https://github.com/SpiderUnicorn/powershell-utilities/blob/master/dotnet/clean-solution-and-clear-nuget-cache/clean_and_clear_cache.ps1). Run in your solution folder. What could go wrong?

## Background 

This is a short tale of the troubles I've faced with running .NET Core together
with a .NET 4.x in a single, monolithic solution file. In a recent greenfield
project, my team and I opted for ASP.NET Core due its immense performance 
improvement over ASP.NET 4.x as well as some other benefits that seemed 
reasonable at the time. However, we still wanted to use the 'old' .NET for parts 
of the project to use Entity Framework 6 and some other dependencies not yet
ported to .NET Core. 

*Enter dependency issues.*

## Problems with mixing .NET Core and .NET 4.x

.NET 4.x is a very stable platform in and of itself. .NET Core is slowly maturing
as well, and works fine for the most part. However, when you mix .NET 4.x with Core
in the same solution, problems quickly surface. Most of the problems stem from the
tooling around .NET Core still being in beta. The first problem we encountered
was adding references between .NET Core projects and .NET 4.x projects. The first
thing to overcome was this:

>Unable to resolve 'xxx' for '.NETFramework,Version=v4.6.2'.

The naive solution would be to run *dotnet restore* to resolve everything in the solution. 
This didn't work well, however, since at the time we worked on the project "dotnet restore" 
only worked on .NET Core projects
and it couldn't infer a complete dependency graph to construct the .NET Core projects in.

### Fixing unable to resolve...
The solution to the problem was to run the restore through the latest version of nuget.exe and have
it successfully resolve both the .NET Core and NET 4.x dependencies. One caveat was that the
this only works with version 3.5 or above of nuget.exe. You can 
[go here to download the latest version](https://dist.nuget.org/win-x86-commandline/latest/nuget.exe).

## Summary
During the course of the project, we put together a list of all the problems
we encountered. The steps were as follows:

- Close visual studio
- Delete bin and obj folders
- Delete project.lock.sjon, project.fragment.json, project.fragment.lock.json
- Open cmd and run: nuget.exe locals all -clear

This list was eventually turned into a script which automated all of the steps 
above. You can find it [here](https://github.com/SpiderUnicorn/powershell-utilities/blob/master/dotnet/clean-solution-and-clear-nuget-cache/clean_and_clear_cache.ps1).
As of writing this post, we've saved many dull hours by using it. I hope
you'll find it useful as well. 