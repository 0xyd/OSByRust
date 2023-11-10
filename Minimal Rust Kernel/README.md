# Minimal Rust Kernel

## Things to do
1. The Boot Process
2. A Minimal Kernel
3. Running the Kernel

## The Boot Process

### A "Pre-boot" Process
When the devices are power up, a specific firmware / software routine storing in ROM will perform [POST (power-on self-test)](https://en.wikipedia.org/wiki/Power-on_self-test). Generally speaking, POST's tasks are following:
* Initlialize the firmware (ex: BIOS/UEFI) and verify itself 
* Initialize main memory (RAM) and verify it
* Verify CPU's registers
* Verify fundamental components like timer, DMA, interrupt controller 
* Identify devices available for booting.

### BIOS Boot
BIOS, nickname of Basic Input/Output System, is an old but simple firmware for booting x86 machines. Since BIOS is designed for backward compatibility, every x86 platform will start in 16-bit real mode. UEFI, Unified extensible Firmware Interface, is a newer booting firmware and has more modern features. The author currently uses BIOS as an example so we currently just follow.

When a computer is on, the BIOS is loaded from a special ROM and start to run testing and initialization hardware. After that, it will look for the bootable disks at where `bootloaders` are located. Bootloaders are code store at the first 512 bytes in the disk. However, the size of bootloaders are larger than 512 bytes normally so that the parts over 512 bytes will be loaded subsequentially. 

The bootloader will find the location of the kernel image and load it into memory. It also switches the CPU from 16-bit real mode to 32-bit protected mode. For 64 bit system, the CPU is even put into 64 long-mode from the protected mode for more 64-bit features. After all these mode switching, the bootloader will gain information from BIOS and pass it to the OS.

### (Optional) Multiboot Standard
Since there are many operating systems developed by communities and companies, they might develop there own bootloaders. To solve this, GNU GRUB is invented to unify operating systems that are multiboot compatible. Nowadays, it is very common in Linux systems. 

Although Multiboot standard is a good solution, it has several drawbacks:
* It support protected mode only and we need to manually configure CPU to 64-bit long model
* Its document is abstruse.
* The boot information it provides is complex since it contains information from varied architectures
* GRUB must be installed in the kernel of the host operating system to create a bootable images. However, it is not pre-installed in MacOS and Windows.

Therefore, the authoer didn't cover the details about it in this chapter.

## A Minimal Kernel

### Install Rust Nightly

First, we need to overwrite our current rust version into nightly. We can simply do:
```bash
rustup override set nightly
```
This will install the nightly rust in our rustc toolchain if we didn't install it before. Also, it will force the current build environment into `rust-nightly` version. If we want to check what toolchains we currently have, we can enter command `rustup toolchain list` and we will see something like:
```bash
>>> rustup toolchain list
stable-aarch64-apple-darwin (default)
nightly-aarch64-apple-darwin (override)
```
The information told us that we have nightly installed and it is currently overwrite the default. Note that the nightly version is set in this directory after the command, not just in the current shell.

### Customize you own target

Rust supports our own target by adding a customized JSON file. The example the author addressed is:
```json
{
    "llvm-target": "x86_64-unknown-none",
    "data-layout": "e-m:e-i64:64-f80:128-n8:16:32:64-S128",
    "arch": "x86_64",
    "target-endian": "little",
    "target-pointer-width": "64",
    "target-c-int-width": "32",
    "os": "none",
    "executables": true,
    "linker-flavor": "ld.lld",
    "linker": "rust-lld",
    "panic-strategy": "abort",
    "disable-redzone": true,
    "features": "-mmx,-sse,+soft-float"
}
```
* `llvm-target`: The name of our customized target
* `data-layout`: This attribute defines the memory layout. Each specification is separated by "-". We have the following specifications here:
    * "e": little-endian form.
    * "m:e": ELF mangling
    * "i64:64": The alignment of an integer type is 64 bit (The first 64). The second 64 is the ABI  


### Build

## Reference
1. [A Minimal Rust Kernel](https://os.phil-opp.com/minimal-rust-kernel/)
1. [Power-On Self-Test by wiki](https://en.wikipedia.org/wiki/Power-on_self-test)
1. [What is POST by GeeksforGeeks](https://www.geeksforgeeks.org/what-is-postpower-on-self-test/)
1. [UEFI](https://en.wikipedia.org/wiki/UEFI)
1. [Real mode](https://en.wikipedia.org/wiki/Real_mode)
1. [Bootloader](https://en.wikipedia.org/wiki/Bootloader)
1. [Protected Mode](https://en.wikipedia.org/wiki/Protected_mode)
1. [Long Mode](https://en.wikipedia.org/wiki/Long_mode)
1. [GRU GRUB BY wiki](https://en.wikipedia.org/wiki/GNU_GRUB)
1. [Install Rust Nightly](https://rust-lang.github.io/rustup/installation/index.html#installing-nightly)
1. [Rustup for managing Rust versions](https://dev-doc.rust-lang.org/beta/edition-guide/rust-2018/rustup-for-managing-rust-versions.html)
1. [Customize Targets](https://doc.rust-lang.org/nightly/rustc/targets/custom.html)
1. [Data Layout](https://llvm.org/docs/LangRef.html#data-layout)
1. [Endianness](https://en.wikipedia.org/wiki/Endianness)
1. [Application binary Interface](https://en.wikipedia.org/wiki/Application_binary_interface)