---
title: "Understanding Frida"
description: "Frida framework allows developers to interact with processes, manipulate memory, hook function, and much more."
date: 2024-12-05
draft: false
---
### What is Frida?

Frida is a powerful and versatile dynamic instrumentation toolkit that allow developers, security researchers, and reverse engineers to:
- Inspect applications and processes at runtime. 
- Modify behaviour dynamically.
- Debug applications

Think of it as a spy that can peek into an app's internals, monitor what's happening, or even change how the app behaves, without access to the app's source code. It enables one to inject custom scripts into running processes, letting you interact with and manipulate the behaviour of the process or application in real-time.

### How Does Frida Work?
Frida consists of:
1. **Frida Server**:
> A small binary that run on the target device that is responsible for interacting directly with the target process. The server listens for connections from the Frida clients (your computer). The server has special priviledges that allows it to:
> - Inject Frida agent (a small piece of Frida code) into the target app.
> - Acts as a middleman between the Frida agent (inside the target process) and the Frida client, i.e. receives commands from the client and sends back results or logs.

2. **Frida Client**:
> It is the interface you use to interact with Frida, it let's you write and send scritps, manage connections, and control the behaviour of the target app or process. It's funcitonality includes:
> - Attaching to target processes
> - Inject scripts
> - Manage Instrumentation.
>  tools provided by the Frida client include `frida`, `frida-ps` and `frida-trace`
3. **Agent Injection**:
> An agent is a small script, written in Javascript, that Frida injects into a target process. The agent acts as a middleman, letting you:
> - Hook a function.
> - Read or modify memory.
> - Intercept and modify data in real time.
> 
> Here's a simple example
> ```javascript
>   Interceptor.attach(Module.findExportByName("libc.so", "system"), {
>       onEnter: function (args) {
>           console.log("Command called: " + Memory.readUtf8String(args[0]));
>       },
>       onLeave: function (retval) {
>           console.log("Command result: " + retval.toInt32());
>       }
>   });
> ```
> To inject this script into a running process, you use the Frida client. Example
> ```bash
> frida -U -n com.example.app -s agent.js
> ```
> Once injected, the agent hooks into the specified function (`system` in this case) and starts logging the commands executed by the app.
> 
> By using Frida's agent injection, you can gain deep insights into how apps work and modify there behaviour dynamically.
> 
4. **Interception and Instrumentation**.
> **Interception** is the process of hooking into a function or method to;
> - observe when it is called.
> - Inspect its arguments and return values.
> - Modify its behaviour
> 
> **Instrumentation** is the process of analyzing or modifying the behaviour of a program in real-time. In Frida, instrumentation allows you to:
> - Inject custom logic into a running process.
> - Analyze function calls, memory, or other runtime behaviour.
> - Modify or bypass restrictions in the program.
> 
> Frida makes it easy to implement both interception and instrumentation in real-time, helping you gain control over apps without needing their source code. It makes this possible through Frida APIs.
> 
> **EXAMPLE: Interception using Interceptor**
> 
> **Scenario:** Hooking a Native function
>
> You want to Intercept the `system()` function in the C standard Library `libc.so` to log every command the app executes:
> ```javascript
> Interceptor.attach(Module.findExportByName("libc.so", "system"), {
>   onEnter: function(args) {
>       var command = Memory.readUtf8String(args[0]);
>       console.log("Command executed: " + command);
>   },
>   onLeave: function(retval) {
>       console.log("Return value: " + retval);
>   }
> });
> ```
> **Explanation**
> 1. **Attaching to the function:**
> > - **Module.findExportByName("libc.so", "system")** locates the address of the `system()` function in the memory.
> 2. **onEnter:**
> > - Executes when the function is called, logs the command passed to `system()`.
> 3. **onLeave:**
>> - Executes when the function completes, logs the return value.
>
> **EXAMPLE: Instrumentation using Java API**
>
> **Scenario:** Hooking a Java Method
>
> You want to log everytime an android app's `sharedpreferences` retrieves a value.
> ```javascript
> Java.perform(function() {
>   let SharedPreferences = Java.use("android.content.SharedPreferences");
>   SharedPreferences.getString.implementation = function(key, defValue) {
>       console.log("Key accessed: " + key);
>       console.log("Default value: " + defValue);
>       let result = this.getString(key, defValue);
>       console.log("Returned value: " + result);
>       return result;
>   }
> });
> ```
> **Explanation**
> 1. **Locate the method:**
> > - **Java.use("android.content.SharedPreferences")** locates the `SharedPreferences` class.
> 2. **Override the method:**
> > - Replace `getString` with custom implementation, logs the key being accesses, it's default value, and the result.
> 3. **call original implementation:**
>> - **this.getString(key, defValue)** calls the original method so the app continues working as expected.

Frida's ability to dynamically inject and execute code makes it an invaluable tool for analyzing and modifying apps on the fly. Whether troubleshooting bugs, analyzing malware, or bypassing restrictions, Frida provides the flexibility and control you need.

With Frida, the app's runtime is your playground!
