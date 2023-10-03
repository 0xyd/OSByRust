# Free Standing Binary
A binary that does not depend on operating systems. Things like standard library, threads, heap memory, and networks.

## Things to do
1. Disable Standard Library
1. Implement Own Panic
1. Language Item `eh_personality`

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

## Language Item `eh_personality`
Language items are required by rust compiler because rust need to know how to deal with types. Hence, it is often for us to add `trait` on types. 

## References
1. [The Rust Core Library](https://doc.rust-lang.org/nightly/core/index.html)
