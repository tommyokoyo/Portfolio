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
>           let argPassed = Memory.readCString(args[0]);
>           console.log("Argument passed: " + argPassed);
>       },
>       onLeave: function(retval) {
>           console.log("Return value: " + retval);
>       }
>   });
> ``` 
>
3. **Java**
> The `Java` API is used to work with Java code in Android applications. You can enumerate classes, hook methods, or call methods at runtime.
> 
>**Main Methods**
> 
> - `Java.perform()`: Executes a callback in the context of the Java runtime.
> - `Java.use(className)`: Returns a wrapper for a Java class that you can interact with (call methods, access fields).
> - `Java.enumerateLoadedClasses()`: Lists all loaded classes in the application.
> - `Java.enumerateClassLoaders()`: Lists all class loaders, useful for accessing hidden classes.
> - 'Java.choose(className, callback)': Finds all live instances of a given class.
>
> **Example:**
>
> Hook a method in a Java class:
> ```javascript
>   Java.perform(function() {
>       let myClass = Java.use("com.example.MyClass");
>       myClass.myMethod.implementation = function (arg1, arg2) {
>           console.log("Original arg1: " + arg1 + ", arg2: " + arg2);
>           return this.myMethod("newArg1, newArg2"); //call the method with new args
>       }
>   });
> ``` 
>
> List all methods of a class:
> 
> ```javascript
>   Java.perform(function() {
>       let myClass = Java.use("com.example.MyClass");
>       console.log(myClass.class.getMethods());
>   });
> ``` 
> 
> Access a static field of a class:
> 
> ```javascript
>   Java.perform(function() {
>       let myClass = Java.use("com.example.MyClass");
>       console.log("Static field value: " + myClass.myStaticField.value);
>   });
> ``` 
4. **Memory**
> The `Memory` API is used to read, write and scan memory regions of a target process. It's useful for tasks like dumping or altering values directly in memory.
>
> **Main Methods**
> 
> - `Memory.read*()`: Reads data from a specific memory address.
>   - `Memory.readUtf8Strings(address)`: Reads a null terminated UTF-8 string.
>   - `Memory.readByteArray()`: Reads a raw byte array.
> - `Memory.write*()`: Writes data to  a specific memory address.
>   - `Memory.writeUtf8String(address, string)`: Writes a UTF-8 string.
>   - `Memory.writeInt(address, value)`: Writes an integer.
>
> **Example:**
>
> Read and write to Memory:
> ```javascript
>   let baseAddress = Module.findBaseAddress("libnative-lib.so");
>   let offset = 0x1234
>   let address = baseAddress.add(offset);
> 
>   console.log("Original value: " + Memory.readInt(address));
>   Memory.writeInt(address, 42); // Change value in memory
>   console.log("Modified value: " + Memory.readInt(address));
> ``` 
>

### How do these API work together?
You can combine these APIs to perform complex tasks, for instance:
- You can use the `Module` api to find the base address of a library or locate a function.
- You can then combine it with the `Interceptor` to hook the function and modify its behaviour.
- You can use `Memory` to manipulate or inspect the memory regions associated with the hooked function.
- For Java code, you can use `Java` to hook Java methods in Android apps and inspect their interactions with native libraries.

With Frida in your toolkit, no app is too sneaky, no function too hidden, and no memory too Untouchable. With great power comes great responsibility - and occasional crash.

Happy hooking!