# Working with the `hardcaml_hobby_boards` and the Nexys A7 board

## Generating the project RTL

To generate the Verilog and build scripts for the board, we can use the `generate_top` function in the Hardcaml hobby boards library:

```
Hardcaml_hobby_boards.Nexys_a7_100t.generate_top (Counter.board ())
```

We'll generally wrap this in a `project.exe` or `generate.exe` executable, i.e.

```
./counter.exe nexys-a7
```

This will generate 5 files:

- `nexys_a7_100t.v` - The generated Verilog source. This contains our Hardcaml project as
  well as surrounding infrastructure to set up the peripherals on the Nexys A7 board
- `nexys_a7_100t.xdc` - The constraints file, which sets up the input and output pins used
  for each peripheral (such as the LEDs and switches)
- `nexys_a7_100t.tcl` - The main build script. Running this with Vivado will run the full
  build process and generate a bitstream that can be loaded onto the board.
- `flash.tcl` - A script that can be run with Vivado to flash the board, if you have
  Vivado installed locally and the board is plugged in via USB.
- `run_vivado_remotely.sh` - A shell script for running Vivado on a remote server via SSH,
  and copying back the resultant files.

## Building the project

### Natively on Linux

If you're working directly on a Linux machine, you can run the following command to
directly invoke Vivado and run the build (depending on where Vivado is installed, you may
need to specify the full path to the `vivado` binary).

```
vivado -mode batch -source nexys_a7_100t.tcl
```

### On Windows using WSL

If you're on Windows and using WSL for OCaml, you'll likely have Vivado installed directly
on Windows, rather than inside the Linux install. In this case, you'll run the same
command in the Windows command line, rather than in the WSL shell.

```
vivado -mode batch -source nexys_a7_100t.tcl
```

### Running on a remote server

If you're using macOS or another machine that can't run Vivado locally, you can run it on
a remote Linux server using the generated script.

```
bash run_vivado_remotely.sh yourusername@server
```

Authentication is handled by the underlying SSH client. The script will attempt to open a
single SSH connection and use it for the entire process. If a password is required, you
should only be prompted for it one time. Note that Vivado can take up to 1-2 minutes extra
to launch on the remote server, and you may not see any output during this time.

### Build results format

By default (when `DEBUG = false`), two files will be emitted by the build process:

- `nexys_a7_100t.timing.rpt` - The timing report from the build; this contains information
  about the critical paths and any setup/hold violations in your design. For convenience,
  the build script also prints the Worst Negative Slack (WNS) upon completion; if this
  number is positive you don't have any setup-time violations
- `nexys_a7_100t.bit` - The bitstream file, this contains the compiled design, ready to be
  configured onto the FPGA.

## Flashing the board

### Using Vivado and the flash.tcl script

The simplest way to flash the board is using the generated Vivado script. Similarly to
building, you'll want to run this script locally (if on Linux) or in the Windows command
line (if using WSL), and ensure you're in the directory containing the `.bit` file.

```
vivado -mode batch -source flash.tcl
```

### Using the cross-platform tool openFPGALoader

If you don't have Vivado installed, you can generally use [openFPGALoader](https://github.com/trabucayre/openFPGALoader) to flash most Digilent and Xilinx boards.

Builds of openFPGALoader are available in [several package
managers](https://trabucayre.github.io/openFPGALoader/guide/install.html). If
openFPGALoader is not available in your package manager, YosysHQ provides [nightly builds
of the oss-cad-suite for Windows, Linux, and macOS (including ARM
devices)](https://github.com/YosysHQ/oss-cad-suite-build/releases/), which include
openFPGALoader in the `bin` directory (macOS users may need to run the `activate` script
inside the download before the binary will be runnable).

To flash the board (depending how you installed it, you may need to specify the full
path to the `openFPGALoader` binary):

```
openFPGALoader --board nexys_a7_100 --write-flash nexys_a7_100t.bit
```

