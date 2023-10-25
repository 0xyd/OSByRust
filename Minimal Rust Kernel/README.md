# Minimal Rust Kernel

## Things to do
1. The Boot Process
2. The Minimal Kernel
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



## Reference
1. [A Minimal Rust Kernel](https://os.phil-opp.com/minimal-rust-kernel/)
1. [Power-On Self-Test by wiki](https://en.wikipedia.org/wiki/Power-on_self-test)
1. [What is POST by GeeksforGeeks](https://www.geeksforgeeks.org/what-is-postpower-on-self-test/)
1. [UEFI](https://en.wikipedia.org/wiki/UEFI)
1. [Real mode](https://en.wikipedia.org/wiki/Real_mode)
1. [Bootloader](https://en.wikipedia.org/wiki/Bootloader)
1. [Protected Mode](https://en.wikipedia.org/wiki/Protected_mode)
1. [Long Mode](https://en.wikipedia.org/wiki/Long_mode)