---
layout: post
title: "Chasing an MSBuild task devenv.exe file lock"
date: 2019-01-18 00:00:00 -0500
comments: true
category: Archive
comments: true
tags: [".NET", "MSBuild"]
author: Jerome
---

During the development of an Uno.UI build task for generating platform specific resources, I found out that invoking the [MSBuild task](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-task?view=vs-2017) with the `BuildInParallel` property set to true works around a task assembly locking issue. 

For some (yet) unknown reason, even if Visual Studio is scheduling most of its work to child msbuild.exe processes, when using the new SDK style project format on large projects, the `devenv.exe` process gets used to build project outputs. This forces the developer to close the IDE and deleting the task assembly to rebuild it again, and work around _file not found_ caching issue...

In this article, I will be discussing the solutions I explored to mitigate this issue.

<!-- more -->

## Troubleshooting an MSBuild task file locking

As part of [developing the Source Generation tasks](2019-01-06-improving-out-of-process-csharp-source-generation-performance.md), I've been able to start deciphering the behavior of MSBuild. All of its magic starts to make more sense now, maybe with the exception of ['%' ItemGroup patterns](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-items?view=vs-2017#BKMK_ReferencingItemMetadata) that I always have to lookup the documentation for...

This file locking behavior has been quite odd and confusing, since MSBuild and VS teams have been making lots of work to make sure most (if not all) of the work is pushed out of the IDE for its reliability, this issue should have been happening at all.

