---
layout: post
title: ".NET Threading and WebAssembly"
date: 2020-02-29 00:00:00 -0500
category: Archive
comments: true
tags: [".NET", "Threading", "WebAssembly"]
author: Jerome
---

Threading, in general operating systems sense, is not something that the web has been able to use until very recently. The addition of [Threads support in WebAssembly](https://github.com/WebAssembly/threads), and the activation of the threading support in Chrome opens up a whole new world of possibilities, including the use of Reactive Extensions (Rx.NET) or the Task Parallel Library (TPL).

Let's dive in, with some sample code.

<!-- more -->

## A bit of history

The only technique that was available for actual local parallelization of work in the javascript land was to use [WebWorkers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers). It's technically not threading in the same sense known by out-of-browsers developers, as it does not provide the ability to share memory between WebWorkers. To synchronize work, [messages](https://developer.mozilla.org/en-US/docs/Web/API/Worker/postMessage) are passed between workers and [the main loop](https://developer.mozilla.org/en-US/docs/Web/API/Worker/onmessage), which looks more like processes would exchange messages via IPC.

This changed recently with the ability for javascript to create [shared array buffers](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer), contiguous pieces of memory that can be both read and written from workers and the main loop. Those buffers can only be [mapped to primitive types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects#Indexed_collections), for which access and manipulated can be synchronized via [atomic operations](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Atomics). Atomics are also used to perform signalling between threads.

Those buffers had been disabled for a while because of [CPU attacks Spectre and Meltdown](https://meltdownattack.com/), but are now enabled by default in Chrome and the new Edge.

## Threads in Emscripten

WebAssembly Threads are [supported in Emscripten](https://emscripten.org/docs/porting/pthreads.html) via the [pthreads library](https://en.wikipedia.org/wiki/POSIX_Threads) and are backed by WebWorkers. 

When threads are created, new WebWorkers are created and [provided with a set of information](https://github.com/emscripten-core/emscripten/blob/4bd0bc3817d06dc5c6cd0178d7b2754248f556e9/src/worker.js#L192-L219), such as stack size, thread ID, shared memory, etc... The same WebAssembly module as the main loop one is loaded in memory in the worker, and [executes the entry point](https://github.com/emscripten-core/emscripten/blob/4bd0bc3817d06dc5c6cd0178d7b2754248f556e9/src/worker.js#L235) requested for the thread.

One interesting aspect of threading is that the creation of WebWorkers needs main loop to yield. This means that if the main loop does not yield control back to the environment, threads may never get the chance to start. That's why Emscripten provides a way set of workers (none by default) to be created before executing any code.

At this point, it is important to note that if the **atomics** feature is not enabled in the browser (e.g. in Firefox or Safari), the emscripten built app will fail to start. This will most probably one of the reason that the [Uno.Wasm.Bootstrap project](https://github.com/unoplatform/Uno.Wasm.Bootstrap) will include multi-configuration generation based on browsers capabilities.

## Threads in .NET for WebAssembly

.NET for WebAssembly now supports the ability to create threads, as the runtime (Mono) uses pthreads. All the existing internal .NET threading APIs have been enabled to make use of pthreads as they do for other platforms, and the `System.Threading` becomes available for use.

With this it becomes possible to use `Monitor` (with `lock()` statements), `AutoResetEvent` and `ManualResetEvent` and other synchronization primitives are working as intended between threads.

The `ThreadPool` is also available, along with `System.Threading.Thread.ManagedThreadId`, thread names and thread local storage (TLS).

## Trying out WebAssembly Threading with Uno.Wasm.Bootstrap

The [Uno.Wasm.Bootstrap package](https://github.com/unoplatform/Uno.Wasm.Bootstrap) provides the configuration to [enable threading](https://github.com/unoplatform/Uno.Wasm.Bootstrap#threads-support) in .NET for WebAssembly by using [the latest 1.1-dev package](https://www.nuget.org/packages/Uno.Wasm.Bootstrap), and changing the active runtime mode.

This can be done in the project file like this:

```xml
<MonoWasmRuntimeConfiguration>threads-release</MonoWasmRuntimeConfiguration>
```

After that, creating a thread becomes possible (view the [full project here](https://github.com/jeromelaban/Wasm.Samples/tree/master/Threading/WasmThreading)):

<script src="https://gist.github.com/jeromelaban/f4b511c85631e3a8b390409db29159a2.js"></script>

Which produces the following output:

```
[tid:1] Startup
[tid:1] Waiting for completion source
[tid:2] Thread begin
[tid:2] Waiting for event
[tid:1] Got task result, raising event
[tid:1] Main thread exiting
[tid:2] Got event, terminating thread
```

The execution is now interleaved properly, as expected when running this sample in common .NET environment.

This sample is build to work with no available upfront WebWorkers, which is why the `Run` method is invoked as Fire and Forget, so that the main loop can get the chance to start the workers.

The bootstrapper configuration does not yet provide a way to change the preexisting workers, but will soon get it.

## Threading affinity

API thread affinity is a tricky subject. Most browser APIs can only be invoked from the main loop, such as DOM manipulation.

Emscripten provides a [feature called "Proxying"](https://emscripten.org/docs/porting/pthreads.html#proxying) which detects APIs need to be invoked on the main loop. The user code is then rewriten to create a blocking call in the worker until the method execution finishes on the other thread. This execution is done through WebWorkers message passing, but this is not something that we'll be able to use in .NET, as IL does not use the proper APIs to executed proxied code.

Along with those limitations are the inability for the main loop to be blocked. Emscripten provides a way to [seemingly block the main](https://emscripten.org/docs/porting/pthreads.html#blocking-on-the-main-browser-thread) thread through spinlocks, though that's generally not a good idea for the end user perceived performance.

## Coming next...

I'll continue the story on how enable threading in the `CoreDispatcher.RunAsync()` in the Uno Platform.