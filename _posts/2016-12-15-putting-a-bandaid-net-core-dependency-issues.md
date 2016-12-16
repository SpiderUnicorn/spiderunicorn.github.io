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

This is a short summary of dependency related issues I've faced with running .NET Core together
with a .NET 4.x in a single, monolithic visual studio solution. In a recent greenfield
project, my team and I opted for ASP.NET Core due its immense performance 
improvement over ASP.NET 4.x as well as some other benefits that seemed 
reasonable at the time. However, we still wanted to use the parts of the 'old trusty' 
.NET for parts that needed Entity Framework 6 and some other dependencies not yet
ported to .NET Core. 

*Enter dependency issues.*

## Problems with mixing .NET Core and .NET 4.x

.NET 4.x is a very stable platform in and of itself. .NET Core is slowly maturing
as well, and works fine for the most part. However, when you mix .NET 4.x with Core
in the same solution, problems quickly surface. Most of the problems stem from the
tooling around .NET Core still being half-baked and in flux. The first problem we encountered
was adding references between .NET Core projects and .NET 4.x projects and promptly
getting greeted by: 

>Unable to resolve 'xxx' for '.NETFramework,Version=v4.6.2'.

We set the project up in a way that *should* work, but the dependencies still couldn't
be resolved. The naïve solution would be to run *dotnet restore* to resolve everything in the solution. 
This didn't work well, however, since at the time we worked on the project "dotnet restore" 
only worked on .NET Core projects
and it couldn't infer a complete dependency graph.
The solution was to run the restore through the latest version of nuget.exe and have
it successfully resolve both the .NET Core and NET 4.x dependencies. One caveat is that
this only works with version 3.5 or above of nuget.exe.

### Fixing unable to resolve...
Download the latest version of nuget.exe and run **nuget restore**
[Go here to download the latest version](https://dist.nuget.org/win-x86-commandline/latest/nuget.exe).

## Assembly reference manifest mismatch 

Some time into to the project, we started getting this message: 

>The located assembly's manifest definition does not match the assembly reference.

As far as I can tell, this is caused by stale dependency references being kept after
some version of a dependency has been updated. Restoring the packages per instructions
above didn't solve the issue for us. We found that sometimes references were kept in
**project.lock** files, despite being updated in project.json. Likewise, the **bin** and **obj**
catalogues would sometimes need to be cleared completely, which visual studios *clean* command
doesn't do a very good job with.

This all seems straight forward enough, but we also faced issues with old dependencies being cached
by NuGet, so the NuGet cached had to be cleared.

### Solving manifest mismatch errors

Remove all cached and generated files. Remove project.lock & project.fragment files,
as well as bin & obj folders. Run *nuget.exe locals all -clear* from cmd. 

## Summary
During the course of the project, we put together a list of steps to perform
when problems ocurred. The steps were as follows:

- Close visual studio
- Delete bin and obj folders
- Delete project.lock.sjon, project.fragment.json, project.fragment.lock.json
- Open cmd and run: nuget.exe locals all -clear

This list was eventually turned into a script which automated all of the steps 
above. You can find it [here](https://github.com/SpiderUnicorn/powershell-utilities/blob/master/dotnet/clean-solution-and-clear-nuget-cache/clean_and_clear_cache.ps1).
As of writing this post, we've saved many dull hours by using it. I hope
you'll find it useful as well. 