In essence, the Uno.UI [custom build task](https://docs.microsoft.com/en-us/visualstudio/msbuild/task-writing?view=vs-2017) in question is injecting itself in the build process early to process `PRIResource` and `Content` files to they get converted to native iOS, Android and Web resources. To do so, the `BeforeTargets` property comes handy:

```xml
<UsingTask AssemblyFile="$(UnoUIMSBuildTasksPath)\Uno.UI.Tasks.v0.dll" TaskName="Uno.UI.Tasks.ResourcesGenerator.ResourcesGenerationTask_v0" />

<Target Name="UnoResourcesGeneration"
        DependsOnTargets="AssignLinkMetadata">
    <ResourcesGenerationTask_v0 ... />
</Target>
```

This will make that the `UnoResourcesGeneration` task will be automatically injected before the `AssignLinkMetadata` target is invoked by another caller (commonly a dependency of the `Build` target).

## The NuGet context workaround
This file locking problem only exists inside of the Uno.UI solution.

The Uno.UI.Tasks file is packaged as part of the Uno.UI NuGet, using the [git commit ID in place of the `v0`](https://github.com/nventive/Uno/blob/6508fe9f8d3972d6121401f205823b907954e9b2/build/Uno.UI.proj#L86). This ensures that when MSBuild loads that build task, it will load the file for the referenced package, side-by-side (SxS) with any previously loaded task of the same package with a different version.

Strong signing the assembly could also have be used, but there are too many gotchas. I preferred using this unique-type-name technique to avoid type aliasing issues when inside the Uno.UI solution itself.

## Same-solution msbuild task dependencies
The Uno.UI solution is built in such a way that Uno.UI itself needs the resources generation task to generate the compiled language resources files for the Uno.UI controls. This means that there is a dependency between the `Uno.UI` project and the `Uno.UI.Tasks` project.

When starting from a clean git repository, to be able to build the Uno.UI solution completely from VS, it was necessary to build it once, have it fail half the way with msbuild saying that the `Uno.UI.Tasks.v0.dll` file could not be loaded even though it was there, close visual studio and build again.

Time consuming, really, and very obscure enough to say _"VS is again throwing a tantrum"_. Not this time, though!

## Design-Time builds
By default, the target above may be executed by what are called design-time builds. Those are builds that are executed by Visual Studio using a subset of the targets. This allows for the resolution of files to show in the IDE, the nuget packages to reference, which projects to show in the project selector at the top of an editor:

![project selector](/assets/images/2019-01-18-project-selector.png)

If for some reason those design-time builds fail, or are slow, this means that IDE features will take time to show up.

This also means that when injecting a task through `BeforeTargets` a custom task may get executed as part of those design time builds.

One tool that is very handy to troubleshoot those builds is the [Project System Tools](https://marketplace.visualstudio.com/items?itemName=VisualStudioProductTeam.ProjectSystemTools). It provides a set of new windows that show build activity history:

![build logging](/assets/images/2019-01-18-build-logging.png)

Clicking on the play icon of the **Build Logging** window enables the recording of explicit and design-time builds, seeing what does builds do and don't.

Using this tool, and the excellent [MSBuild binlog viewer](http://www.msbuildlog.com/), you can find out if and when a custom task is run. One possible solution to make sure that my task would not be executed in during design-time (and not by mistake in `devenv.exe`) is to [use the `BuildingProject` property](https://github.com/dotnet/project-system/blob/f28a7397ca13a57e08fb9db9e7dcbe35eb977c8b/docs/design-time-builds.md#determining-whether-a-target-is-run-in-a-design-time-build):

```xml
<Target Name="UnoResourcesGeneration"
        DependsOnTargets="AssignLinkMetadata"
        Condition="'$(BuildingProject)' == 'true'">
    <ResourcesGenerationTask_v0 ... />
</Target>
```

Making this change fixed the fact that the build task assembly was trying to be loaded too early in the build process, while it was not compiled yet, and have MSBuild cache the fact that the file could not be loaded.

But this did not change much with regards to the file locking. Randomly, the file would get locked by the IDE, and MSBuild would report so by saying the following when building the largest Uno.UI project: 

```
Beginning retry 1 in 500ms. The process cannot access the file 'bin\Debug\\Uno.UI.Tasks.v0.dll' because it is being used by another process. The file is locked by: "devenv.exe (16587), MSBuild.exe (15548), MSBuild.exe (13092), MSBuild.exe (16508), MSBuild.exe (19808), MSBuild.exe (3932), MSBuild.exe (19320)"
```

## The AppDomain approach

MSBuild has an `AppDomainIsolatedTask` class, which can be used to execute tasks in another AppDomain, a common use case for unloading assemblies.

Unfortunately, this cannot be used as MSBuild itself is loading the task assembly using [.NET's reflection APIs to find the `[LoadInSeparateAppDomain]` attribute](https://github.com/Microsoft/msbuild/blob/e70a3159d64f9ed6ec3b60253ef863fa883a99b1/src/Deprecated/Engine/Shared/LoadedType.cs#L146), which locks the file. Could be a good use of either the newer [System.Reflection.Metadata](https://www.nuget.org/packages/System.Reflection.Metadata) or even [Cecil](https://github.com/mono/cecil) inside MSBuild. This base task is generally used to mitigate other assemblies locking issues, but not the task itself. It's definitely not a silver bullet, as it has performance issues associated with the AppDomain creation and interop marshalling.

Another approach could have been to using MSBuild [inline tasks](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-roslyncodetaskfactory?view=vs-2017):

```xml
<UsingTask
    TaskName="ResourcesGenerationTask"
    TaskFactory="CodeTaskFactory"
    AssemblyFile="$(MSBuildToolsPath)\Microsoft.Build.Tasks.Core.dll">
    <ParameterGroup>
        <AssemblyPath ParameterType="System.String" 
                    Required="true" />
        <Resources ParameterType="Microsoft.Build.Framework.ITaskItem[]" />
        <GeneratedFiles ParameterType="Microsoft.Build.Framework.ITaskItem[]"
                        Output="true"/>
    </ParameterGroup>
    <Task>
        <Reference Include="System.Core" />
        <Reference Include="Microsoft.CSharp" />
        <Using Namespace="InlineTasks" />
        <Code Type="Class" Language="cs">
        <![CDATA[ 
        using System;  
        using System.Reflection;
        using Microsoft.Build.Framework;  
        using Microsoft.Build.Utilities;

        namespace InlineTasks
        {      
            public class IsolatedResourcesGenerationTask : AppDomainIsolatedTask
            {
                [Required]
                public string AssemblyPath { get; set; }

                [Required]
                public ITaskItem[] Resources { get; set; }

                [Output]
                public ITaskItem[] GeneratedFiles { get; set; }

                public override bool Execute()
                { 
                    var asm = Assembly.LoadFrom(AssemblyPath);
                    var type = asm.GetType("Uno.UI.Tasks.ResourcesGenerator.ResourcesGenerationTask_v0");
                    var instance = Activator.CreateInstance(type);

                    SetPropertyValue(type, instance, "Resources", Resources);

                    SetPropertyValue(type, instance, "BuildEngine", BuildEngine);
                    type.GetMethod("Execute").Invoke(instance, null);

                    GeneratedFiles = (ITaskItem[])type.GetProperty("GeneratedFiles").GetValue(instance);

                    return true;  
                }

                private void SetPropertyValue(Type taskType, object instance, string name, object value)
                {
                    taskType.GetProperty(name).SetValue(instance, value);
                }
            }
        } 
            ]]>
        </Code>
    </Task>
</UsingTask>
```

The idea behind this technique is to let msbuild create and manage an inline task, run it in another AppDomain, and manually interact with our final task using reflection. This ensure the target assembly is not referenced prior to the loading in the other AppDomain.

But it did not work either, as devenv.exe still ended up keeping a lock on the target assembly file. It's also very difficult to maintain and debug.

It happens that MSBuild manages its AppDomains in lazy way, keeping them alive. It also [does not](https://github.com/Microsoft/msbuild/blob/e70a3159d64f9ed6ec3b60253ef863fa883a99b1/src/Shared/TaskLoader.cs#L96) enable [shadow copying](https://docs.microsoft.com/en-us/dotnet/api/system.appdomainsetup.shadowcopyfiles?view=netframework-4.7.2) referencing the original assembly directly. 

Managing AppDomains manually is not an option either because we cannot control how msbuild loads it own assemblies, or provide a proper configuration file.

## Using the MSBuild task to create sub-processes

Another approach is to use sub-MSBuild processes, by executing specific targets outside of a currently building project, then getting its [returned items](https://docs.microsoft.com/en-us/visualstudio/msbuild/target-element-msbuild?view=vs-2017#attributes-and-elements):

```xml
<UsingTask AssemblyFile="$(UnoUIMSBuildTasksPath)\Uno.UI.Tasks.v0.dll" TaskName="Uno.UI.Tasks.ResourcesGenerator.ResourcesGenerationTask_v0" />

<Target Name="UnoResourcesGeneration"
		BeforeTargets="PrepareForBuild;_CheckForContent"
		DependsOnTargets="AssignLinkMetadata"
		Condition="'$(BuildingProject)' == 'true'">

    <MSBuild Projects="$(MSBuildProjectFile)"
             Targets="_InnerUnoResourcesGeneration_Resources"
             BuildInParallel="true"
             Properties="Configuration=$(Configuration);Platform=$(Platform)">
      <Output TaskParameter="TargetOutputs" ItemName="UnoResourceFiles" />
    </MSBuild>

    <ItemGroup>
	  <EmbeddedResource 
            Condition="'%(UnoResourceFiles.UnoResourceTarget)' =='Uno'" 
            Include="@(UnoResourceFiles)" />
	</ItemGroup>

</Target>

<Target Name="_InnerUnoResourcesGeneration_Resources"
	    DependsOnTargets="AssignLinkMetadata"
        Outputs="@(UnoGeneratedFiles)">
    <ResourcesGenerationTask_v0 ...>
           <Output TaskParameter="GeneratedFiles"
			       ItemName="UnoGeneratedFiles" />
    </ResourcesGenerationTask_v0>

</Target>
```

The original task now does not load the `ResourcesGenerationTask_v0` directly, but defers its execution in what should be a sub-process. This takes under the assumption that if `devenv.exe` still executes `UnoResourcesGeneration`, having a sub-process will not use `devenv.exe`.

A target can only return one set of items, but it's possible merge them in the sub target, then  filter/split them using custom metadata, with the `'%(UnoResourceFiles.UnoResourceTarget)' =='Uno'` condition above.

But it did not work either! `devenv.exe` was still locking the file...

To be able to determine when devenv is used, and fail the build, I added the following helper:

```xml
<UsingTask
  TaskName="CheckForDevenv"
  TaskFactory="CodeTaskFactory"
  AssemblyFile="$(MSBuildToolsPath)\Microsoft.Build.Tasks.Core.dll" >
  <ParameterGroup />
  <Task>
    <Reference Include="System.Xml"/>
    <Using Namespace="System"/>
    <Using Namespace="System.IO"/>
    <Code Type="Fragment" Language="cs">
      <![CDATA[
        var processName = System.Diagnostics.Process.GetCurrentProcess().ProcessName;
        if (processName.Equals("devenv", System.StringComparison.OrdinalIgnoreCase))
        { 
            throw new System.InvalidOperationException(); 
        }
      ]]>
    </Code>
  </Task>
</UsingTask>
```

Used like this:

```xml
<Target Name="_InnerUnoResourcesGeneration_Resources"
	    DependsOnTargets="AssignLinkMetadata"
        Outputs="@(UnoGeneratedFiles)">
    <CheckForDevenv />
</Target>
```

This made the build fail, and know exactly when a task is executed under the wrong process.

## Digging into msbuild

Digging a bit deeper into what MSBuild does with its [MSBuild task](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-task?view=vs-2017), I found out that the `BuildInParallel` property is [forcing the dispatch to another build node](https://github.com/Microsoft/msbuild/blob/0263a588f3db16abe4ae4a051213f4809e8b55c2/src/Tasks/MSBuild.cs#L345), something [forcing properties values](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-task?view=vs-2017#parameters) does not seem to do by default even with this:

```
When you pass properties to the project through the Properties parameter, MSBuild creates a new instance of the project even if the project file has already been loaded.
```

A new instance of a project does imply running in another process.

## Onto msbuild.exe not locking the files either

With this last technique, `devenv.exe` does not lock the task assembly anymore but `msbuild.exe` still does. This means that for now 

`TASKKILL /F /IM msbuild.exe`

is still needed to rebuild the build task project in the solution, but that's definitely faster that reloading the full IDE.

It would be very useful if MSBuild would include a way to execute tasks in a sub-process, shadow copied, or in another app-domain so it does not stay locked. 

Maybe in MSBuild 17.0, or with a new PR in 2019...