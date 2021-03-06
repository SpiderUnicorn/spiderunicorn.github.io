---
layout: single
title: "Readable test results with xUnit.net"
tags: 
   - .NET Core
   - Testing
   - xUnit
---

How to configure xUnit.net using settings and custom attributes to achieve readable test output.

## What is the problem?

xUnit is an excellent testing framework with a few caveats when it comes to its default behavior. One of its less-than-stellar features is the way it outputs test results as *namespace.className.methodName*. This has its use in that it makes it easy to know where the test is located. However, I never find this to be a problem anyway since the test explorer in visual studio makes it trivial to navigate to a test by just clicking on it, and any other editor has useful navigation and search tools too. 
 
### So if knowing where a failing test is located isn’t important, what is?

My most desired output is not knowing which method failed, but knowing what piece of functionality failed. Good tests test for business value and are given a name to indicate in what way. Some people tend to tie the name of their test to the method it is testing such as this

```csharp
public void classUnderTest_methodUnderTest()
```

the output from xUnit would then be: *namespace.className.classUnderTest_methodUnderTest*. I won’t go into a long discussion on naming as it’s not really what this post is about. In brief, I prefer to name tests along the lines of

```csharp
public void When_that_happens_it_should_do_this()
```

This is all highly subjective and there are many ways to name tests that all are good in their own way. A common trait of good tests is that they test for behavior and are oblivious of code structure. 
Writing tests in this style produces the slightly more readable output *namespace.className.methodName.When_that_happens_it_should_do_this*. Still, there is a lot of noise that we could do without. 

## Creating readable output 

What xUnit lacks in default conventions it makes up for by being straight forward to customize and extend. I won’t go into detail on how to work with .NET core CLI, but I will provide some examples of console output as we go along. I’ll detail the steps I take here only for completeness. Feel free to skip ahead if you don’t care about my specific example. 

### Setting up a sample project

To begin, let’s start from a clean slate by creating a new test project from the command line by utilizing the xUnit template. 

```
$ dotnet test xUnit --framework netcoreapp1.1
$ dotnet restore
```

Then we run the tests and observe the output

```
$ dotnet test 
```

This should give an output similar to:

```
Starting test execution, please wait...
[xUnit.net 00:00:01.3461910]   Discovering: xUnit_demo
[xUnit.net 00:00:01.5770306]   Discovered:  xUnit_demo
[xUnit.net 00:00:01.6750701]   Starting:    xUnit_demo
[xUnit.net 00:00:02.0331776]   Finished:    xUnit_demo

Total tests: 1. Passed: 1. Failed: 0. Skipped: 0.
Test Run Successful.
Test execution time: 3,2711 Seconds
```

As we can see, the xUnit project template ships with a sample test, but we don’t know what the test is, only that is passes. To see the name of the test, we need to add an argument to the command.

```
$ dotnet test –-list-tests
```

```
The following Tests are available:
[xUnit.net 00:00:01.0693821]   Discovering: xUnit_demo
[xUnit.net 00:00:01.2909779]   Discovered:  xUnit_demo
    xUnit_demo.UnitTest1.Test1
```

Note that this doesn’t actually run the test, but we can see the name that xUnit produces with the format *namespace.className.methodName*, or in this case **xUnit_demo.UnitTest1.Test1**. This would be same name written to the test explorer in visual studio.

Now lets move on to configuring xUnit for better readability.

## Configuring xUnit

