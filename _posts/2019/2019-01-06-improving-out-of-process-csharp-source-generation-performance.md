---
layout: post
title: "Improving out-of-process C# Source Generation performance"
date: 2019-01-06 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET","Source Generation"]
author: Jerome
---

_It's been a very long while since I've blogged, and a number of factors had reduced my time and ability to write about 
what I was working on for these past years. Now that most of what I work on is public through the [Uno Platform](https://github.com/nventive/uno), 
I'll be spending some time writing in depth articles about some technical aspects, but also a bit about IoT, a very fast-paced area these days._

_Happy new year!_

For starting up blogging again, I'll be discussing the implementation of the [Uno.SourceGenerationTasks](https://github.com/nventive/Uno.SourceGeneration) project, a source generation framework that allows for NuGet-distributable source generators, in the same way roslyn allows for distributable analyzers. 

People have [recently been talking about it on twitter](https://twitter.com/jasonbock/status/1080905751776751616), and I figured I could talk a bit more about the path the implementation took to this day.
<!-- more -->

Lots of generators have been developed recently, such as a [Xaml/C# to C# generator](https://github.com/nventive/Uno/tree/master/src/SourceGenerators/Uno.UI.SourceGenerators/XamlGenerator), [Xaml bindings generator](https://github.com/nventive/Uno/tree/master/src/SourceGenerators/Uno.UI.SourceGenerators/BindableTypeProviders), a [C# to TypeScript generator](https://github.com/nventive/Uno/tree/master/src/SourceGenerators/Uno.UI.SourceGenerators/TSBindings), [Immutable Entities](https://github.com/nventive/Uno.CodeGen/blob/master/doc/Immutable%20Generation.md) and [Equality generator](https://github.com/nventive/Uno.CodeGen/blob/master/doc/Equality%20Generation.md), [Class LifeCycle generator](https://github.com/nventive/Uno.CodeGen/blob/master/doc/Class%20Lifecycle%20Generation.md), ...

These generators generally save a lot of time by avoiding developer maintained plumbing code, or create code from facts known a compile time.

## A bit of background about C# Source Generation

A few years back, a very interesting specification had been added to the C# 6.0 proposal, [Source Generators](https://github.com/dotnet/roslyn/blob/master/docs/features/generators.md). It would have provided a way to define plug-able source generation binaries to the compiler, giving those the ability to inject new source files into the compilation pipeline. The API for the generation is simple enough:

```csharp
public abstract class SourceGenerator
{
    public abstract void Execute(SourceGeneratorContext context);
}

public abstract class SourceGeneratorContext
{
    public abstract Compilation Compilation { get; }
    public abstract void ReportDiagnostic(Diagnostic diagnostic);
    public abstract void AddCompilationUnit(string name, SyntaxTree tree);
}
```
The compiler calls the Execute method, your generator does its work and adds a compilation unit back to the compilation environment.

I was waiting on this feature to be implemented, but for a variety of reasons, it got shelved by the roslyn team.

So, aside from [IL weaving](https://github.com/Fody/Fody) and [T4 templates](https://docs.microsoft.com/en-us/visualstudio/modeling/code-generation-and-t4-text-templates?view=vs-2017), there was not much available that was consolidating the use of source generation, and even less using the Roslyn API. Not being a big proponent of IL weaving because of the "magic" it introduces, I was mostly relying on custom MSBuild tasks and the excellent [Mono.Cecil](https://github.com/jbevain/cecil) library to generate source files. I kept going in this direction because generated files are easily browsed and debugged, the execution flow can be followed like any normal source file.

A requirement for source generation is to be able generate code on every build, so the developer cannot alter the generated files, and cannot rely on modified generated files. I've seen too many times developers modify single-time generated files, then be stuck on the inability to re-run the generation tool to fix an issue or update the generated code because the generation inputs had changed. Even having a very large `*** DO NOT MODIFY THIS FILE***` at the top was not enough (that must speak to quite a few people!).

At the time, using MSBuild tasks became a key feature, because of the [introduction of NuGet support for MSBuild target/props](https://docs.microsoft.com/en-us/nuget/create-packages/creating-a-package#including-msbuild-props-and-targets-in-a-package). It became possible to create a simple NuGet package that injects itself into the project's build pipeline, doing all sorts of operations before and after many others, including [adding new files to the compilation](https://github.com/nventive/Uno.SourceGeneration/blob/a4bb268a8ae869e3b9156b99034b9d78f505c4ec/src/Uno.SourceGeneratorTasks.Shared/Content/Uno.SourceGenerationTasks.targets#L119). This feature avoided the development of complex VS-only add-ins to generate code.

But this MSBuild/Cecil approach had some limitations, as it only worked on already compiled binaries, which makes some scenarios (such as XAML integration, or `INotifyPropertyChanged` generation impossible). It also has the extremely annoying limitation of locking binaries from the NuGet package that provide custom tasks.

This last limitation has been one of the key feature of the Uno Source Generation tasks. It is possible to change a generator's code, rebuild and test it without having to restart visual studio (and kill all the MSBuild nodes).

## Iterations of the Uno Source Generation Tasks

Given that the Roslyn team would not implement that specification in the near feature (and it's still not in scope for C# 8.0), I really wanted to be able to tap into the power of Roslyn and generate code using it. But I still wanted to stay aligned with the original specification and its `SourceGenerator` class, in case it ever got released (one can still hope!).

It became clear very quickly that to be able to avoid the issue of file locking by `msbuild.exe`, the generation had to be done inside of separate `AppDomain` instances, with their own [Binding Redirects](https://docs.microsoft.com/en-us/dotnet/framework/configure-apps/file-schema/runtime/bindingredirect-element). It took a while to get this working properly, as it is always complex to deal with Cross AppDomain marshaling, and its picky Lifetime services.

AppDomains have the [ShadowCopy feature](https://msdn.microsoft.com/en-us/library/system.appdomain.setshadowcopyfiles(v=vs.110).aspx), which forces the runtime to make a copy of an assembly that is not in the GAC before loading it. This ensure that source generators can be rebuilt from visual studio, and each new version can be used immediately without jumping through assembly versioning hoops. The source generator framework can then unload or change active AppDomains when an assembly gets changed, creating a new shadow copy without locking the original.

This AppDomain technique is still working quite well, breaking once in a while when Visual Studio would change its MSBuild assemblies and/or targets (e.g. between Dev 14.0 and Dev 15.0).

This MSBuild dependency is also a particularly important aspect of the source generation framework. The Uno Source Generation Task load the full project `.csproj` file, and executes all targets defined by the project and its dependencies until the compilation phase. This way, all the project and assembly references, properties and items are defined properly and Roslyn is able to provide an exact replica of the final compilation environment to the source generators.

This MSBuild environment is also accessible to the source generators, something that roslyn is not willing to provide in its infrastructure. This makes for some scenarios complex to implement, such as configurable generators from MSBuild properties or items.

One problem with using MSBuild is that the generation framework must use the currently used MSBuild binaries, which is where custom resolvers had to be introduced along with explicit binding redirects. This would not be working all the time, because some MSBuild nodes (used to avoid reloading MSBuild binaries and improve performance) are run inside devenv.exe itself, but only for Xamarin.Android projects. In this case, AppDomains would have to be configured in a very specific way to avoid the reuse of binaries across domains.

## macOS support

Working in a mostly Windows/Visual Studio environment at the time, macOS support was not much of a concern. For a while, requests were coming to add support for macOS, but MSBuild was not properly supported on non-Windows environments at the time either.

This changed radically when [MSBuild got open-sourced](https://blogs.msdn.microsoft.com/dotnet/2015/03/18/msbuild-engine-is-now-open-source-on-github/),  later supported by mono, and integrated into Visual Studio for macOS. From that point in time, most of the source generators worked untouched, given that some small modifications were added here and there (e.g. the usual directory separator bugs)

Aside from a few hiccups, like the fact that VS4Mac (still) does not support multiple MSBuild nodes, or that the implementation of AppDomains broke remote logging, the implementation worked for a while.

## Out of (MSBuild) process - VS4Mac 7.6, linux and .NET Core support

In the same time frame, some new requests arrived for Linux (to support [mono-wasm AOT](https://github.com/nventive/Uno.Wasm.Bootstrap)), VS4Mac 7.6 and .NET Core builds.

VS4Mac introduced a breaking change in the support for some shared binaries in the Roslyn APIs (some internal SQLite database), which made the compilation in a separate AppDomain impossible to solve.

On the linux front, MSBuild incompatibilities with the way roslyn work made it impossible to load it in-process properly from an MSBuild task.

Finally, on the .NET Core front, AppDomains are not supported at all, making the parallel loading impossible.

While those environments are quite different, supporting all those environment properly lead to the extraction of the source generation framework in an out-of-process execution. This way, the dependencies are clearly defined, and the already loaded binaries of some environments (VS4Mac IDE or CLI) are not interfering with the generation framework and generators dependencies.

While this architecture works, it is not particularly fast, especially in the context of incremental builds. The out-of-process host is re-started for every platform of every built project, and [there can be a lot of those](https://github.com/nventive/Uno/blob/master/src/Uno.UI/Uno.UI.csproj).

## Using Roslyn's Generation Server Host

I had in mind for a while that using [the `VBCSCompiler.exe` infrastructure](https://github.com/dotnet/roslyn/tree/161765c2d12b228692dab38339701dbd24b4757b/src/Compilers/Server/VBCSCompiler) would be required at some point, and that point has been reached recently.

Roslyn's shared compiler goal is to stay alive in memory, and process compilation requests as they come from the `CSC` MSBuild compilation task or `csc.exe`, through named pipes for the current user. The main reason for the [presence of this host is performance](https://github.com/dotnet/roslyn/blob/04e481180a7e0b64b36f6b8c475c8a3a7a263f28/docs/compilers/Compiler%20Server.md), and to cache metadata references, a step of the compilation that is particularly expensive. In most cases, there will not be that many live instances of the `VBCSCompiler.exe` binary, as there is not much state kept in memory other than roslyn itself.

This architecture can be reused in the context of source generation, but because of MSBuild assemblies, source generators and their dependencies are loaded in memory, the number of `Uno.SourceGeneration.Host.exe` processes is based on the unicity of project file path, built platform and generators paths. This means that when building a C# project with 7 target frameworks, there will be at least 7 instances of the host process in memory. This is a trade-off between memory and performance, but this also means that processes can be 64 Bits, removing the limits that MSBuild and the VS IDE currently have.

Having this server generation architecture reuse and metadata reference reuse make for incremental compilation very fast (from 5 to 10s down to 500ms).

## Generation Server on Linux and macOS

When trying to run the generation server process on macOS or Linux using mono, either System.IO.Pipes completely fails, or partially because of some [System.Native missing implementation](https://github.com/mono/mono/blob/18c632ed43145ccbc430136a04b272e6b1f1d533/mcs/class/System.Core/unix_net_4_x_System.Core.dll.sources#L39). This will still be the case until the import of many parts of the CoreFX repository get integrated in Mono.

This means that the implementation of this server host process is only available on Windows or when building .NET Core projects, until Mono supports new System.Native imports. For the time being building Xamarin projects in VS4Mac will take a bit more time as they fall back on a single-use out-of-process generation mode.

## Up next...

The source generation server host work is [still in progress in this PR](https://github.com/nventive/Uno.SourceGeneration/pull/84), but it's been a interesting dive into Roslyn's infrastructure.

Upcoming work on the source generation framework, such as active roslyn version selection, expanding generation to anything, not just C#.

Until then, have fun in the .NET universe!