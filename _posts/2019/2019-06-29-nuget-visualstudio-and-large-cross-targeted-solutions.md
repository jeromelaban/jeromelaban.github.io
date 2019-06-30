---
layout: post
title: "NuGet, Visual Studio 2019 Solution Filters and Large Cross-targeted Solutions"
date: 2019-06-29 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "xamarin", "msbuild"]
author: Jerome
---

The combination of the [Visual Studio solution filtering](https://docs.microsoft.com/en-us/visualstudio/ide/filtered-solutions?view=vs-2019) and [NuGet support for it](https://github.com/NuGet/Home/issues/5820) allows for faster trimmed-down solution loading. This enables developers of large cross-targeted solutions to have a viable environment by temporarily focusing on a single target framework, and still have Visual Studio cooperate.

<!-- more -->
## Cross-targeted projects support in Visual Studio

Since the introduction of the new msbuild project file structure (also known as [Common Project System or CPS](https://github.com/dotnet/project-system)), it's been increasingly easy to create libraries that target multiple platforms at once. [Oren's msbuild.sdk.extras](https://github.com/onovotny/MSBuildSdkExtras) definitely helped a lot to make this seamless.

It's been a delight to use at first, as it replaces linked files and a myriad of framework specific projects, makes the creation of [multi-targeted NuGet packages](https://docs.microsoft.com/en-us/nuget/reference/msbuild-targets), supports [conditional project and package references](https://docs.microsoft.com/en-us/nuget/consume-packages/package-references-in-project-files#adding-a-packagereference-condition), and many other features.

But not all is rosy. The impact of this new project format on Visual Studio, and on NuGet is very significant. Each new target framework specified in the `<TargetFrameworks>` property is actually creating a new independent project in memory, even if large parts of the project definition are the same. For instance, using [conditional globbing to include files per target framework](https://github.com/unoplatform/uno/blob/master/src/PlatformItemGroups.props#L52-L56) can give a hard time to Visual Studio.

The CPU goes up significantly, memory as well (along with longer GC pauses), where it's very easy to reach the 32 bits process memory limit (and the [old 64 bits debate](https://blogs.msdn.microsoft.com/ricom/2009/06/10/visual-studio-why-is-there-no-64-bit-version-yet/)).

Intellisense is also impacted, as for each file present in a target framework, each popup showing types information has to iterate through all of them. NuGet is impacted in the same way.

## How did we get here ?

Through quite a natural flow. Xamarin.iOS, Xamarin.Android (per SDK version), Xamarin.Mac, UWP, .NET Core, WebAssembly, .NET Standard, etc... are valid targets for many libraries that provide abstractions over platforms. [The Uno Platform](https://github.com/unoplatform/uno) is a prime target for that, as each platform gets its own implementation of the UWP API.

This means that to get a valid solution, validate builds for all targets, and provided in-solution sample applications to get a proper debugging experience, it's required to include all those target frameworks. And there can be many of those, as many upcoming (.NET Standard 2.0, netcoreapp 3.0, uap versions, etc...)

## Temporarily reducing the target framework count

In general, when developing a cross-targeted library, it's very rare to work on multiple targets at once. This means that reducing the `TargetFrameworks` count during a development session can make sense.

Problem is that when reducing the number of target frameworks from many to `xamarinios10`, for instance, will make NuGet fail to restore the solution. This happens because some projects (sample apps generally) won't be able to find valid target frameworks from their dependencies, as they've been removed (e.g. an android app references a library that should support `monoandroid90`, but only supports `xamarinios10`).

A temporary solution is to make multiple solutions, but this gets out of hands very quickly when new projects get added to the main solution, and trimmed-down solutions don't.

## Combining solution filtering and single target frameworks

Visual Studio added the [Solution Filtering feature](https://docs.microsoft.com/en-us/visualstudio/ide/filtered-solutions?view=vs-2019) recently, which gives the ability to create a partially loaded solution for an improved IDE experience. There are two nice touches for this feature, with unloaded projects being hidden by default when a solution is opened from an `.slnf` file, and also the file format being json, not the exotic `.sln` format.

From the outside, this seems like _just_ a nice solution to load less projects. It does not allow to reduce the number of loaded target frameworks **per project**.

But, hidden there, in the [NuGet changelog](https://github.com/NuGet/Home/issues/5820), is the the fact that the NuGet restore operation is now able to take the solution filtering into account, and **ignore** unloaded projects!

This means that it's now possible to create a `Directory.Build.Props` file with this content:

```xml
<Project ToolsVersion="15.0">
  <PropertyGroup>
    <!--<TargetFrameworkOverride>netstandard2.0</TargetFrameworkOverride>-->
  </PropertyGroup>
</Project>
```

And for each project that can be adjusted, the following:

```xml
<Project Sdk="MSBuild.Sdk.Extras" ToolsVersion="15.0">
  <PropertyGroup>
    <TargetFrameworks>xamarinmac20;xamarinios10;MonoAndroid90;net461;netstandard2.0</TargetFrameworks>
  </PropertyGroup>

  <PropertyGroup Condition="'$(TargetFrameworkOverride)'!=''">
    <TargetFrameworks>$(TargetFrameworkOverride)</TargetFrameworks>
  </PropertyGroup>
</Project>
```

When debugging or developing for a specific target framework, un-commenting the `TargetFrameworkOverride` will essentially change the active target framework, and have Visual Studio only consider that one.

After that, to avoid having NuGet fail for projects that are not valid, [create a solution filter file](https://docs.microsoft.com/en-us/visualstudio/ide/filtered-solutions?view=vs-2019#create-a-solution-filter-file), and make sure to use it when appropriate.

A few things to note: 
- Changing that property must not be done when the solution is loaded, otherwise project and NuGet caching issues may arise, exposing instabilities in Visual Studio's project system.
- The `.slnf` format is [not supported by CLI msbuild](https://github.com/microsoft/msbuild/issues/4097), which makes the use of the `TargetFrameworkOverride` fail the build, as all the projects of the solution will be tentatively restored, incorrectly.
- The `TargetFrameworkOverride` property should be [declared in a file that is excluded from source control](https://github.com/unoplatform/uno/blob/9a70fb6df74cf64c40d22c4033d3d111c13dd1fb/src/Directory.Build.props#L3), so that it does not unwillingly break a build.

## Final thoughts

It still feels like a hack, because cross-targeting is not yet a first class citizen in Visual Studio. 

Yet, it definitely does work properly and gives a **lot** of processing power and time back to the developer, even it is at the expense of being able to view all target frameworks at once.