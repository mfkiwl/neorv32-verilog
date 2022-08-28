# NEORV32 _in_ Verilog

[![neorv32-verilog](https://img.shields.io/github/workflow/status/stnolting/neorv32-verilog/Verification/main?longCache=true&style=flat-square&label=neorv32-verilog%20check&logo=Github%20Actions&logoColor=fff)](https://github.com/stnolting/neorv32-verilog/actions/workflows/main.yml)
[![License](https://img.shields.io/github/license/stnolting/neorv32-verilog?longCache=true&style=flat-square&label=License)](https://github.com/stnolting/neorv32-verilog/blob/main/LICENSE)
[![Gitter](https://img.shields.io/badge/Chat-on%20gitter-4db797.svg?longCache=true&style=flat-square&logo=gitter&logoColor=e8ecef)](https://gitter.im/neorv32/community?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

1. [Prerequisites](#Prerequisites)
2. [Configuration](#Configuration)
3. [Conversion](#Conversion)
4. [Simulation](#Simulation)

This repository shows how to convert the [NEORV32 RISC-V Processor](https://github.com/stnolting/neorv32), which is
written in platform-independent **VHDL**, into a plain and synthesizable **Verilog netlist module** using
[GHDL's](https://github.com/ghdl/ghdl) synthesis feature. The resulting Verilog module can be instantiated into an
all-Verilog design and can be successfully simulated and synthesized - both tested with Xilinx Vivado.

:books: Detailed information regarding GHDL's synthesis feature can be found in the
[GHDL synthesis documentation](https://ghdl.github.io/ghdl/using/Synthesis.html).

:heavy_check_mark: The [verification workflow](https://github.com/stnolting/neorv32-verilog/actions/workflows/main.yml)
converts a pre-configured setup of latest processor version into a Verilog netlist and tests the result by running
an [Icarus Verilog](https://github.com/steveicarus/iverilog) simulation.


## Prerequisites

1. Clone this repository recursively to include the NEORV32 submodule.

2. Install GHDL. On a Linux machine GHDL can be installed easily via the package manager.
Make sure to install a version with `--synth` option enabled (should be enabled by default).

```
$ sudo apt-get install ghdl
```

3. Test the GHDL installation.

```
$ ghdl -v
GHDL 3.0.0-dev (v2.0.0-652-g6961b3f82) [Dunoon edition]
 Compiled with GNAT Version: 9.4.0
 mcode code generator
Written by Tristan Gingold.

Copyright (C) 2003 - 2022 Tristan Gingold.
GHDL is free software, covered by the GNU General Public License.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

[[back to top](#NEORV32-in-Verilog)]


## Configuration

GHDL `synth` option is used to convert the whole NEORV32 processor - including all peripherals, memories
and memory images - into a single Verilog netlist module file.

:warning: The output of the GHDL synthesis is a _post-elaboration_ result. Therefore, all the processor's configuration
options (i.e. VHDL generics) are resolved _before_ the actual output is generated (see the GHDL
[internals documentation](http://ghdl.github.io/ghdl/internals/index.html)).

To ease configuration and customization a minimal VHDL wrapper
[`src/neorv32_verilog_wrapper.vhd`](https://github.com/stnolting/neorv32-verilog/blob/main/src/neorv32_verilog_wrapper.vhd)
is provided. This wrapper can be used to configure the processor setup according to the requirements.
The default wrapper from this repository only implements a minimal subset of the available configuration options
and interfaces - just enough to run the processor's built-in bootloader.

Have a look at the original [processor top entity (`neorv32_top.vhd`)](https://github.com/stnolting/neorv32/blob/main/rtl/core/neorv32_top.vhd)
and just copy the generics and interfaces that you would like to use for the Verilog setup.
Note that all NEORV32 interface inputs and configuration generics do provide default values.

[[back to top](#NEORV32-in-Verilog)]


## Conversion

The actual conversion is conducted by a conversion shell script, which analyzes all the processor's sources and finally
calls GHDL `synth` to create the final Verilog netlist `neorv32-verilog/src/neorv32_verilog_wrapper.v`.

```
neorv32-verilog/src$ sh convert.sh
```

After generating the netlist the interface of the resulting `neorv32_verilog_wrapper` Verilog
module is shown in the console. This can be used as instantiation template.

```
------ neorv32_verilog_wrapper interface ------
module neorv32_verilog_wrapper
  (input  clk_i,
   input  rstn_i,
   input  uart0_rxd_i,
   output uart0_txd_o);
```

### Notes

* GHDL synthesis generates an unoptimized plain Verilog netlist without any (technology-specific) primitives.
However, optimizations will be performed by the synthesis tool (e.g. mapping to FPGA primitives like block RAMs).
* The interface of the resulting NEORV32 Verilog module lists all inputs first followed by all outputs.
* The original NEORV32 module hierarchy is preserved as well as most (all?) signal names, which allows easy inspection and debugging
of simulation waveforms and synthesis results.
* Custom VHDL interface types and records are collapsed into linear arrays. For example, the 8x32-bit interface type `sdata_8x32_t`
(NEORV32 SLINK interface) is packed into a one-dimensional 8*32=256-bit vector.

[[back to top](#NEORV32-in-Verilog)]


## Simulation

This repository provides a simple [Verilog testbench](https://github.com/stnolting/neorv32-verilog/blob/main/sim/testbench.v)
that can be used to simulate the default NEORV32 configuration. The testbench includes a UART receiver, which is driven by the
processor UART0. It outputs received characters to the simulator console.

A pre-configured simulation script based on [Icarus Verilog](https://github.com/steveicarus/iverilog) can be used to simulate
the Verilog setup (takes several minutes to complete):

```
neorv32-verilog/sim$ sh iverilog_sim.sh
neorv32-verilog testbench




<< NEORV32
Simulation successful!
./testbench.v:72: $finish called at 97188150 (100ps)
```

The simulation is terminated automatically as soon as the string "`NEORV32`" has been received from the processor's bootloader.
In this case `Simulation successful!` is printed to the console. If `Simulation terminated!` appears in the simulator console the simulation
has failed.

[![Check_iverilog](https://img.shields.io/github/workflow/status/stnolting/icarus-verilog-prebuilt/Check%20Icarus%20Verilog%20Packages/main?longCache=true&style=flat&label=Check%20iverilog%20packages&logo=Github%20Actions&logoColor=fff)](https://github.com/stnolting/icarus-verilog-prebuilt/actions/workflows/check_iverilog.yml)
\
Prebuilt Icarus Verilog binaries for Linux can be downloaded from
[stnolting/icarus-verilog-prebuilt](https://github.com/stnolting/icarus-verilog-prebuilt).

[[back to top](#NEORV32-in-Verilog)]
