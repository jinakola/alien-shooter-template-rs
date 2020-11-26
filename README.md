# Alien shooter, Rust version

This branch has been edited to run on an undetermined linux machine configuration. See `standalone` for a version that runs on an undetermined machine configuration. See `master` branch for a version that runs at TC219.

## Setup

1. Install Rust:
    - Go to [rustup.rs](https://rustup.rs/) and follow the instructions.
2. Install the nightly toolchain:
    - ```rustup install nightly```.
    - We need the nightly toolchain to compile a line of assembly to enable interrupts via a `libxil` C-FFI library.
3. Install the cross-compiler:
    - `rustup target add armv7a-none-eabi`.
4. Make sure you have Xilinx SDK installed. More information about installing Xilinx SDK at https://github.com/hegza/comp.ce.100-rust-exercise-guide.

## Build and run
- Build the binary:
    - `cargo build`.
    - You may have to first specify the location of Xilinx SDK with `export XILINX_SDK=/location/of/Xilinx/SDK/20xx.x` or `$Env:XILINX_SDK = "C:/Apps/Xilinx_Vivado2017/SDK/2017.2`.
    - If you get a message like "'arm-none-eabi-gcc' not found", you will have to specify the location of the linker at ".cargo/config" in place of "arm-none-eabi-gcc". You could use the Xilinx' linker, found at "C:/Apps/Xilinx_Vivado2017/SDK/2017.2/gnu/aarch32/nt/gcc-arm-none-eabi/bin/arm-none-eabi-gcc" on TC219 machines.
    - The first build will take a while, since it has to download all dependencies.
    - Consecutive builds from the same terminal can just use `cargo build` without re-setting the environment variable as the Xilinx dependent dependency is cached.
- Run the binary on a connected PYNQ-Z1:
    - Open a "Xilinx Software Command Line Tool 2017.2" -prompt and connect the PYNQ.
    - Navigate to the project directory.
    - Run `source run_on_pynq.tcl` to initialize and run the built program on a connected PYNQ.
    - Alternatively on Windows, double-click on the `run_on_pynq.tcl` script. If Windows asks for a program to run the script with, find "Xilinx Software Command Line Tools 2017.2".
    - See [Open an input-output interface to the board](https://github.com/hegza/comp.ce.100-rust-exercise-guide/blob/master/src/3_launch.md#open-an-input-output-interface-to-the-board) for how to see the output using PuTTY. The program needs to be re-run to get the output to show on the terminal.

## Directory structure and files

| Path            | Description                                                                                                    |
|-----------------|----------------------------------------------------------------------------------------------------------------|
| .cargo/config   | A file describing the location and options of the C-linker.                                                    |
| Cargo.toml      | A Rust manifest describing dependencies.                                                                       |
| Cargo.lock      | A generated file describing the last successful build configuration. Commit along with changes to dependencies |
| pynq/           | Extra files required to cross-compile on PYNQ-Z1 and program the FPGA.                                         |
| run_on_pynq.tcl | A tickle-script to initialize the FPGA and run the project on Pynq                                             |
| rust-toolchain  | A file to ask rustup to use a particular toolchain                                                             |
| src/            | The Rust source code                                                                                           |


## Troubleshooting
- `cargo build` fails with can't find crate for `core`.
    * The Rust `core` library cannot be compiled for the target. The likely cause is that the correct component for the target is not added via rustup.
    * You can add the component for the Cortex-A cross-compiler with ```rustup target add armv7a-none-eabi```.
- `cargo build` fails with "warning: couldn't execute `llvm-config --prefix`".
    * Likely cause: cargo cannot detect the LLVM toolchain, LLVM needs to be installed.
    * LLVM can be installed from https://releases.llvm.org/download.html.
- `cargo build` fails with "cannot detect Xilinx SDK at C:/Xilinx".
    * The libxil FFI dependency cannot locate the Xilinx toolchain automatically. Make sure that Xilinx SDK is installed and set its path using `export XILINX_SDK=/path/to/Xilinx/SDK/version`.
- `cargo build` fails with "error: linker `arm-none-eabi-gcc` not found".
    * The appropriate linker is not found.
    * arm-none-eabi-gcc can be installed on debian (like Ubuntu) using apt/dpkg using `apt-get install gcc-arm-none-eabi`.
    * If the build system cannot find the linker, use .cargo/config to point cargo to a functioning linker using the key "linker = /location/of/linker-executable". Xilinx ships with a functioning cross-linker, so if you can find the directory where the SDK is installed, the Windows path to the linker will be `"</location/of/Xilinx>/SDK/<version>/2017.2/gnu/aarch32/nt/gcc-arm-none-eabi/bin/arm-none-eabi-gcc"`. On linux, "nt" will be "lin" instead.
- `cargo build` fails with "error: linker `C:/.../arm-none-eabi-gcc` not found".
    * The GCC linker used for this work is not available at its pre-configured location at .cargo/config.
    * See above for a potential solution.
- `source run_on_pynq.tcl` returns "no targets found with ...".
    * Make sure the PYNQ-Z1 is turned on and connected.

