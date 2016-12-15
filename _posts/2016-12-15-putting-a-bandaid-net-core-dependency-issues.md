---
layout: single
title: "Putting a band aid on .NET Core dependency issues"
---
>**TL;DR** Problems with .NET Core? [Download this script](https://github.com/SpiderUnicorn/powershell-utilities/blob/master/dotnet/clean-solution-and-clear-nuget-cache/clean_and_clear_cache.ps1). Run in your solution folder. What could go wrong?

## Background 

In my current project, we opted for ASP.NET Core due its immense performance 
improvement over ASP.NET 4.x as well as some other benefits that seemed 
reasonable at the time. However, we still wanted to use the 'old' .NET for parts 
of the project to use Entity Framework 6 and some other dependencies not yet
ported to .NET Core. 

*Enter dependency issues.*

## Problems with mixing .NET Core and .NET 4.x

Some common problems we had in the beginning of the project were due to making 
references between .NET Core projects and .NET 4.x projects. Here's one of the
errors we got.

>Unable to resolve 'xxx' for '.NETFramework,Version=v4.6.2'.

All the more confusing, this error seemed to pop up on the build server after
we stopped getting it locally. An issue was that we couldn't resolve
dependencies with "dotnet restore" on the solution level, because it couldn't
resolve dependencies for 4.x projects. 

### The fix
The solution to the problem was to run the restore through nuget.exe and let
it resolve both the .NET Core and NET 4.x dependencies. One caveat is that the
you'll need version 3.5 or above of nuget.exe. You can 
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