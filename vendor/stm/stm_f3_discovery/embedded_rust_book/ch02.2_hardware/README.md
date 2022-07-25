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