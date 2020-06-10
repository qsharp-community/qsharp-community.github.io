---
title: "Standalone Q# console applications"
author: filipw
date: 2020-06-10
categories:
  - blog
tags:
  - contributed-post
  - documentation
  - standalone
---

[Microsoft Quantum Development Kit](https://docs.microsoft.com/en-us/quantum/welcome) is constantly evolving, continuously bringing new features to Q# developers. One of the excellent recent features was the introduction of standalone Q# command line applications, allowing the Q# users to create Q# programs without a need of a separate host application.

In this post we will have a closer look at that feature, and examine how it is bootstrapped under the hood.

### Hello @EntryPoint

If you worked through any of the older Q# documentation or tutorials, you are surely accustomed to building hybrid quantum programs - with quantum logic in Q#, and the host application being written in another language programs. That was because Q# had no notion of a self contained executable - instead, the executable host was supposed to be built in a mainstream language (C#, F#, Python), and that host would then interop with your Q# algorithms, invoking them out of process.

The new `@EntryPoint()` feature shipped in QDK 0.11.2004.2825, which was released on [30 April 2020](https://docs.microsoft.com/en-us/quantum/relnotes/), and finally allows us Q# developers to create fully standalone Q# programs, greatly simplifying the way we work with the language and its quantum runtime. Q# developers can now use the `@EntryPoint()` attribute to annotate a relevant `operation` as the function that gets executed upon application start up.

Once such Q# program with an `@EntryPoint()` is compiled, the console application can be invoked using the [dotnet SDK](https://dotnet.microsoft.com/download/dotnet-core/3.1):

```
dotnet MyProgram.dll
```

The quantum templates that can be used to quickly generate new programs, and which are also part of the QDK, have already been updated to take advantage of the new feature. Now, creating a new standalone Q# application doesn't create a separate non-Q# host program anymore, but instead leverages the `@EntryPoint()` feature. A most basic example of a Q# program, with a Q# operation that happens to be used as the executable entry point is shown below.

```
namespace MyProgram {
    @EntryPoint()
    operation Start() : Unit {
    	// application logic
    }
}
```

The entry point feature is - naturally - only available to console applications, and not to Q# libraries. Therefore, the `csproj` project file of your program, needs to have a corresponding `<OutputType>Exe</OutputType>` entry in it:

```
<Project Sdk="Microsoft.Quantum.Sdk/0.11.2006.403">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>

</Project>
```

If it was to be missing, then the compiler would throw a build error indicating that libraries cannot have entry points.

### Passing arguments into a Q# console application

When creating console applications with any general purpose languages, such as for example C#, we are used to being able to pass various arguments to our program. This is typically achieved by using a conventional `Main` signature such as the one below:

```
class Program
{
	static void Main(string[] args)
	{
		// process args array
		// program logic
	}
}
```

As a developer you are then responsible for parsing and validating the `args` array. This works reasonably well, but, especially for richer sets of inputs, tends to be tedious task. On that front, standalone Q# applications offer something much more developer friendly, and that's support for sophisticated function signatures on the entry point operations. That allows the Q# developers to accept application arguments in a strongly typed declarative manner. An example of that technique is shown next:

```
namespace MyProgram {
    @EntryPoint()
    operation Start(number : Int, text : String, flag : Bool) : Unit {
        // application logic
    }
}
```

In the snippet above, the Q# application will accept command three line arguments, the names of which correspond to the ones declared in code. We can then invoke such application using, the following command line syntax:

```
dotnet MyProgram.dll --number 100 --text foo --flag true
```

By convention, each argument is mandatory, and the program won't execute without all three of them being passed in.

Aside from the three example types used in the sample above, other types that are supported as arguments on the entry points are several of the Q# numeric types (such as `BigInt` or `Double`), `Range` and `Unit` types, single dimension arrays of built-in types, Pauli matrices (I, X, Y, Z) and Q# `Result` type (`One` and `Zero`).

One more interesting aspect of the feature that's worth mentioning, is that if an argument on your entry point operation is using a camel case name - for example `operation Start(importantEntries : String[]) : Unit`, then the generated corresponding command line argument would be converted into kebab case - `important-entries`. That's because typically command line applications use kebab case over camel case argument naming schemes.

Overall, this excellent functionality is achieved because the Q# runtime is now actually shipping with [System.CommandLine](https://github.com/dotnet/command-line-api) library, which is used to provide binding and conversion of arguments. Under the hood, the compiled .NET Core assembly actually contains a dynamically generated application entry point that satisfies the CoreCLR entry point requirements and a "driver" type functionality, not much different from the one that previously needed to be rolled out by the developer by hand. The dynamically generated type is named with the esoteric `__QsEntryPoint__` name and it has a generic console application entry point signature of `private static async Task<int> Main(string[] args)`. Those generic arguments are then parsed and converted to the appropriate arguments required by the Q# operation which was marked to be the entry point by the developer. All of that machinery, however, is nicely hidden away from the developer and normally we do not worry about how all of these things are glued together. However, if you feel like looking under the hood, I really recommend to check out the main PR responsible for bringing this feature into Q#, from Sarah Marshall - it can be found [here](https://github.com/microsoft/qsharp-runtime/pull/169/files).

### Documentation and help

Q# console applications have another excellent feature to them - also related to the usage of `System.CommandLine` library - and that's automatic help generation for the Q# console application. Each compiled application automatically gets three built-in arguments `-?, -h, --help` which can be used to display the application's help. The help includes documentation about all the typed arguments our entry point operation exposes. As users, we simply need call one of the following:

```
dotnet MyProgram.dll -?
dotnet MyProgram.dll -h
dotnet MyProgram.dll --help 
```

For our above example with number/text/flag arguments, the help displays:

```
Usage:
  MyProgram [options] [command]

Options:
  --number <number> (REQUIRED)    
  --text <text> (REQUIRED)        
  --flag (REQUIRED)               
  -s, --simulator <simulator>     The name of the simulator to use.
  --version                       Show version information
  -?, -h, --help                  Show help and usage information

Commands:
  simulate    (default) Run the program using a local simulator.
```

This looks already very cool, and aside from the built-in options - the help itself, the version display and the ability to choose a simulator - we also see an entry for each of the three input arguments that were defined by us on the operation marked as entry point. What's even better, is that the help generator integrates the documentation comments, should we put any on the Q# entry point. The next snippet shows our original entry point, extended with the documentation:

```
namespace MyProgram {
    /// # Summary
    /// A program that runs a cool quantum computing algorithm
    /// # Input
    /// ## number
    /// Some important number
    /// ## text
    /// Some cool text
    /// ## flag
    /// A useful flag
    @EntryPoint()
    operation Start(number : Int, text : String, flag : Bool) : Unit {
        // application logic
    }
}
```

If we were to run the help again, it would look the following way - notice the comments being part of the new help output.

```
MyProgram:
  A program that runs a cool quantum computing algorithm

Usage:
  MyProgram [options] [command]

Options:
  --number <number> (REQUIRED)    Some important number
  --text <text> (REQUIRED)        Some cool text
  --flag (REQUIRED)               A useful flag
  -s, --simulator <simulator>     The name of the simulator to use.
  --version                       Show version information
  -?, -h, --help                  Show help and usage information

Commands:
  simulate    (default) Run the program using a local simulator.
```

### Validation and diagnostics

The entry point validation is also already integrated into the Q# compiler. The compiler now contains a set of diagnostics specifically dedicated to helping Q# developers diagnose problems with their Q# entry points. Those diagnostics are summarized in the table below. Please note that this code in the compiler is under active development and the detail level of the diagnostics, as well as their IDs may change.

| Diagnostic ID        | Description           |
| ------------- |-------------|
| 6230 | Entry point attribute is placed on an invalid symbol |
| 6231 | Qubit in the entry point signature is not allowed      |
| 6232 | Callable type in the entry point signature is not allowed      |
| 6233 | User defined type in the entry point signature is not allowed        |
| 6234 | Inner tuple in the entry point signature is not allowed        |
| 6235 | Array of arrays in the entry point signature is not allowed       |
| 6236 | Other entry point already exists      |
| 6238 | Entry point is completely missing      |
| 6239 | Specializations other than default body are not allowed      |
| 6240 | Argument names are not unique      |
| 6241 | Entry point in the library is not allowed      |

### Summary

The ability to create standalone Q# programs is an excellent addition to the Q# language landscape. It is a sign of the maturing Q# application runtime, drastically simplifies working with the language and removes barriers of entry for newcomers.

These self contained Q# applications are also going to be supported in Azure Quantum, which is an even more exciting piece of news, emphasizing the important role of this approach to building Q# quantum programs.
