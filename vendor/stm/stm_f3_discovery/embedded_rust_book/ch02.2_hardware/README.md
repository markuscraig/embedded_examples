# Ch2.2 - Hardware

# Create Project

Create the `app` project from the `cortex-m-quickstart` template:
```
$ cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart

🤷   Project Name : app
✨   Done! New project created

$ cd app
```

# Set Default Target

Set the default compilation target to `thumbv7em-none-eabihf` in `.cargo/config.toml`
```
target = "thumbv7em-none-eabihf" # Cortex-M4F and Cortex-M7F (with FPU)
```

# Set Memory Regions

Update the `memory.x` file with the memory regions of the `STM F3 Discovery` board:
```
MEMORY
{
  /* NOTE 1 K = 1024 bytes */
  FLASH : ORIGIN = 0x08000000, LENGTH = 256K
  RAM : ORIGIN = 0x20000000, LENGTH = 40K
}
```

# Cross-Compile "hello" Example

Cross-compile the `hello` example (using the default build target):
```
$ cargo build --example hello
```

# Start OpenOCD

Verify the `openocd.cfg` file:
```
# Sample OpenOCD configuration for the STM32F3DISCOVERY development board
source [find interface/stlink.cfg]
source [find target/stm32f3x.cfg]
```

Start OpenOCD:
```
$ openocd
```

# Start GDB

Start `arm-none-eabi-gdb` in a separate terminal:
```
$ arm-none-eabi-gdb -q target/thumbv7em-none-eabihf/debug/examples/hello
```

Connect `gdb` to `openocd`:
```
(gdb) target remote :3333
Remote debugging using :3333
0x080039bc in ?? ()
```

Flash the program onto the microcontroller:
```
(gdb) load
Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1518 lma 0x8000400
Loading section .rodata, size 0x414 lma 0x8001918
Start address 0x08000400, load size 7468
Transfer rate: 15 KB/sec, 2489 bytes/write.
```

Enable OpenOCD semihosting:
```
(gdb) monitor arm semihosting enable
semihosting is enabled
```

Set a breakpoint at `main` and continue running:
```
(gdb) break main
Breakpoint 1 at 0x8000490: file examples/hello.rs, line 12.
Note: automatically using hardware breakpoints for read-only addresses.
(gdb) continue
Continuing.

Breakpoint 1, hello::__cortex_m_rt_main_trampoline () at examples/hello.rs:12
12	#[entry]
```

# Start GDB with openocd.gdb

Create `openocd.gdb` config file:
```
target extended-remote :3333

# print demangled symbols
set print asm-demangle on

# detect unhandled exceptions, hard faults and panics
break DefaultHandler
break HardFault
break rust_begin_unwind

monitor arm semihosting enable

load

# start the process but immediately halt the processor
stepi
```

Start `arm-none-eabi-gdb` with the `openocd.gdb` config file:
```
$ arm-none-eabi-gdb -q -x openocd.gdb target/thumbv7em-none-eabihf/debug/examples/hello
```

# Start GDB as a Runner

Edit `.cargo/config.toml` runner:
```
[target.'cfg(all(target_arch = "arm", target_os = "none"))']
runner = "arm-none-eabi-gdb -x openocd.gdb"
```

Run with the custom cargo runner:
```
$ cargo run --example hello

    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
     Running `arm-none-eabi-gdb -q -x openocd.gdb target/thumbv7em-none-eabihf/debug/examples/hello`
Reading symbols from target/thumbv7em-none-eabihf/debug/examples/hello...
hello::__cortex_m_rt_main () at examples/hello.rs:20
20	    loop {}
Breakpoint 1 at 0x8000bb0: file /Users/Mark/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.15/src/lib.rs, line 570.
Note: automatically using hardware breakpoints for read-only addresses.
Breakpoint 2 at 0x8001906: file /Users/Mark/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.15/src/lib.rs, line 560.
Breakpoint 3 at 0x800169a: file /Users/Mark/.cargo/registry/src/github.com-1ecc6299db9ec823/panic-halt-0.2.0/src/lib.rs, line 32.
Breakpoint 4 at 0x8000490: file examples/hello.rs, line 12.
semihosting is enabled

Loading section .vector_table, size 0x400 lma 0x8000000
Loading section .text, size 0x1518 lma 0x8000400
Loading section .rodata, size 0x414 lma 0x8001918
Start address 0x08000400, load size 7468
Transfer rate: 15 KB/sec, 2489 bytes/write.
halted: PC: 0x08000402
0x08000402 in cortex_m_rt::Reset ()
    at /Users/Mark/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.15/src/lib.rs:497
497	pub unsafe extern "C" fn Reset() -> ! {
(gdb)
```
