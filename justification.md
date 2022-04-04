# Audiences

BorrowScript targets the engineering of high level application development, namely:
- Graphical applications
  - Web applications via web assembly
  - Native mobile applications
  - Native desktop applications
- Web servers
- CLI tools
- Embedded applications

# Justification

## GUI Applications

Graphical applications written targeting web, desktop and mobile have seen a lot of innovation and attention. From technologies like React/React Native, Electron/Cordova to Flutter, there is an interest in creating client applications that are portable between platforms, are performant and memory efficient.

While BorrowScript is simply a language and does not describe a framework that manages bindings to the various platform UI APIs - it does allow for a simple, familiar language which addresses the concerns that graphical applications have, making it a great candidate for such a use case.

While its primary focus is targeting the web platform via web assembly, there is no reason we would not be able to see React-Native like platform producing native desktop and native mobile platforms.

### Multi threading

Effective use of multi-threading is critical for making graphical applications feel responsive. 

As an example, web applications are often criticized for their lack of performance when compared to their native counterparts. Native applications however make maximal use of threads and as a result have interfaces that seldom block.

Including a borrow checker into a familiar language used in web development means that concurrency across threads can be used without fear of data races. This will also allow graphical applications to be used on lower end devices as their resources can be utilized more effectively.

### Small Binary

As web assembly requires languages to ship their runtime, it's important for languages to ship as little runtime code as possible. 

A borrow checker allows a compiler to compile the code without the need for a garbage collection implementation or a substantial runtime.

This is valuable for web applications however, while not strictly required, efficient small binaries are also appreciated when distributing native applications and web server applications.

## Web Servers:

Web servers require high and consistent IO performance, maintainable code bases and simple distribution.

There are many languages to choose from in this space making the argument for BorrowScript less compelling on the server side.

There are a few ways that BorrowScript can be competitive in this context, however.

### Performance (speculation)

Rust performance is often compared to C++ and out performs languages like Go and Java. 

While it's too early to guarantee performance, we know that TSCB is a slightly higher level Rust, where the types are replaced with built-ins. 

I would like to see its performance sit between Rust and Go.

### Go-like concurrency with Rust-like safety

Having an application server written in a high level, familiar and maintainable language that promises the concurrently of a language like Go while also ensuring you cannot write data races might be compelling for back-end application developers.

### Small, statically linked binaries are good for containers

While this is an implementation detail of the compiler, entirely self contained binaries that embed their dependencies (unless explicitly specified) are fantastic from a distribution and security perspective.

It allows engineers to distribute containerized applications based on `scratch` images.

### No GC is good for performance consistency

Languages without garbage collection have consistent performance as they avoid locks originating from garbage collection sweeps.

This is especially noticeable in applications with lots of activity - such as chat servers.

### Good candidate for PaaS, FaaS

Lastly; small, self contained, memory and performance optimized binaries make for great candidates in PaaS or FaaS contexts (Lambda, App Engine, Heroku, etc). 
