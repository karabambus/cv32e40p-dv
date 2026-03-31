<!--
Copyright 2022 OpenHW Group
SPDX-License-Identifier: Apache-2.0 WITH SHL-2.1
-->
# CV32E40P Verification Environment (cv32e40p-dv)

Verification environment for the [CV32E40P](https://github.com/openhwgroup/cv32e40p) RISC-V core,
adapted from OpenHW's cv32e20-dv for running
[RISC-V Architectural Certification Tests (ACT4)](https://github.com/riscv/riscv-arch-test).

## Core Configurations

Select a configuration with `CV_CORE_CONFIG=`:

| Config | RTL | ISA | FPU |
|---|---|---|---|
| `rv32imc` (default) | v1.8.3 | RV32IMC | off |
| `rv32imcf` | v1.8.3 | RV32IMCF | on |
| `v1.0.0_rv32imc` | v1.0.0 | RV32IMC | off |

## Quick Start

```bash
cd sim/core

# Verilator — build and run sanity test
make sanity

# Generate ACT4 tests, build and run all certification ELFs
make gen-certify                                    # default rv32imc
make gen-certify CV_CORE_CONFIG=rv32imcf            # with FPU
make gen-certify CV_CORE_CONFIG=v1.0.0_rv32imc      # legacy RTL

# Questa — compile and run all certification ELFs
make gen-certify-vsim
make certify-vsim FILTER=I                          # run only I-extension ELFs

# Wave dumping (FST format)
make gen-certify WAVES=1
```

## Prerequisites

- [Verilator](https://verilator.org/guide/latest/install.html) (primary simulator)
- RISC-V GCC toolchain at `$CV_SW_TOOLCHAIN` (default: `/opt/riscv`)
- [uv](https://docs.astral.sh/uv/getting-started/installation/) (for ACT4 test generation)
- Questa (optional, for cross-checking Verilator results)

## Directories

- **bsp**: board support package for test-programs
- **sim**: simulation entry point (`sim/core/Makefile`)
- **tb**: testbench sources (tb_top, mm_ram, stall/IRQ generators)
- **tests**: test-programs (custom C/asm, compliance)
- **mk**: shared Makefiles
- **env**: UVM environment (from upstream, not used by core testbench)
- **vendor_lib**: third-party libraries (core-v-verif, ACT4)
