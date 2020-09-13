---
layout: post
title: "Using MSBuild Items and Properties in C# 9 Source Generators"
date: 2020-07-19 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "C#", "Source Generation", "Roslyn"]
author: Jerome
---

C# 9.0 Source Generation is [progressing quite nicely lately](https://twitter.com/jaredpar/status/1301315173244788736) (Thanks, Jared!), with the addition of the ability to interact with the MSBuild environment such as getting Properties and Items to control how the generation happens.

In this post, I'll explain how to parse `.resw` files of a project to generate an enum that contains all the resources.

<!-- more -->

The full sample for this article is [here in the Fonderie Generators project](https://github.com/jeromelaban/fonderie).

## Reading msbuild items and properties

In the [Roslyn generators cookbook](https://github.com/dotnet/roslyn/blob/master/docs/features/source-generators.cookbook.md), new entries have been added to include the APIs needed to get information from msbuild. In order to make the reading of those properties easier, here's a small extensions class:

```csharp
internal static class SourceGeneratorContextExtensions
{
    private const string SourceItemGroupMetadata = "build_metadata.AdditionalFiles.SourceItemGroup";

    public static string GetMSBuildProperty(
        this SourceGeneratorContext context,
        string name,
        string defaultValue = "")
    {
        context.AnalyzerConfigOptions.GlobalOptions.TryGetValue($"build_property.{name}", out var value);
        return value ?? defaultValue;
    }

    public static string[] GetMSBuildItems(this SourceGeneratorContext context, string name)
        => context
            .AdditionalFiles
            .Where(f => context.AnalyzerConfigOptions
                .GetOptions(f)
                .TryGetValue(SourceItemGroupMetadata, out var sourceItemGroup)
                && sourceItemGroup == name)
            .Select(f => f.Path)
            .ToArray();
}
```

Let's dive into what those extensions do.

### GetMSBuildProperty
The `GetMSBuildProperty` method is assuming that a defined property has a non-empty value, as per the msbuild semantics. Here's how to get the default namespace for the current project:
```csharp
var defineConstants = context.GetMSBuildProperty("RootNamespace");
```
Assuming that the associated msbuild property is added in the generator's associated props file:
```xml
<ItemGroup>
    <CompilerVisibleProperty Include="RootNamespace" />
</ItemGroup>
```

### GetMSBuildItems
For `GetMSBuildItems`, since the roslyn APIs does not provide a way to discriminate items per their MSBuild item name, we need to use some metadata that can be added to the `AdditionalFiles` items. In order to get the `resw` files from a WinUI project, we can do the following:

```csharp
var priResources = context.GetMSBuildItems("PRIResource");
```

For this code to work, we need to change a little bit the way items are added to the roslyn context:
```xml
<Target Name="_InjectAdditionalFiles" BeforeTargets="GenerateMSBuildEditorConfigFileShouldRun">
    <ItemGroup>
        <AdditionalFiles Include="@(PRIResource)" SourceItemGroup="PRIResource" />
    </ItemGroup>
</Target>
```

The use of a target here is needed because of the way NuGet packages property or targets files are handled by msbuild. If the ItemGroup is included directly at the root of the project, its evaluation is performed too early in the build process. This sequence misses items being added by the project or dynamically by other targets.

At this point, there's no clear injection point to execute this targe in Roslyn, but [GenerateMSBuildEditorConfigFileShouldRun](https://github.com/dotnet/roslyn/blob/ff854c695779990b9b269029a8615782a59ec530/src/Compilers/Core/MSBuildTask/Microsoft.Managed.Core.targets#L112) seems like an appropriate location for doing so at this point, right before the capture of the properties and items by the build.

Finally, to be able to discriminate items in the `AdditionalFiles` group, we use the `SourceItemGroup` metadata. For Roslyn to pick it up, we need to add the following:

```xml
<ItemGroup>
	<CompilerVisibleItemMetadata Include="AdditionalFiles" MetadataName="SourceItemGroup" />
</ItemGroup>
```

## Generating code from the resw file

Now that we can read the items and properties, we can write a small generator that creates an enum with all the names found in a `resw` file:

```csharp
[Generator]
public class ReswConstantsGenerator : ISourceGenerator
{
    public void Initialize(InitializationContext context)
    {
        // Debugger.Launch(); // Uncomment this line for debugging
        // No initialization required for this one
    }

    public void Execute(SourceGeneratorContext context)
    {
        var resources = context.GetMSBuildItems("PRIResource");

        if (resources.Any())
        {
            var sb = new IndentedStringBuilder();

            using (sb.BlockInvariant($"namespace {context.GetMSBuildProperty("RootNamespace")}"))
            {
                using (sb.BlockInvariant($"internal enum PriResources"))
                {
                    foreach (var item in resources)
                    {
                        XmlDocument doc = new XmlDocument();
                        doc.Load(item);

                        // Extract all localization keys from Win10 resource file
                        var nodes = doc.SelectNodes("//data")
                            .Cast<XmlElement>()
                            .Select(node => node.GetAttribute("name"))
                            .ToArray();

                        foreach (var node in nodes)
                        {
                            sb.AppendLineInvariant($"{node},");
                        }
                    }
                }
            }

            context.AddSource("PriResources", SourceText.From(sb.ToString(), Encoding.UTF8));
        }
    }
}
```

This will generate a file which contains an enum with all the resource names, in the default namespace of the current assembly.

Note that this generator does not validate the name's format, and if there are reserved characters or keywords, those are needed to be rewritten for C# to accept it.

## Wrapping up

This simple sample should most likely be improved.

For instance, it could be interesting to create a generator analyzing another generator to create a `.targets` file which contains the appropriate `CompilerVisibleItemMetadata` and `CompilerVisibleProperty` for that generator to work properly.

The extension also only supports getting the identity of an item, but getting additional metadata would be useful, like getting the `Link` attribute when dealing with linked files in projects.

You can find the [sample of this article here](https://github.com/jeromelaban/fonderie), and as of the writing of this post, Visual Studio 16.8 Preview 2.1 does not yet show the generated code or highlights properly but builds with the generated code properly. This should be improving the next previews.

Until next time, happy generation!