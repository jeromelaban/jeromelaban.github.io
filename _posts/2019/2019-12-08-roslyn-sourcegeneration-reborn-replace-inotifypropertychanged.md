---
layout: post
title: "Roslyn Source Generation Reborn, the replace keyword and INotifyPropertyChanged"
date: 2019-12-08 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "C#", "Source Generation"]
author: Jerome
---

A very long time ago, during the C# 6.0 time frame, a Source Generation proposal was added to the list of possible features, it was abandoned, but in a recent PR, the Roslyn team [is taking a look again](https://github.com/dotnet/roslyn/pull/40162) at the feature proposal, as there are lots of generation happen around Microsoft that could benefit from an integrated story.

<!-- more -->
## The original feature of C# 6.0
One of its goals were to tackle the `INotifyPropertyChanged` (INPC) problem, where having to manually raise the `PropertyChanged` event was a very repetitive task. That feature could have enabled very interesting scenarios such as the inclusion of the `replace` keyword. It could allow for a source generator to replace the content of a method (a property setter for instance), with another method, while allowing it to call back the original method through an `original` keyword.

The objectives were probably a bit too broad at the time. The implications with the build pipeline as a whole, and particularly with the IDE were a bit too large to chew. I [discussed that a little bit](https://jaylee.org/archive/2019/01/06/improving-out-of-process-csharp-source-generation-performance.html) in article I wrote a few months ago.

At time though, I though it would still be interesting to write a source generator that would have less ambitious goals, but still would allow for source generation in a build pipeline, even if it would not be tightly integrated in Roslyn. That's how the Uno.SourceGeneration project was born.

It's still going strong as it is used throughout Uno, to solve a large variety of problems.

## Source Generation reborn

In most recent use cases at Microsoft, the use of T4 templates seems to be the choice, and it has its set of issues such as build performance, the inability to work on partially valid or complete source trees, the inability to use information from the syntactic model or the duplication of the semantic model work.

The team is looking at a smaller set of features, removing the ability to use `replace` or `original`, but provide an end to end experience that includes the IDE, something that the [Uno.SourceGeneration framework](https://github.com/unoplatform/Uno.SourceGeneration) is not able to support completely.

This would solve one of the most glaring issue of source generation in Visual Studio, where arbitrary modified files (e.g. non XAML files) can't re-rerun the generation without an explicit build, and where intellisense caching is getting in the way of using generated file symbols.

## The INotifyPropertyChanged case

As [Robin Sue mentions](https://github.com/dotnet/roslyn/pull/40162#issuecomment-562971937), the often requested scenario for generation is *INPC*, for which tagging a property automatically raising the `PropertyChanged` event, will not be addressed by the newly proposed version of the feature.

There's still a way around this, albeit a bit more verbose or at first counter intuitive. Generating source in `replace`-less model requires to make use of partial classes and methods, and more generally delegate the creation of significant language items to the source generator.

For the case of INPC, it is possible to write a class this way:

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
Notice here that the property is not visible at all, and it will be assumed that the generator will recognize the `GeneratedPropertyAttribute` and do the rest of the work for the author of the class.

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

The complete property body is then generated with the appropriate boiler plate. It can also include a `OnMyPropertyChanged` method that can optionally be implemented as needed in the main class to react to the property changes. Same could also be done for the getter, if there's work to be done in that context.

In the end, users of the class won't notice that the properties were generated.

This approach is not without drawbacks, though. For instance, documentation is not generated here, and the generator may need to forward the documentation of the `_myProperty` field over to the generated property. The same applies to the attributes that may be needed on that property; it's not possible to apply an attribute on a property defined in another partial type declaration.

## Implementing the field backed INPC generator

Let's try creating this generator using the Uno.SourceGeneration package. Its API is very similar to the current Roslyn proposal (aside from the optional msbuild ties). It could be easily ported over to a future roslyn-based implementation of the generators if it does not change too much.

First, we can assume that there will be an available property named `INPC.GeneratedPropertyAttribute`, which allows us to find its symbol:

```csharp
public class INPCGenerator : SourceGenerator
{
    public override void Execute(SourceGeneratorContext context)
    {
        // Search for the GeneratedPropertyAttribute symbol
        var _generatedPropertyAttributeSymbol =
            context.Compilation.GetTypeByMetadataName("INPC.GeneratedPropertyAttribute");
    }
}
```

This will ease our ability to find fields in types that are tagged with this attribute.

Now let's find those tagged fields:

```csharp
// Search in all types defined in the current compilation (but not in the dependents)
var query = 
    from typeSymbol in context.Compilation.SourceModule.GlobalNamespace.GetNamespaceTypes()
    from property in typeSymbol.GetFields()

    // Find the attribute on the field
    let info = property.FindAttributeFlattened(_generatedPropertyAttributeSymbol)
    where info != null

    // Group properties by type
    group property by typeSymbol into g
    select g;
```

This will create a grouped `IEnumerable`, for which the `Key` is the owner of the marked fields, and the group items are the fields symbols.

Then we can generate the class and the properties plumbing:
```csharp
foreach(var type in query)
{
  var builder = new IndentedStringBuilder();

  builder.AppendLineInvariant("using System;");
  builder.AppendLineInvariant("using System.ComponentModel;");

  using (builder.BlockInvariant($"namespace {type.Key.ContainingNamespace}"))
  {
    // Add the INotifyPropertyChanged interface to the existing type
    using(builder.BlockInvariant($"partial class {type.Key.Name} : INotifyPropertyChanged"))
    {
      // Implement the event itself
      builder.AppendLineInvariant($"public event PropertyChangedEventHandler PropertyChanged;");

      foreach(var fieldInfo in type)
      {
        // Uppercase name for camel case
        var propertyName = fieldInfo.Name.TrimStart('_');
        propertyName = propertyName[0].ToString().ToUpperInvariant() + propertyName.Substring(1);

        // Create the property body
        using (builder.BlockInvariant($"public {fieldInfo.Type} {propertyName}"))
        {
          builder.AppendLineInvariant($"get => {fieldInfo.Name};");

          using (builder.BlockInvariant($"set"))
          {
            builder.AppendLineInvariant($"var previous = {fieldInfo.Name};");
            builder.AppendLineInvariant($"{fieldInfo.Name} = value;");
            builder.AppendLineInvariant(
                $"PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof({propertyName})));");
            builder.AppendLineInvariant($"On{propertyName}Changed(previous, value);");
          }
        }

        builder.AppendLineInvariant(
            $"partial void On{propertyName}Changed({fieldInfo.Type} previous, {fieldInfo.Type} value);");
      }
    }
  }

  var sanitizedName = type.Key.ToDisplayString().Replace(".", "_");
  context.AddCompilationUnit(sanitizedName, builder.ToString());
}
```

And it's done! It's a big string generator that most probably produces valid C#, depending on the efficiency of the type names sanitization (to remove non-valid characters).

This generator makes use of the `Uno.Roslyn` package, which provides a set of helpers to browser the types and their members more easily, such as `GetNamespaceTypes()`, `FindAttributeFlattened()` or `GetFields()`. It also provides the `IndentedStringBuilder` class, which allows for a nicely formatted generated files (because developers always want to read generated code 😊).

Finally, here's how to create a class that uses it :

```csharp
namespace INPC {
    public class GeneratedPropertyAttribute : Attribute { }
}

public partial class MyClass
{
    [GeneratedProperty]
    private string _stringProperty;

    [GeneratedProperty]
    private int _intProperty;

    partial void OnIntPropertyChanged(int previous, int value)
        => Console.WriteLine($"OnIntPropertyChanged({previous},{value})");
    partial void OnStringPropertyChanged(string previous, string value) 
        => Console.WriteLine($"OnIntPropertyChanged({previous},{value})");
}
```

And how to use it:
```csharp
var c = new MyClass();
c.IntProperty = 42;
c.StringProperty = "My 42";

// Output:
//
// OnIntPropertyChanged(0,42)
// OnIntPropertyChanged(,My 42)
```

## Wrapping up

This generator does not handle all possible cases for properties generation, such as nested types or fancy identifier names. That leaves lots to experiment with if you choose source generation to enable this scenario for your project.

You can find a [sample solution for this generator on GitHub](https://github.com/jeromelaban/inpc.generator).

That's it for now! Until next time, happy source-gen'ing!