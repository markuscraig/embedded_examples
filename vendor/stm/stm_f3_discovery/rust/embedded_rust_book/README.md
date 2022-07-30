# Embedded Rust Book Tutorial

The source code follows the `Embedded Rust Book`:

* https://docs.rust-embedded.org/book/

# STM F3 Discovery Board

This version of the book uses the `STM F3 Discovery` development board:

* https://www.st.com/en/evaluation-tools/stm32f3discovery.html

The `STM F3 Discovery` board can be purchased online from these distributors:

* https://www.st.com/en/evaluation-tools/stm32f3discovery.html#sample-buy

# Install Dependencies

## Rust Installation

Install Rust:
* https://rustup.rs/

Verify the Rust compiler version:
```
$ rustc -V
rustc 1.62.1 (e092d0b6b 2022-07-16)
```

## ARMv7 Cross-Compiler

For `STM F3 Discovery` board, install the toolchain for the `thumbv7em-none-eabihf` target (ARM Cortex-M4 and M7 with hardware floating-point support):
```
$ rustup target add thumbv7em-none-eabihf
```

## Cargo-Binutils

Install Cargo subcommands which make it easy to use the LLVM tools which are packaged with Rust:
```
$ cargo install cargo-binutils
```

Verify the installation:
```
$ which rust-lld
/Users/Mark/.cargo/bin/rust-lld

$ rust-lld -h
```

## LLVM Tools

Install the LLVM tools:
```
$ rustup component add llvm-tools-preview
```

## Cargo-Generate

Install cargo-generate which is used to generate a project from a template:
```
$ cargo install cargo-generate
```

## ARM GDB Debugger

### MacOS
Install the GDB debugger for ARM architecture:
```
$ brew install armmbed/formulae/arm-none-eabi-gcc
```

Verify:
```
$ arm-none-eabi-gdb -v
GNU gdb (GNU Arm Embedded Toolchain 10.3-2021.07) 10.2.90.20210621-git
```

## QEMU

### MacOS
Install the QEMU emulator:
```
$ brew install qemu
```

## OpenOCD

### MacOS
Install OpenOCD:
```
$ brew install openocd
```

Verify:
```
$ openocd -v
Open On-Chip Debugger 0.11.0
```

# Running OpenOCD

Start OpenOCD with a USB-connected `STM F3 Discovery` board:
```
$ openocd -f interface/stlink.cfg -f target/stm32f3x.cfg

Open On-Chip Debugger 0.11.0
Licensed under GNU GPL v2
For bug reports, read
	http://openocd.org/doc/doxygen/bugs.html
Info : auto-selecting first available session transport "hla_swd". To override use 'transport select <transport>'.
Info : The selected transport took over low-level target control. The results might differ compared to plain JTAG/SWD
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : clock speed 1000 kHz
Info : STLINK V2J37M26 (API v2) VID:PID 0483:374B
Info : Target voltage: 2.883382
Info : stm32f3x.cpu: hardware has 6 breakpoints, 4 watchpoints
Info : starting gdb server for stm32f3x.cpu on 3333
Info : Listening on port 3333 for gdb connections
```
