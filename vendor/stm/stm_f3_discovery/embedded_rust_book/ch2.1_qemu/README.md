# QEMU Test

# Create Project from Template

Create the project named `app` from the `cortext-m-quickstart` template:
```
$ cargo generate --git https://github.com/rust-embedded/cortex-m-quickstart

🤷   Project Name : app
```

# Add the Cortext-M3 Target

```
$ rustup target add thumbv7m-none-eabi
```

# Cross-Compile

Cross-compile the project for the Cortex-M3 target:
```
$ cargo build --target thumbv7m-none-eabi
```

NOTE: Since the `thumbv7m-none-eabi` target is set in `cargo.toml`, the following is equivalent:
```
$ cargo build
```

# Inspect the Executable

Use `readobj` to dump the ELF file header:
```
$ cargo readobj --bin app -- --file-headers

    Finished dev [unoptimized + debuginfo] target(s) in 0.07s
ELF Header:
  Magic:   7f 45 4c 46 01 01 01 00 00 00 00 00 00 00 00 00
  Class:                             ELF32
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              EXEC (Executable file)
  Machine:                           ARM
  Version:                           0x1
  Entry point address:               0x401
  Start of program headers:          52 (bytes into file)
  Start of section headers:          846120 (bytes into file)
  Flags:                             0x5000200
  Size of this header:               52 (bytes)
  Size of program headers:           32 (bytes)
  Number of program headers:         4
  Size of section headers:           40 (bytes)
  Number of section headers:         22
  Section header string table index: 20
```

Use `size` to dump the linker section sizes of the executable:
```
$ cargo size --bin app --release -- -A

    Finished release [optimized + debuginfo] target(s) in 0.07s
app  :
section             size        addr
.vector_table       1024         0x0
.text                648       0x400
.rodata                0       0x688
.data                  0  0x20000000
.bss                   0  0x20000000
.uninit                0  0x20000000
.debug_loc           346         0x0
.debug_abbrev       1141         0x0
.debug_info         5692         0x0
.debug_aranges       592         0x0
.debug_ranges       1240         0x0
.debug_str          9458         0x0
.debug_pubnames     2711         0x0
.debug_pubtypes      270         0x0
.ARM.attributes       50         0x0
.debug_frame        1420         0x0
.debug_line         6500         0x0
.comment              19         0x0
Total              31111
```

Disassemble the executable binary:
```
$ cargo objdump --bin app --release -- --disassemble --no-show-raw-insn --print-imm-hex
```

# Build "hello" Example

Cross-compile the `hello` example:
```
$ cargo build --example hello
```

Verify:
```
$ ls -l target/thumbv7m-none-eabi/debug/examples/hello

-rwxr-xr-x  1 Mark  staff  925916 Jul 22 22:59 target/thumbv7m-none-eabi/debug/examples/hello
```

# Run "hello" on QEMU

Run the `hello` executable using QEMU with semi-hosting (allowing the executable to print text to the host console):
```
$ qemu-system-arm -cpu cortex-m3 \
  -machine lm3s6965evb -nographic \
  -semihosting-config enable=on,target=native \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello

Timer with period zero, disabling
Hello, world!
```

# Start QEMU using Cargo Runner

Edit `.cargo/config.toml`:
```
[target.thumbv7m-none-eabi]
runner = "qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel"
```

Run on QEMU using cargo:
```
$ cargo run --example hello --release

Compiling qemu-test v0.1.0 (/Users/Mark/go/src/github.com/markuscraig/embedded_rust_examples/vendor/stm/stm_f3_discovery/embedded_rust_book/qemu-test)
    Finished release [optimized + debuginfo] target(s) in 1.08s
     Running `qemu-system-arm -cpu cortex-m3 -machine lm3s6965evb -nographic -semihosting-config enable=on,target=native -kernel target/thumbv7m-none-eabi/release/examples/hello`
Timer with period zero, disabling
Hello, world!
```

# Debugging using QEMU

Start QEMU in debugging mode (freeze at start-up; wait for GDB connection):
```
$ qemu-system-arm \
  -cpu cortex-m3 \
  -machine lm3s6965evb \
  -nographic \
  -semihosting-config enable=on,target=native \
  -gdb tcp::3333 \
  -S \
  -kernel target/thumbv7m-none-eabi/debug/examples/hello
```

Start ARM GDB (in another terminal):
```
$ arm-none-eabi-gdb -q target/thumbv7m-none-eabi/debug/examples/hello

Reading symbols from target/thumbv7m-none-eabi/debug/examples/hello...
(gdb)
```

Connect GDB to QEMU:
```
(gdb) target remote :3333

Remote debugging using :3333
cortex_m_rt::Reset ()
    at /Users/Mark/.cargo/registry/src/github.com-1ecc6299db9ec823/cortex-m-rt-0.6.15/src/lib.rs:497
497	pub unsafe extern "C" fn Reset() -> ! {
```

List the `main` function:
```
(gdb) list main

6	use panic_halt as _;
7
8	use cortex_m_rt::entry;
9	use cortex_m_semihosting::{debug, hprintln};
10
11	#[entry]
12	fn main() -> ! {
13	    hprintln!("Hello, world!").unwrap();
14
15	    // exit QEMU
```

Set a breakpoint before println:
```
(gdb) break 13
Breakpoint 1 at 0x4c6: file examples/hello.rs, line 13.
```

Continue running and hit breakpoint:
```
(gdb) continue

Continuing.

Breakpoint 1, hello::__cortex_m_rt_main () at examples/hello.rs:13
13	    hprintln!("Hello, world!").unwrap();
```

Step to the next line (printing `Hello, world!` to the host console):
```
(gdb) next

17	    debug::exit(debug::EXIT_SUCCESS);
```

Step to next line to terminate the QEMU process. Exit to quit GDB:
```
(gdb) next
[Inferior 1 (process 1) exited normally]
(gdb) quit
```
