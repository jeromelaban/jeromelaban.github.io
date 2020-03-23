---
layout: post
title: "C# interop with C/C++ and Rust in WebAssembly"
date: 2020-03-22 00:00:00 -0500
category: Archive
comments: true
tags: [".NET", "C#", "Interop", "Rust", "C", "C++"]
author: Jerome
---

Having the ability to call code written in other languages is increasingly important, as there are many very useful libraries that are getting ported over to WebAssembly. In .NET, the common defined way for doing interop is [P/Invoke and DllImport](https://docs.microsoft.com/en-us/dotnet/standard/native-interop/pinvoke), and .NET for WebAssembly has support for it in the form of static linking of LLVM Bitcode object files.

In this article, I will walk through how to call some simple C/C++ and Rust code from C# in a WebAssembly app.

<!-- more -->

In the general .NET sense, P/Invoke was built to perform dynamic linking with Windows PE Dlls, but has been extended in mono to allow for static linking. This technique is also [used by mono on iOS to call native code](https://github.com/mono/SkiaSharp/blob/d16fd524b0e4c8715fc89abaca3ccfd6fb103b93/binding/Binding/SkiaApi.cs#L10), and allows for a single executable package to contain the code to execute the application. This is what is used for the demos in this article.

## Setting up the C# WebAssembly project

First let's create create a .NET WebAssembly app, using the [Uno WebAssembly Bootstrapper](https://github.com/unoplatform/Uno.Wasm.Bootstrap).

_Note that macOS is not yet supported for the static linking scenario, and that on Windows 10 you'll need to have [WSL installed](https://platform.uno/blog/build-net-aot-for-webassembly-in-visual-studio-with-uno-platform)._

- Create a .NET Standard 2.0 Library in Visual Studio for Windows or using `dotnet new classlib` under linux.
- Replace the content of the project with the following:

    ```xml
    <Project Sdk="Microsoft.NET.Sdk.Web">

        <PropertyGroup>
            <OutputType>Exe</OutputType>
            <TargetFramework>netstandard2.0</TargetFramework>
            <MonoRuntimeDebuggerEnabled>false</MonoRuntimeDebuggerEnabled>
            <WasmShellMonoRuntimeExecutionMode>InterpreterAndAOT</WasmShellMonoRuntimeExecutionMode>
        </PropertyGroup>

        <ItemGroup>
            <PackageReference Include="Uno.Wasm.Bootstrap" Version="1.1.0-dev.426" />
            <DotNetCliToolReference Include="Uno.Wasm.Bootstrap.Cli" Version="1.1.0-dev.426" />
        </ItemGroup>

        <ItemGroup>
            <!-- This automatically includes any Bitcode file for static linking -->
            <Content Include="*.bc" />
        </ItemGroup>

    </Project>
    ```
- Create a file named `Program.cs`
    ```csharp
    using System;

    namespace MyApp
    {
        public class Program
        {
            static int Main(string[] args)
            {
                return 0;
            }
        }
    }
    ```
- Build the app once. This will download the .NET WebAssembly SDK, and install emscripten for the app.

## Build a C/C++ library

To build a C/C++ library, we'll need to create a simple file that contains an `extern "C"` exported function, in order to have a signature and calling convention that can be used properly with .NET P/Invoke.

In the WebAssembly app, let's create a folder named `myclib` and add a file named `mylib.cpp`:

```cpp
#include <stdio.h>

extern "C" {
    int cpp_add(int a, int b) {
        return a + b;
    }
}
```

Save the file, and open a bash or WSL window.

We can go the path containing the file using the great `wslpath` tool:
```bash
cd `wslpath "C:\YourPathToYourProject\myclib"`
```
We'll need to initialize emscripten:
```bash
source ../obj/emsdk-*/emsdk/emsdk_env.sh
```
Then build the library:
```bash
emcc mylib.cpp -r -o ../myclib.bc -s WASM=1
```

At this point, we've generated a Bitcode file that can be used by the .NET tool chain, and for which an `extern "C"` marked function can be called from C#.

## Call the C++ function from C#

Now that we have our library built, we can update our C# program to make the C++ function callable:

```csharp
using System.Runtime.InteropServices;

public class Program
{
    [DllImport("myclib")]
    private static extern int cpp_add(int a, int b);

    static int Main(string[] args)
    {
        Console.WriteLine($"cpp_add: {cpp_add(21,21)}");
        return 0;
    }
}
```

When building the app, and running it, this will appear in the browser's console:

```
cpp_add: 42
```

## Build a Rust static library

Following a similar path, to build a Rust static library we'll need to create a simple file that contains a function marked with the [`#[export_name]` attribute](https://doc.rust-lang.org/reference/abi.html#the-export_name-attribute), so that it can be found via P/Invoke.

In the WebAssembly app, let's create a folder named `myrustlib` and add a file named `mylib.rs`:

```rust
mod tests {
    #[export_name = "rust_add"]
    pub extern "C" fn rust_add(a: u32, b: u32) -> u32 {
        return a + b;
    }
}
```

Save the file, and open a bash window.

Let's [setup Rust](https://www.rust-lang.org/tools/install), `Rustup` and add `Cargo` to your path:
```bash
curl https://sh.rustup.rs -sSf | sh
set PATH=$PATH:$HOME/.cargo/bin
```

Then setup WebAssembly support for Rust:
```bash
rustup install stable
rustup default stable
rustup target add wasm32-unknown-emscripten
```

We'll need to initialize emscripten here as well, if not done previously:

```bash
source ../obj/emsdk-*/emsdk/emsdk_env.sh
```

Now we can go the path containing the file:

```bash
cd `wslpath "C:\YourPathToYourProject\myrustlib"`
```

Then build the library

```bash
rustc --target=wasm32-unknown-emscripten mylib.rs --crate-type=staticlib -o ../myrustlib.bc
```

In a similar way we've done this for C++, the Bitcode file is now available for the .NET tool chain to use. The `staticlib` parameter is important as it forces the rust compiler to create a standalone library, with all its internal support code included.

## Calling the C++ function from C#

With the Rust library built, we can update our C# program to make the Rust function callable:

```csharp
using System.Runtime.InteropServices;

public class Program
{
    [DllImport("myclib")]
    private static extern int cpp_add(int a, int b);

    [DllImport("myrustlib")]
    private static extern int rust_add(int a, int b);

    static int Main(string[] args)
    {
        Console.WriteLine($"cpp_add: {cpp_add(21,21)}");
        Console.WriteLine($"rust_add: {rust_add(21,22)}");
        return 0;
    }
}
```

When building and run the app again, the following will appear in the browser's console:

```
cpp_add: 42
rust_add: 43
```

## Current set of restrictions for P/Invoke

Under the covers, mono is generating a table of methods marked DllImport, and [generates a set of callable methods](https://github.com/mono/mono/blob/7038b1a4261f86dac2fda4f3894f397bddf88f2c/mcs/tools/wasm-tuner/tuner.cs#L180-L183) from referenced external libraries. 

The runtime needs to do so in order to determine what to call. Once a library and function has been found, the runtime [has to determine the signature of the function](https://github.com/mono/mono/blob/ac1a12f9971fc7ba4f8f675af020dc18fd9d35ee/mono/mini/wasm_m2n_invoke.g.h#L1021) in a [pre-defined signatures list](https://github.com/mono/mono/blob/6c0bfdc3f3d5855d27628112a505ba01bdfc4584/mono/mini/m2n-gen.cs#L30).

If the native function signature is not in the list, you may encounter the `CANNOT HANDLE COOKIE XXXX` assertion [defined here](https://github.com/mono/mono/blob/ac1a12f9971fc7ba4f8f675af020dc18fd9d35ee/mono/mini/wasm_m2n_invoke.g.h#L1823). If you're facing this error, you may want to adjust your native function signature so it finds itself in the [pre-defined signatures list](https://github.com/mono/mono/blob/6c0bfdc3f3d5855d27628112a505ba01bdfc4584/mono/mini/m2n-gen.cs#L30).

All of this is caused by the fact that, for security reasons, WebAssembly functions cannot be called with a set of parameters unknown at compile time; a technique the mono runtime has to use when calling functions through P/Invoke.

If you're wondering how to determine the cookie of a function, as an example this function:

```
static extern int cpp_add(int a, int b);
```

has the cookie `III`, where the first character defines the return type.

Another example with this function:
```
static extern void OtherFunction(int a, double b, float c);
```

will have the `VIDF` cookie.

## WebAssembly validations

Along with the restrictions of the P/Invoke list, making sure that the signature of the function tagged in the C# with `DllImport` matches the function defined in the other libraries. In case of a mismatch, browsers will raise an error such as `RuntimeError: function signature mismatch`.

## Next up...

We'll discuss how to use strings back and forth in both environments.