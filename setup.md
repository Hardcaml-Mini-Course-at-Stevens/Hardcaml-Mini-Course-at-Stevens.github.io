# Setting up the environment

## OS support & Prerequisites

The OCaml environment will we install only works on a Linux or MacOS based OS.  For Windows we recommend installing the WSL2 Linux VM environment (other VMs will also work, but WSL2 is what we have tested).

One quirk of WSL2 based development is that VSCode will generally run as a native Windows application and will SSH into the WSL2 VM. For this to work we must install the Microsoft `Remote Development` extension. Follow the [VSCode WSL instructions](https://code.visualstudio.com/docs/remote/wsl) to set this up, so that you can properly interface with your WSL2 environment from VSCode. Note that actually programming the physical FPGA board will still be done from within Windows.

In all cases there are some basic OS packages that will need to be installed: git, gcc, curl, wget, m4, pkg-congfig (among others).

For Linux (and WSL2) use the OS(/VM) package manager to install them.  For MacOS, you will need to install a 3rd party package manager such as `homebrew`. Opam (the OCaml package manager) will generally tell you if a package it needs is missing.

## Installing Opam and the OCaml compiler

1. Install [opam](https://opam.ocaml.org/doc/Install.html) from the official binary distribution.

    ```
    bash -c "sh <(curl -fsSL https://opam.ocaml.org/install.sh)"
    ```

2. Ensure packages are up to date

    ```
    opam update --all
    ```

3. Add the two repositories to your opam. These are our forks of the opam repositories (both the oxcaml one and the default one), pinned at the package versions used in the course examples.

    ```
    opam repository add --all oxcaml-for-hardcaml-course git+https://github.com/Hardcaml-Mini-Course-at-Stevens/oxcaml-opam-repository-mirror#summer2025

    opam repository add --all upstream-for-hardcaml-course git+https://github.com/Hardcaml-Mini-Course-at-Stevens/opam-repository-mirror#summer2025
    ```

4. Create an opam switch, backed by these repositories (A "switch" is an isolated environment which contains a specific set of ocaml packages, at specific versions. We create a dedicated switch in which we will install the versions of packages used in this course). This may take a while to build all of the packages.

    ```
    opam switch create hardcaml-course 5.2.0+ox --repos oxcaml-for-hardcaml-course,upstream-for-hardcaml-course
    ```

5. Ensure you have the newly-created switch selected

    ```
    opam switch hardcaml-course
    ```

## Installing some basic packages

1. Install the following packages from opam:

    ```
    opam install -y ocamlformat merlin ocaml-lsp-server dune utop core
    ```

    - `ocamlformat` - An autoformatter for improving the readability of OCaml code
    - `merlin` + `ocaml-lsp-server` - Language server tools to provide autocompletion, type checking, and other reference capabilities in your editor
    - `dune` - The build system we use to organize dependencies and build our projects
    - `utop` - An interactive OCaml interpreter, useful for experimentation
    - `core` - A replacement standard library for OCaml which provides better safety and consistency than the default, and is also used by Hardcaml

2. Test your installation with `utop`. The following steps will import (`#require`) some libraries, then open `Core` (allowing it to shadow the standard library), then do some math and print the result. Note that the `#require` part is only required in `utop`, not in saved `.ml` files.

    ```
    ─( 20:33:27 )
    utop # #require "core";;

    ─( 20:33:27 )
    utop # #require "ppx_jane";;

    ─( 20:33:30 )
    utop # open Core;;

    ─( 20:33:31 )
    utop # let x = 2 + 3 ;;
    val x : int = 5

    ─( 20:33:36 )
    utop # print_s [%message "hello world" (x : int)];;
    ("hello world" (x 5))
    - : unit = ()
    ```

## Editor setup

The recommended editor for this course is [VSCode](https://code.visualstudio.com/), which has very powerful language-server integration and provides handy utilities including inline errors, type lookups, and jump-to-definition.

Instructions for other editors can be found in [Real World OCaml](https://dev.realworldocaml.org/install.html), however we will not be able to provide debugging support for other editors in this course.

To install and test the OCaml VSCode extension:

1. Open VSCode and go to the Extensions menu (View -> Extensions)

2. Search for and install the "OCaml Platform" extension by OCaml Labs. Additionally install the "ocamlformat" extension by Shuumatsu.

3. Open up the VSCode settings menu and search for "Format On Save", and make sure it is set to true.

4. Clone our test project from GitHub:

    ```
    git clone https://github.com/Hardcaml-Mini-Course-at-Stevens/ocaml-test-project
    ```

5. Open the cloned project in VSCode (File -> Open Folder)

6. Open up `src/test.ml`. It should be a very simple OCaml file that defines one expect-test and specifies the expected output.

7. Now we're going to try to build this project. We always run the OCaml build system (dune) through the command-line, even if we are using VSCode. The language server extension will automatically detect the running build system and interface with it as needed. Open up a terminal (preferably inside of VSCode, using the "Terminal" -> "New Terminal" button), and make sure you're in the project directory (this should be the same directory that contains a `dune-project` file.

8. Run the command 

    ```
    dune build --watch --terminal-persistence=clear-on-rebuild-and-flush-history @src/runtest
    ```

    This should take a few seconds and then print "Success, waiting for filesystem changes...". Breaking down this command:
    - `dune build`: the command for dune to build an OCaml project
    - `--watch`: tells dune to rebuild every time you edit a source file, this gives you continous real-time feedback
    - `--terminal-persistence=clear-on-rebuild-and-flush-history`: tells dune to clear the terminal every time it rebuilds, this makes it clear which errors and warnings are currently active
    - `@src/runtest`: this is the target we're building. `@` tells dune that this is an alias and not a file, and `runtest` is a built-in alias for running all of the inline tests in a directory.
     
9. Try hovering over some of the identifiers in the source, such as `sum`. If the OCaml language server is installed correctly, it should show "int" as the type. Hovering over `print_endline` should also show both its type (`string -> unit`) and a docstring.

10. Try changing one of the values defined in the file (change `a` from 1 to 2), but without changing the contents of the `[%expect]` block. Dune should briefly attempt to rebuild and rerun the tests, and then present you with a diff showing the changed test output. If dune still says "Success", make sure the `runtest` build target was correctly specified in the command run earlier. Note how the diff indicates the new behavior of the code, and how the printed output has changed.

11. To accept the diff, open up a new terminal ("Terminal" -> "Split Terminal") and make sure you're in the project directory again. Then run `dune promote` and watch as the updated `[%expect]` block is added to your file (it should update automatically, but you may need to close and reopen the file if it doesn't). Now that the file has been updated, dune will try to run the tests again, and this time it should succeed. If you want to learn about some other cool ways to use expect-tests, [check out this blog post.](https://blog.janestreet.com/the-joy-of-expect-tests/)

12. Finally, we'll test the autoformatter. Try making some changes to the formatting of the file (i.e. add extra blank lines to the file, or remove the indentation of a random line), and then saving it. If Format on Save is enabled, then `ocamlformat` should automatically run and reformat your file to the canonical style. The autoformatter isn't required, but it can help substantially with preventing certain kinds of subtle bugs, as well as consistently keeping your code in a readable state.

## Hardcaml installation

To use Hardcaml, you'll need to install a few more packages:

1. Install the essential Hardcaml packages from opam

    ```
    opam install -y hardcaml ppx_hardcaml hardcaml_waveterm hardcaml_of_verilog simple_xml
    ```

2. In addition, we'll install the `hardcaml_hobby_boards` library. We'll install this library directly from the GitHub source, rather than from the opam package repository, in order to get the most up-to-date version.

    ```
    opam pin -y hardcaml_hobby_boards https://github.com/Hardcaml-Mini-Course-at-Stevens/hardcaml_hobby_boards.git#main
    ```

3. Finally, pop open `utop` one more time to make sure the library is installed correctly, by trying to look up the type of the `generate_top` function:

    ```
    ─( 20:44:59 )
    utop # #require "hardcaml_hobby_boards";;
    ─( 20:44:59 )
    utop # Hardcaml_hobby_boards.Nexys_a7_100t.generate_top;;
    - : Hardcaml_hobby_boards.Board.t -> unit = <fun>
    ```

## Vivado Installation

The tool we'll use to compile our designs to the FPGA is called Xilinx Vivado. If you're on Windows or Linux, follow the below steps to install it on your local machine (if using WSL2, you may find it easier to install Vivado directly in Windows and invoke it via the Windows command line). Note that Vivado requires ~90GB of free disk space to set up and will take a few hours to download.

If you're on MacOS, you won't be able to install Vivado locally. Please reach out to get access to the shared course server on which you will be able to run your Vivado builds.

1. Download the "AMD Unified Installer for FPGAs & Adaptive SoCs 2025.1" from the [Vivado install page](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2025-1.html) (make sure to get the 300MB "Self Extracting Web Installer" version, not the 100GB full package. You may need to register an account with AMD here.

2. Run the installer. When prompted, select "Vivado" as the app you want to install (*not* Vitis, which is the default).

3. When asked for which variant, select "Vivado ML Standard Edition"

4. On the page which prompts which features to install, make sure "Vivado Design Suite", "DocNav", and "Artix-7" (under Devices -> Production Devices -> 7 Series) are selected. Everything else can be unselected to save disk space.

5. Allow the Vivado install to finish. This may take a few hours to finish downloading.

6. Verify that you can launch Vivado by running the `vivado` command in your shell or command prompt. Depending on your exact setup, you may need to find and specify the exact path where the Vivado binary was installed.

## Wrap-up

If you've reached this point, you've successfully installed and set up OCaml, Hardcaml, and your development environment and are ready to start working with Hardcaml! If you'd like to refresh yourself on OCaml and how to use the `Core` standard library, check out [Real World OCaml](https://dev.realworldocaml.org/)
