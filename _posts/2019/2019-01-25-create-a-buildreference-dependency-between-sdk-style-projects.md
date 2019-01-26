---
layout: post
title: "Create a BuildReference Dependency Between C# Sdk-Style Projects"
date: 2019-01-25 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "MSBuild"]
author: Jerome
---

I know, a `BuildReference` does not exist. But it really should.

<!-- more -->

When using what are now called Legacy Projects (the big and clunky ones), it is possible to have the build system make an explicit ordering when building interdependent projects. This is very useful when creating [C# analyzers](https://github.com/nventive/Uno/blob/master/src/Uno.Analyzers/UnoNotImplementedAnalyzer.cs), [MSBuild Custom tasks](https://docs.microsoft.com/en-us/visualstudio/msbuild/task-writing?view=vs-2017), [.NET Core CLI extensions](https://docs.microsoft.com/en-us/dotnet/core/tools/extensibility), that are built and used within the same solution. The resulting output of those projects is not used to compile the referencing project, but rather by the tooling building the project (e.g. Roslyn or MSBuild).

In the case of analyzers, the target framework of the analyzer itself is either .NET Standard 2.0 or net46x because that's what the compiler uses. When building a `net47` or `xamarinios10` project, this dependency will not be compatible if it were a "standard" project reference.

In order to do this, Visual Studio stores build dependency information inside the solution file, and uses it to make build projects in the proper order, with the right amount of parallelism. And it worked great.

For as long as SDK-Style C#/F#/... projects (or [CPS](https://github.com/dotnet/project-system)) have existed, making build dependencies has been a pain to deal with, such as target injection or [cryptic techniques involving synthetic references (?)](https://github.com/Microsoft/msbuild/issues/3626#issuecomment-413889441), but [there's a solution to this](https://github.com/Microsoft/msbuild/issues/2274#issuecomment-426661962).

To make a `BuildReference` of sorts, from a project to another so they build in the proper sequence, add the following:

```xml
<ItemGroup>
  <ProjectReference Include="..\ProjectA\ProjectA.csproj">
    <!-- 
    this tells the current project to ignore the output assembly of ProjectA 
    -->
    <ReferenceOutputAssembly>false</ReferenceOutputAssembly>

    <!-- 
    This prevents the current project from trying to 
    determine the target framework of ProjectA
    -->
    <SkipGetTargetFrameworkProperties>true</SkipGetTargetFrameworkProperties>

    <!-- 
    This makes sure there isn't a TargetFramework defined anyways 
    -->
    <UndefineProperties>TargetFramework</UndefineProperties>
  </ProjectReference>
</ItemGroup>
```

Failing to add the last two lines will make both MSBuild and NuGet complain about incompatible project types, with different errors.