<!-- TODO asinghani: These are rough notes, clean these up slightly if we want
     to give this to students -->

# Running Vivado remotely

We may sometimes want to do our development on a laptop but run Vivado on a remote server. This is particularly applicable on macOS devices as Vivado is only available on x86 Windows & Linux.

## Running Vivado

When a project is generated with hardcaml_hobby_boards, a `run_vivado_remotely.sh` script is also provided, which logs into a remote server via SSH, runs the build, and copies back the results.

```
./run_vivado_on_server.sh yourusername@server
```

Authentication is handled by the underlying SSH client. The script will attempt to open a single SSH connection and use it for the entire process. If a password is required, you should only be prompted for it one time. 

## Flashing the board

Most Digilent and Xilinx boards (amongst others) can be flashed using
[openFPGALoader](https://github.com/trabucayre/openFPGALoader).

Builds of openFPGALoader are available in [several package managers](https://trabucayre.github.io/openFPGALoader/guide/install.html).

If openFPGALoader is not available in your package manager, YosysHQ provides [nightly builds of the oss-cad-suite for Windows, Linux, and macOS (including ARM devices)](https://github.com/YosysHQ/oss-cad-suite-build/releases/), which include openFPGALoader in the `bin` directory (macOS users may need to run the `activate` script inside the download before the binary will be runnable).

To flash the board:

```
openFPGALoader --board nexys_a7_100 --write-flash nexys_a7_100t.bit
```

<!-- TODO asinghani: verify whether the command is the same on Windows -->
