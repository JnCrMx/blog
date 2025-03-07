---
title: "Creating a Website with modern C++ and WebAssembly"
date: 2025-03-07T16:12:17+01:00
draft: true
toc: true
images:
tags:
  - cpp
  - wasm
  - modern-cpp
  - cmake
  - webdev
  - fun
---
Heyo and welcome to my little journey of creating [my personal website](https://jcm.re) using mainly
(very) modern C++, WebAssembly and some minimal self-written JavaScript bindings.

## WHY?!
Mainly, because I can ;)

I know that I am making stuff needlessly complicated for myself (making an equivalent to the website purely in JavaScript
would most likely be faster and easier), but I love C++ (weird, I know) and I wanted to see how far I could get and
how well JavaScript concepts would map to C++ (spoiler: relatively well actually).
Also it gives me additional experience when it comes to working with modern C++ features such as modules, coroutines
and using `std::expected` instead of exceptions (which are unavailable in the toolchain I am using for WebAssembly).

## Running C++ in the browser
First of all, how can one run C++ in the browser?
It's easy: Compile it to WebAssembly, load it via JavaScript, run the `main()` function and let the C++ code take over.

So, let's first of all begin with the toolchain and configuration we need to target WASM.
I decided to use the `clang` C++ compiler, which is based on LLVM and can therefore target lots of different backends,
including WebAssembly.

Let's give it try with a very simple "Hello World" program:
```cpp
#include <iostream>

int main() {
  std::cout << "Hello World\n";
  return 0;
}
```
On Ubuntu 24.04, I `apt install`ed the `clang-19` package and gave compiling it a try:
```bash
clang++-19 -target wasm32 test.cpp -o test.wasm
```
But no, `clang` complains, that is cannot find the `iostream` header.
This is because by just targeting `wasm32` we do not have access to a standard library,
and therefore cannot use `iostream` or any other headers from the standard library.

