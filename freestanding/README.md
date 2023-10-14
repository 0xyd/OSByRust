# Free Standing Binary
A binary that does not depend on operating systems. Things like standard library, threads, heap memory, and networks.

## Things to do
1. Disable Standard Library
1. Implement Own Panic
1. Language Items
1. Handle Linker Errors

## Disable Standard Library
To disable standard library, `no_std` attribute is added into the `main.rs`. After that, the `println!` macro is no longer usable since it is part of the standard library.
```rust
#![no_std]
fn main(){}
```
## Implement Own Panic
Panic handler is the basic component of a rust program. However, it is also part of the standard library so it is necessary to implement one on our own.

According to the official document, "Rust Core library is the dependency-free foundation of The Rust Standard Library". In the core library, there is `PanicInfo` parameter which handles varied panic messages. Since the panic function doesn't need to return any value, the panic handler has never return type `!`. Currently, there is not thing we need to handle so it only has a loop inside.
```rust
#[panic_handler]
fn panic(_info: &PanicInfo) -> ! {
    loop {}
}
``` 

## Language Item
Language items are required by rust compiler because rust needs to know how to deal with types. Hence, it is often for us to add `trait` on types. However, customizing language items are not recommanded due to their lack of stability (ex: no type checked). 

### Language Item: `eh_personality`
`eh_personality` marks function with stack unwinding so that memory are freed when the destructors of functions are invoked. The destructors are raised when an exception happens as well. When an exception occurs, the stack will be unwinded by functions' destructors and the parent thread can get the `panic` info and continue. However, the stack unwinding is heavily relied on implementation of OS and we have to disable it if we want to build a standalone rust. To disable the unwinding stack, we assign panic to abort in `Cargo.toml` file. Note that panic for dev and released both have to be aborted.

```rust
[profile.dev]
panic = "abort"

[profile.release]
panic = "abort"
```

### Language Item: `start`
After disabling stack unwinding, the compiler still complains that it needs `start` item.
```bash
cargo build
   Compiling freestanding v0.1.0 (/Users/yudelin/projects/OSByRust/freestanding)
error: requires `start` lang_item
```
Before a binary starts, a runtime system is on to prepare everything the upcoming binary needs. For the rust binary, it starts with a C runtime library called `crt0` which is dedicated to set up the environment for C such as creating stacks and placing the arguments to the registers. `crt0` is the utmost start of a rust binary. After `crt0`'s utmost beginning, the rust's runtime which is marked by `start` item is finally triggered. The rust runtime only handles several things like stack overflow guard and backtracing on panic. Once those are done, it calls `main` function. 

Both `crt0` and rust runtime are not existed in standalone scenario, so it is necessary to overwrite an entry point to surrogate `crt0`'s job. Overwriting a `start` item is not necessary since the rust runtime is not used.

To disable the normal entry point, we add new attribute `#![no_main]` in the `main.rc`. Besides, there is nothing going to invoke main function. Thus, we shall just remove it. 

However, we still have to implement a function playing as the entry point of our rust program. As a normal case which the entry point is a C runtime, the entry point we want to build manually shall be something following C's calling conventions.
```rust
#[no_mangle]
pub extern "C" fn _start() -> ! {
    loop {}
}
```
The attribute `no_mangle` tells the compiler not to do the name mangling for this function. The reason is that `_start` is the default entry point name for many systems. Without `no_mangle` attribute, the compiler will rename `_start` into other compiler's defined format. keyword `extern "C"` tells the compiler to invoke this function with C calling convention. We use type never `!` again because this is the beginning of the whole system and there is nothing it needs to return to.

## Linker Errors
In the previous section, we admonish the compiler that this build is a standalone binary and shall not be invoked with C runtime. But the operating system doesn't know that, its linker then try to link every required for a normal build. The solution is to add some options for its linker to tell OS what this binary build is for. In our case, our build is a bare-metal.

## References
1. [Freestanding Rust Binary](https://os.phil-opp.com/freestanding-rust-binary/)
1. [The Rust Core Library](https://doc.rust-lang.org/nightly/core/index.html)
1. [Name mangling](https://en.wikipedia.org/wiki/Name_mangling)
