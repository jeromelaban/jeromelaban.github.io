---
layout: post
title: "INotifyPropertyChanged with C# 9.0 Source Generators"
date: 2020-04-29 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "C#", "Source Generation", "Roslyn"]
author: Jerome
---

In a design meeting far, far away, source generators were [designed to be part of C# 6.0](https://github.com/dotnet/roslyn/blob/12bd769ebcd3121b88f535e8559f5a42d9c0e873/docs/features/generators.md), but sadly never came to be. At the time, wanting that source generation feature pretty badly, I went on implementing the specification which later became [Uno.SourceGeneration](https://github.com/unoplatform/Uno.SourceGeneration), and it turns out it was the right decision to stick with a very similar API.

I could port the [INotifyPropertyChanged (INPC) generator I built a while back](https://jaylee.org/archive/2019/12/08/roslyn-sourcegeneration-reborn-replace-inotifypropertychanged.html) that uses Uno.SourceGeneration tasks to use Roslyn's [shiny new C# 9.0 feature](https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/) in a matter of minutes. Amazing!

<!-- more -->

## The INPC Generator

As a quick refresher, the generator works by allowing the creation of a class this way:

```csharp
partial class MyClass
{
  [GeneratedProperty]
  private string _myProperty;

  partial void OnMyPropertyChanged(string previousValue, string newValue)
  {
    Console.WriteLine($"OnMyPropertyChanged({previousValue}, {newValue}");
  }
}
```

You'll notice here that the property is not visible at all, and it will be assumed that the generator will recognize the `GeneratedPropertyAttribute` and do the rest of the work for the author of the class.

The generator could then produce something like this:

```csharp
partial class MyClass : INotifyPropertyChanged
{
  public string MyProperty
  {
    get => _myProperty;
    set
    {
      var previous = _myProperty;
      _myProperty = value;
      PropertyChanged?.Invoke(new PropertyChangedEventArgs(nameof(MyProperty)));
      OnMyPropertyChanged(previous, value);
    }
  }

  partial void OnMyPropertyChanged(string previousValue, string newValue);
}
```

This generator is generating the property on your behalf, and automatically implements the `INotifyPropertyChanged` as well as a partial method that you can optionally implement to get local notification on changes.

For a more detailed explanation of the generation process, [head to this earlier article](https://jaylee.org/archive/2019/12/08/roslyn-sourcegeneration-reborn-replace-inotifypropertychanged.html).

## Upgrading the source generator to C# 9.0

First, we'll need the [INPC.Generator Sample Project](https://github.com/jeromelaban/inpc.generator) to use the the C# 9.0 preview:

```xml
<PropertyGroup>
  <LangVersion>preview</LangVersion>
</PropertyGroup>
```

Then change the NuGet packages from Uno.SourceGeneration to Roslyn's Analyzers:

```xml
<ItemGroup>
	<PackageReference Include="Microsoft.CodeAnalysis.CSharp.Workspaces" Version="3.6.0-3.20207.2" PrivateAssets="all" />
	<PackageReference Include="Microsoft.CodeAnalysis.Analyzers" Version="3.0.0-beta2.final" PrivateAssets="all" />
</ItemGroup>
```

Then we'll need to adjust the source code to be use the new interfaces: 

```csharp
[Generator]
public class MySourceGenerator : ISourceGenerator
{
    public void Initialize(InitializationContext context)
    {
        // No initialization required for this one
    }

    public void Execute(SourceGeneratorContext context)
    {
        // Generation
    }
}
```

The execute method in Uno.SourceGeneration has the same signature, and the new `SourceGeneratorContext` type the only property that really matters `Compilation`.

We'll need to update the path to the binary, where `SourceGenerator` becomes `Analyzer`, in the `INPC.Generator.props` file:

```xml
<ItemGroup>
    <Analyzer Include="$(MSBuildThisFileDirectory)..\bin\$(Configuration)\netstandard2.0\INPC.Generator.dll"
            Condition="Exists('$(MSBuildThisFileDirectory)..\bin')" />
    <Analyzer Include="$(MSBuildThisFileDirectory)..\tools\INPC.Generator.dll"
            Condition="Exists('$(MSBuildThisFileDirectory)..\tools')" />
</ItemGroup>
```

And finally, we simply need to compile the main program with the C# 9.0 preview.

When running the sample, here's what happens:

```
OnIntPropertyChanged(0,42)
OnIntPropertyChanged(,My 42)
```

The job is done!

## In closing

I hope the feature will grow, and provide lots more like access to msbuild properties, or [dependencies between generators](https://github.com/unoplatform/Uno.SourceGeneration#general-guidelines-for-creating-generators), and it will [finally fix the issues with dynamically generated](https://developercommunity.visualstudio.com/content/problem/588021/the-compile-itemgroup-intellisense-cache-is-not-re.html) `<Compile />` item groups.

I hope you will give this feature a try, as it opens up a world of possibilities, as we've been finding out while developing Uno (Here's the list of generators), as well as the [Uno.CodeGen generators](https://github.com/unoplatform/Uno.CodeGen) (which some may not be needed anymore with the C# 9.0 records feature!).

Happy source-generation!