To address this, we can make use of `libc++-wasi`, which is a version of the [`libc++` standard library](https://libcxx.llvm.org/)
targeting the [**W**eb**A**ssembly **S**ystem **I**nterface](https://wasi.dev/).
We will not be using WASI as the interface to communicate with the JavaScript, but having a standard library on our side is
still a win.

So, let's `apt install` the packages `libc++-19-dev-wasm32` (the standard library itself), `libclang-rt-19-dev-wasm32` (for providing some builtin functions) and `lld-19` (for linking everything) and give it another try:
```bash
clang++-19 -target wasm32-wasi test.cpp -o test.wasm
                         ^^^^^ specify that we are using WASI
```
This time the compilation goes well, but we fail at the linker step:
```
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(stdlib_new_delete.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(stdlib_new_delete.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(stdexcept.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(stdexcept.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(string.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(string.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(string.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(string.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(new_helpers.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(new_helpers.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(ios.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(ios.cpp.o): undefined symbol: __cxa_throw
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(locale.cpp.o): undefined symbol: __cxa_allocate_exception
wasm-ld-19: error: /usr/lib/wasm32-wasi/libc++.a(locale.cpp.o): undefined symbol: __cxa_throw
clang++-19: error: linker command failed with exit code 1 (use -v to see invocation)
```
As far as I understand, this is because of the limited exception support of WASI and sadly cannot be fixed
by disabling exceptions (because the symbols are present in the precompiled, static standard library).

To work around this, we can simply define our own exception support functions:
```cpp
#include <iostream>

int main() {
  std::cout << "Hello World\n";
  return 0;
}

extern "C" {
void* __cxa_allocate_exception(size_t) { __builtin_trap(); return nullptr; }
void __cxa_throw(void *, void *, void (*) (void *)) { __builtin_trap(); }
}
```
Since I do not intend to support or use exceptions, I've decided to just trap the execution using
my all time favorite `__builtin_trap()` should any exception to allocated or thrown.

Now our program compiles and we can think about how to load it using JavaScript.
This is luckily rather simple and straightforward (for now) and can be done like this:
```js
const { instance } = await WebAssembly.instantiateStreaming(
  const importObject = {}
  fetch("./test.wasm"), importObject
);
instance.exports.main(); // calls the main() function
```

Trying this gives us yet another error, this time in the browser console:
```
Uncaught (in promise) TypeError: WebAssembly.instantiate(): Import #0 "wasi_snapshot_preview1": module is not an object or function
```

This happens because WASI specifies certain functions that need to be available for the WebAssembly module and hence must be passed
using the `importObject`.
Since I do not intend to use WASI (it is mostly means for regular programs) and instead my own interface, I just created stubs for
these required functions:
```js
const importObject = {
  wasi_snapshot_preview1: {
    fd_close: () => { console.log("fd_close"); },
    fd_seek: () => { console.log("fd_seek"); },
    fd_read: () => { console.log("fd_read"); },
    fd_write: () => { console.log("fd_write"); },
    fd_fdstat_get: () => { console.log("fd_fdstat_get"); },
    fd_prestat_get: () => { console.log("fd_prestat_get"); },
    fd_prestat_dir_name: () => { console.log("fd_prestat_dir_name"); },
    environ_get: () => { console.log("environ_get"); },
    environ_sizes_get: () => { console.log("environ_sizes_get"); },
    proc_exit: () => { console.log("proc_exit"); },
  }
}
```

Now, time for the next error:
```
Uncaught (in promise) TypeError: instance.exports.main is not a function
```
Looks like JavaScript fails to find the `main()` function.
This happens because we never specified that it should be exported to be availabe for JavaScript.
We can fix this by adding a `clang::export_name("main")` attribute to our main function:
```cpp
#include <iostream>

[[clang::export_name("main")]]
int main() {
  std::cout << "Hello World\n";
  return 0;
}

extern "C" {
void* __cxa_allocate_exception(size_t) { __builtin_trap(); return nullptr; }
void __cxa_throw(void *, void *, void (*) (void *)) { __builtin_trap(); }
}
```

Now we are back to errors from C++, this time a very obscure one:
```
Uncaught (in promise) RuntimeError: memory access out of bounds
    at test.wasm.std::__2::basic_ostream<char, std::__2::char_traits<char>>::sentry::sentry(std::__2::basic_ostream<char, std::__2::char_traits<char>>&) (http://127.0.0.1:5500/test.wasm:wasm-function[163]:0x4769)
    at test.wasm.std::__2::basic_ostream<char, std::__2::char_traits<char>>& std::__2::__put_character_sequence[abi:ne190101]<char, std::__2::char_traits<char>>(std::__2::basic_ostream<char, std::__2::char_traits<char>>&, char const*, unsigned long) (http://127.0.0.1:5500/test.wasm:wasm-function[14]:0xa79)
    at test.wasm.std::__2::basic_ostream<char, std::__2::char_traits<char>>& std::__2::operator<<[abi:ne190101]<std::__2::char_traits<char>>(std::__2::basic_ostream<char, std::__2::char_traits<char>>&, char const*) (http://127.0.0.1:5500/test.wasm:wasm-function[12]:0x9bd)
    at test.wasm.__original_main (http://127.0.0.1:5500/test.wasm:wasm-function[11]:0x949)
    at init (http://127.0.0.1:5500/test.html:28:30)
```
After some digging I found out that the constructors of global objects are not called automatically
and big parts of the standard library are therefore still in an invalid uninitialized state.

To fix this, we need to call a function called `__wasm_call_ctors`.
We could either do this at the beginning of our `main` function, but I have decided to instead create and export a new function,
which I called `_initialize`:
```cpp
extern "C" void __wasm_call_ctors();
[[clang::export_name("_initialize")]]
void _initialize() {
    __wasm_call_ctors();
}
```

After adding this function, recompiling the WASM module, and modifying the JavaScript to call it before main
everything works as excepted. *Kind of.*

Instead of a glorious "Hello World", we instead only see "fd_fdstat_get" and tons of "fd_write" messages in our console.
This is because we have decided to not implement any of the WASI functions, which would be responsible for handling our
output and as we can see get called to do exactly this.

## Communication between JavaScript and C++
Rather than implementing these functions and using WASI, which for my very special use-case would probably be limiting,
I have decided to build my own JavaScript bindings.

### Calling JavaScript functions from C++ code

Let's for example have a look at a basic `log` function, which is supposed to just log a message to the console:

In C++-land, we define an *imported* function, which we can then use to call JavaScript code.
```cpp
[[clang::import_name("log")]] void log(const char* message, size_t len);

// for convenience
void log(std::string_view sv) {
  log(sv.data(), sv.size());
}

[[clang::export_name("main")]]
int main() {
  log("Hello World");
  return 0;
}
```
We need to pass a char pointer and a length, since we cannot except JavaScript to know about
how exactly a `std::string` works and therefore need to deal with raw pointers instead.

In JavaScript we now need to implement the corresponding function:
```js
const importObject = {
  env: { // name of the default import module
    log: (message, len) => {
      // get a view of the WASM memory exactly where "message" points of length "len"
      const cArray = new Uint8Array(instance.exports.memory.buffer, message, len);
      const msgString = new TextDecoder().decode(cArray); // decode it
      console.log(msgString); // log it
    }
  },
  wasi_snapshot_preview1: {
    // ...
  }
}
```

And **BAM**! It works and we see "Hello World" printed to web console.

Let's recap what exactly happens:
1. JavaScript loads a WebAssembly module and provides it with our JS `log` implementation
2. JavaScript calls our exported C++ `_initialize` function, which takes care of calling global constructors
3. JavaScript calls our exported C++ `main` function
4. The C++ `main` function calls the imported JavaScript `log` function and passes the pointer and length to our "Hello World" string
5. The JavaScript `log` function accesses the memory of the WebAssembly module (which is exactly the memory we are accessing in C++)
   and creates a view of the region that contains our "Hello World" string using the pointer and length we passed to it
6. The JavaScript `log` functions decodes this string and then logs it using `console.log`
7. The JavaScript `log` function returns and gives control back to our C++ `main` function
8. The C++ `main` function returns and gives control back to JavaScript

We can now call any JavaScript code from C++ and build bindings for whatever feature we need!
For example, we could trivially give full control to our C++ code by providing it with an `eval` function.

### Returning values

### Events and callbacks

## Rendering HTML in C++

## Fun with coroutines
