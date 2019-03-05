---
layout: post
title: "Improving C# Source Generation Performance with a Custom Roslyn Workspace"
date: 2019-03-04 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "MSBuild", "Roslyn"]
author: Jerome
---

In my [previous article](https://jaylee.org/archive/2019/01/06/improving-out-of-process-csharp-source-generation-performance.html) about the process of improving the source generation performance, I mentioned some improvements around the generation inside of an `AppDomain` for an improved isolation, inside of an out-of-process generation host.

It turned out to be problematic on both the memory consumption side, as well as on the cold start cost of creating a host, which I'll talk about in a later post. 

This quest for the generation performance also led me to rethink the use of the [MSBuildWorkspace class](https://gist.github.com/DustinCampbell/32cd69d04ea1c08a16ae5c4cd21dd3a3). It completely hides MSBuild object model created to build the Roslyn [Compilation](https://docs.microsoft.com/en-us/dotnet/api/microsoft.codeanalysis.compilation?view=roslyn-dotnet) object, which also needs to be provided to the source generators to get access to the code being built, forcing a double parsing and loading of a project file.

<!-- more -->

## The previous Source Generation Pipeline

Since the almost beginning of the SourceGeneration tasks (almost, because as it was originally based on Cecil), the generation pipeline was the following :

![Source Generation flow](/assets/images/2019-03-04-srcgen-01.png)

A performance problem lies in the Roslyn `MSBuildWorkspace`, which needs to get its own MSBuild instances to build its own tree of projects to create a `Compilation` object.

Since the Source Generators need those [MSBuild project instances](https://github.com/nventive/Uno.SourceGeneration/blob/6912bc1a451c7b2652dddba4065b6b00b5584f3e/src/Uno.SourceGeneration.Engine.Shared/SourceGeneratorEngine.cs#L73) directly, for a long time, the Source Generation task had to duplicate the work done by the roslyn `MSBuildWorkspace`, for the sake of simplicity.

After discussing this a bit with [Jason Malinowsky on twitter](https://twitter.com/jasonmalinowski/status/1093179721351815168), it turns out that having access to MSBuild underneath is not part of the envisioned API.

## The MSBuildWorkspace multiple scanning of the ProjectReference tree

Opening an MSBuild project file can be expensive, particularly when there are [globbing (or wildcards) patterns](https://docs.microsoft.com/en-us/visualstudio/msbuild/import-element-msbuild?view=vs-2017#wildcards). It can also be very expensive if a project opens the whole tree of `ProjectReference` items found while scanning project files.

The way the `MSBuildWorkspace` is built enables the creation of a valid Roslyn `Compilation` object, even none of the dependent projects of an opened project have been built. This makes sense in a general Roslyn use case, but not for the generators, which are invoked right before the normal compilation step, and that all the dependent projects have already been built.

In the context of the Uno.UI project, for which the projects tree is 6 or 7 levels deep, each MSBuildWorkspace of the tree has to load all of its dependencies. This makes the last project spend a lot of time parsing msbuild files.

There's the [`LoadMetadataForReferencedProjects`](https://github.com/dotnet/roslyn/blob/aab7fd01a12e7a96961f1c75b2d97af46faed1dc/src/Workspaces/Core/MSBuild/MSBuild/MSBuildWorkspace.cs#L116) that allows to specify that the workspace should favor built assemblies, but it still does not prevent the deep scanning.

This also has the annoying effect of breaking Target Framework compatibility dependency chains, with a [Roslyn issue like this one](https://github.com/dotnet/roslyn/issues/23114). This forced all project of the graph to build all possible target frameworks of the solution, instead of say, only `netstandard2.0`. Another way to make the build slower.

## Creating a custom MSBuildWorkspace

In this quest for trying to reduce the number of parsed MSBuild projects, I tried extracting parts of the Roslyn's `MSBuildWorkspace` to remove the step that was scanning and loading the projects tree. This could allow to take the Source Generation's own MSBuild project parsing, and give it to that custom workspace, and skip all the references.

A custom workspace can be built on top of the [`AdHocWorkspace`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.codeanalysis.adhocworkspace?view=roslyn-dotnet), a generic workspace that deals with Solutions and Projects, but without any relation to msbuild, nor any project or solution loader. It can't be used directly from msbuild, and that's what the MSBuildWorkspace does: it converts one object model to the other.

After lots of pulling strings of source files and encountering interesting bits such as the [`ProjectFileInfo` class](https://github.com/nventive/Uno.SourceGeneration/blob/6912bc1a451c7b2652dddba4065b6b00b5584f3e/src/Uno.SourceGeneration.Engine.Shared/Workspace/ProjectFileInfo.cs#L157) and the [`ProjectInfo` builder](https://github.com/nventive/Uno.SourceGeneration/blob/6912bc1a451c7b2652dddba4065b6b00b5584f3e/src/Uno.SourceGeneration.Engine.Shared/Workspace/ProjectInfoBuilder.cs#L67) which take all the msbuild properties and turns them into something an [`AdHocWorkspace`](https://docs.microsoft.com/en-us/dotnet/api/microsoft.codeanalysis.adhocworkspace?view=roslyn-dotnet) can load, here's what the sequence looks like:

![Source Generation flow](/assets/images/2019-03-04-srcgen-02.png)

The result is a single parsing of the currently built project, and a compilation object that goes through a [`ProjectInfo` builder](https://github.com/nventive/Uno.SourceGeneration/blob/6912bc1a451c7b2652dddba4065b6b00b5584f3e/src/Uno.SourceGeneration.Engine.Shared/Workspace/ProjectInfoBuilder.cs#L67)

Parsing just the project being built also removes the issues of the dependent projects loading incompatibilities. This means that it's now possible to remove all the dummy target frameworks in projects of the solution that would not have needed it without the source generation running.

If you want to take a deeper look, [this PR](https://github.com/nventive/Uno.SourceGeneration/pull/94) made those changes.

## Next steps

Pulling the sources from Roslyn is no small task, as there are lots of internal methods required for the interpretation of an MSBuild file, such as the surprising parsing of the [CSC command line arguments produced by the msbuild targets](https://github.com/nventive/Uno.SourceGeneration/blob/6912bc1a451c7b2652dddba4065b6b00b5584f3e/src/Uno.SourceGeneration.Engine.Shared/Workspace/ProjectFileInfo.cs#L157).

This gives a better view of the internals of the `MSBuildWorkspace` class, and how to suggest a new type of lightweight MSBuild workspace, which would take a single MSBuild project instance as an input, would not deep load dependent projects, and would provide a compilation as an output.

That looks like an interesting upcoming Roslyn PR!