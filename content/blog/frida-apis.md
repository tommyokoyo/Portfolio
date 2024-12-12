---
title: "Frida APIs"
description: "Frida APIs are the programmatic tools provided by the Frida framework."
date: 2024-12-10
draft: false
---
### What are Frida APIs?
They are part of the Frida dynamic instrumentation toolkit, they allow one to interact with and modify the behaviour of applications and processes at runtime. Think of them as tools to 'inject' custom scripts into an application to monitor or manipulate its behaviour.

#### Core Concepts of Frida APIs
- **Injection**
> Frida allows you to inject Javascript code into running processes (like apps in android), they injected script can interact with the application's memory, functions, and data in real time.
- **Runtime Interaction.**
> Once the script is injected, it communicates with the Frida client on your computer, letting you inspect or modify behaviour.
- **Cross-platform.**
> Frida works with native (C/C++) and managed (Java, .Net) applications.

### Key Frida APIs and their Uses
1. **Module**
> The `Module` API is used to interact with loaded libraries or binaries in the target process. You can enumerate their exports, imports, and base addresses.
> 
>**Main Methods**
> 
> - `Module.findExportByName(name, exportName)`:Finds the address of an exported function or variable.
>   - `name`: Name of the Module (for instance `libnative-lib.so`, or `null` for the main executable).
>   - `exportName`: Name of the exported symbol (function or variable).
> - `Module.enumerateExports(name)`: lists all imported symbols of a module.
> - `Module.enumerateImports(name)`: Lists all imported symbols of a module.
> - `Module.findBaseAddress(name)`: Finds the base address of a loaded module.
>
> **Example:**
> 
> Find and list all exports of a module:
> ```javascript
>   let exportedModules = Module.enumerateExports("libc.so");
>   exportedModules.forEach(function (export) {
>       console.log("Name: " + export.name + ", Address: " + export.address);
>   });
> ``` 
>
> Find and call a specific function:
> ```javascript
>   let myFunction = Module.findExportByName("libnative-lib.so", "myFunction");
>   let nativeFunction = new NativeFunction(myFunction, "int", ["int", "int"]);
>   console.log(nativeFunction(10, 20));
> ``` 
 

2. **Interceptor**
> The interceptor API is used to hook into functions in the target application. It allows you to monitor, modify, or replace the behaviour of those functions.
> 
> **Main Methods**
> 
> - `Interceptor.attach()`: Hooks into a specific function to observe or modify its behaviour.
>
>   **Arguments**
> 
>   - `address`: The memory address of the function to hook into (can be obtained using `Module.findExportByName()` or other methods).
>   
>   - `callBacks`: an object defining what to do when a function is entered or exited.
>     - `onEnter(args)`: Called when the function is invoked. `args` is an array-like object containing arguments when passed to the function.
>     - `onLeave(retval)`: Called when the function returns. `retval` represents the return value.
>
> **Example:**
>
> Hook into a function and print its arguments:
> ```javascript
>   let systemCall = Module.findExportByName("libc.so", "fopen");
> 
>   Interceptor.attach(systemCall, {
>       onEnter: function(args) {
>           let variablePassed = Memory.readCString(args[0]);
>           console.log("Argument passed: " + variablePassed);
>       },
>       onLeave: function(retval) {
>           console.log("Return value: " + retval);
>       }
>   });
> ``` 
>
3. **Java**
4. **Memory**
5. **objC**
6. **NativeFunction**