The default output of *namespace.className.methodName* can be shortened to just *methodName* by configuring the test project. For desktop and PCL projects you’d need an [App.config](https://xUnit.github.io/docs/configuring-with-xml.html) file. For dotnet core, add an **xunit.config.json** file.

A good starting point is to add diagnostics to the test output. Put the following into the **xunit.config.json** file. Note that this has no bearing on the output we are working on, it's just nice to have.

```json
{
  "diagnosticMessages": true
}
```

Next, make sure the file is included in the build. If you are using visual studio, right click on the file and edit properties. Set “Copy to Output Directory” to "Copy if newer". If you are using any other editor. Open the **csproj** file and add the following.

```xml
 <ItemGroup>
    <None Include="xunit.runner.json">
      <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
    </None>              
  </ItemGroup>
```

Now look at the output.

```
$ dotnet test –-list-tests
```

```
The following Tests are available:
[xUnit.net 00:00:00.9726020]   Discovering: xUnit_demo (method display = ClassAndMethod)
[xUnit.net 00:00:01.1809032] xUnit_demo: Discovered test case 'xUnit_demo.UnitTest1.Test1' (ID = '2147c12097450ec4683f1752d79ec8e5460f4689', VS FQN = 'xUnit_demo.UnitTest1.Test1')
[xUnit.net 00:00:01.1856495]   Discovered:  xUnit_demo (running 1 test case)
    xUnit_demo.UnitTest1.Test1
```

On the first line, we see something interesting. We see that method display is set to *ClassAndMethod*. As it turns out, this is a setting we can change. As of writing this, the available options are *ClassAndMethod* which is the default, and then there is *Method*, which will strip the namespace and class information from the output.

Edit **xunit.runner.json** to include

```json
{
  "diagnosticMessages": true,
  "methodDisplay": "method"
}
```

Running the test command again should produce the following: 

```
[xUnit.net 00:00:01.3235469]   Discovering: xUnit_demo (method display = Method)
[xUnit.net 00:00:01.5804080] xUnit_demo: Discovered test case 'Test1' (ID = '2147c12097450ec4683f1752d79ec8e5460f4689', VS FQN = 'xUnit_demo.UnitTest1.Test1')
[xUnit.net 00:00:01.5850655]   Discovered:  xUnit_demo (running 1 test case)
    Test1
```

We have now gone from the test being displayed as *xUnit_demo.UnitTest1.Test1* to simply *Test1*. But we can do better!

## Custom attributes

First, let’s change the test a bit and rename it to *When_that_happens_it_should_do_this*. Open **Test1.cs** and change the name if you are following along. Wouldn't it be sweet if this could be displayed without the pesky underscores? I suspect my human readers would agree.

To gain full control of what is displayed, we need to move away from the standard FactAttribute and create our own. Let’s create a new class, **Spec.cs** and add the following:

```csharp
using xunit;
using System.Runtime.CompilerServices;

public class Spec : FactAttribute {

    public Spec([CallerMemberName] string testMethodName = "")
    {
        DisplayName = testMethodName.Replace("_", " ");
    }
}
```

The Spec class extends xUnits standard *FactAttribute*. In the constructor, we use the *CallerMemberNameAttribute* to get the name of the calling method, which in this case will be the test we annotate with the attribute. Note that the a default value is required, which is why we specify that **testMethodName = ""**. 

In the body of the constructor we call the Replace method on the testMethodName string to replace all underscores with spaces. We assign it to a property called **DisplayName**, which is one of two properties that can be called on the attribute, the other being **Skip** (which is used to skip a test from running). Internally, xUnit will use any value provided to the **DisplayName** property and use that to generate test results. See the end of this post for more on DisplayName. 

Now, change **UnitTest1.cs** to use the new attribute.

```csharp
[Spec]
public void When_that_happens_it_should_do_this()
{

}
```

And at last, we can list the tests again.

```
$ dotnet test –-list-tests
```

```
[xUnit.net 00:00:01.0327560]   Discovering: xUnit_demo (method display = Method)
[xUnit.net 00:00:01.2498380] xUnit_demo: Discovered test case 'When that happens it should do this' (ID = '36bc79c08111574998da4f7abfc848f369c324c9', VS FQN = 'xUnit_demo.UnitTest1.When_that_happens_it_should_do_this')
[xUnit.net 00:00:01.2538312]   Discovered:  xUnit_demo (running 1 test case)
    When that happens it should do this
```

And voilà! On the last line, we finally see our desired output. Again, this will also be included in the visual studio test explorer. Working with the custom attribute is straightforward. You can use it like you would any other function, just remember to put the CallerMemberNameAttribute last in the constructor arguments. Play around with it and find an implementation that suits your needs.

## But what about Display Name?

As you saw, we simply set a new value to the xUnit FactAttribute property **DisplayName**. As it turns out, you can call this property without creating a new attribute.

```csharp
[Fact(DisplayName = "When that happens it should do this")]
public void When_that_happens_it_should_do_this()
```

This would give us the exact same results as going the above route, but with seemingly little effort. I recommend you only use DisplayName if you have a very few methods that you think needs a touch of a readability. Still, I think it’s better to create an attribute in almost all cases where you’d like to change the way a method is displayed. In my experience, plain text strings require a lot more effort to maintain and are downright dangerous. Should you rename the test or alter its behaviour without changing the DisplayName your own test will lie to you.
