---
layout: post
title: "How to get the SyntaxNode of an ISymbol using Roslyn"
date: 2020-07-19 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "C#", "Source Generation", "Roslyn"]
author: Jerome
---

In this post, I'll describe how to determine if a property is an auto-property, using its `ISymbol` as the source, and not by using reflection into Roslyn which computes this information internally.

During the original development of the [Uno CodeGen source generators](https://github.com/unoplatform/Uno.CodeGen/), when building the [Immutable generators](https://github.com/unoplatform/Uno.CodeGen#create-truly-immutable-entities-in-c) (soon to be deprecated by the [records feature in C# 9.0](https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/)), we had to determine [if a property's backing field is generated or not](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/auto-implemented-properties).

<!-- more -->

At the time, not understanding fully the Roslyn sources, and with lack of time, we chose to go the easy and hacky route (with reflection on internal members). And like any hacky route, it always come back at your one way or another. In Roslyn 3.6, internals have changed breaking that reflection based code. 

Still, we knew that it was going to break in the future so the [error message was very explicit](https://github.com/unoplatform/Uno.CodeGen/blob/0bdde7524c34346c4115e074c89c1585dfa74217/src/Uno.CodeGen/Helpers/TypeSymbolExtensions.cs#L368-L369) in saying that we were looking for something internal that could not be found.

## How determine if a property is an auto-property

Given an `IPropertySymbol`, it's possible to get the location of the definition:

```csharp
if (symbol.Locations.FirstOrDefault() is Location location)
```

If a symbol has no location, it generally means it's defined outside the current compilation (e.g. in a referenced assembly).

Then we can get the actual `SyntaxNode` of that location:

```csharp
var node = location.SourceTree?.GetRoot()?.FindNode(location.SourceSpan);
```

The `FindNode` method uses the location's source span to lookup the appropriate so it can be traversed and analyzed syntactically. Note that there may not be a SourceTree for the that location either.

Once we have the node, we can analyze it:

```csharp
var isExplicitProperty = node
    .DescendantNodesAndSelf()
    .OfType<PropertyDeclarationSyntax>()
    .Any(prop =>
    {
        if(prop.AccessorList != null)
        {
            return prop.AccessorList.Accessors
                .Any(a => a.Body != null || a.ExpressionBody != null);
        }

        // readonly arrow property declaration
        return true;
    });
```

Here's what happens: 
- The code take only the `PropertyDeclarationSyntax` nodes
- Determine if there's an accessor list (are there explicit `get` or `set` keywords)
- Then if there are accessors, determine if they have a body (with curly braces) or are using the arrow syntax (lambda properties).
- If there are no accessors, this means it's a readonly property with an arrow syntax.

If this, you can know if a property is an auto-property or not.

This code is based on the [actual roslyn source code](https://github.com/dotnet/roslyn/blob/ba014d9d7728de0d4b5df3859507f9701e7032c0/src/Compilers/CSharp/Portable/Symbols/Source/SourcePropertySymbol.cs#L205) for the internal IsAutoProperty property.

## Wrapping up
This code is not future proof, though. Even if it's not using internal APIs, the language will evolve and may add newer features that may break or ignore newer constructs that will appear in later versions of the language.

That will be the topic of a future post, if that happens :)