Simulation Directory for CV32E40P Core Testbench
================================================
This is the directory in which you should run all tests of the Core Testbench.
The testbench itself is located at `../../tb/core` and the test-programs are at
`../../tests`.  See the README in those directories for more information.

Supported SystemVerilog Simulators
----------------------------------
The core testbench supports two simulators:
- **Verilator** v5.042 or later — used for the standard `sanity`, `test`, and `certify` flows.
- **Questa** (Siemens Questa Altera FSE or full) — used for the `certify-vsim` cross-check flow,
  primarily to validate results against Verilator for FPU-intensive tests where Verilator's
  UNOPTFLAT handling of iterative combinatorial paths may produce incorrect results.

RISC-V GCC Compiler "Toolchain"
-------------------------------
Pointers to the recommended toolchain for CV32E40P are in `../TOOLCHAIN`.

Running the testbench with [verilator](https://www.veripool.org/wiki/verilator)
----------------------
Point your environment variable `RISCV` to your RISC-V toolchain. Call `make`
to run the default test (hello-world).

Running your own C or Assembler test-programs
---------------------
Manually written test-programs are located in the `custom` folder. The relevant sections
in the Makefile on how to compile and link this program can be found under `Running
custom programs`.  Make sure you have a working C compiler (see above) and keep in
mind that you are running on a very basic machine.
Try the following:<br>
```
make test TEST=dhrystone
make test TEST=misalign
make test TEST=fibonacci
make test TEST=illegal
make test TEST=riscv_ebreak_test_0
```

Running RISC-V Architectural Certification Tests (ACT4)
-------------------------------------------------------
CV32E40P-DV supports the RISC-V Architectural Certification Tests (ACT4). The ACT4 repository is
fetched on demand via a Make target — it is **not** a git submodule.

### ISA Configuration

CV32E40P supports two certification configurations controlled by `CV_CORE_CONFIG`:

| `CV_CORE_CONFIG` | Extensions | FPU | Config name |
|---|---|---|---|
| `rv32imc` (default) | I, M, C, Zca, Zicsr, Zifencei | off | `cv32e40p_v1.8.3_rv32imc` |
| `rv32imcf` | I, M, C, Zca, Zcf, F, Zicsr, Zifencei | on | `cv32e40p_v1.8.3_rv32imcf` |

### Certification Profile

The `CERT_PROFILE` variable selects which extension set is passed to ACT4 for test generation.
ACT4 then dynamically filters tests against the core's UDB YAML — only extensions actually
implemented in the DUT are tested.

```
make gen-certify [CERT_PROFILE=rvi20]   # default: rvi20
```

Currently supported profiles:

| `CERT_PROFILE` | Description |
|---|---|
| `rvi20` (default) | RVI20U32 unprivileged profile |

### Prerequisites
- **RISC-V GCC toolchain** (`riscv64-unknown-elf-gcc`): upstream GCC with RISC-V multi-lib support.
  See `../TOOLCHAIN.md`. Set `CV_SW_TOOLCHAIN` and `CV_SW_PREFIX` in your environment, e.g.:
  ```
  export CV_SW_TOOLCHAIN=/opt/riscv
  export CV_SW_PREFIX=riscv64-unknown-elf-
  ```
- **Sail RISC-V reference model v0.10** (`sail_riscv_sim` on `$PATH`): required version is 0.10.
  Install the pre-built binary:
  ```
  curl --location https://github.com/riscv/sail-riscv/releases/download/0.10/sail_riscv_sim-$(uname)-$(arch).tar.gz | sudo tar xvz --directory=/usr/local --strip-components=1
  ```
- **Python uv** package manager: https://docs.astral.sh/uv/getting-started/installation/
- **Verilator** v5.042 or later.

### Verilator Certification Flow

ACT4 is fetched automatically when you first run `make gen` or `make gen-certify`.
Run the full certification flow:
```
make gen-certify [CV_CORE_CONFIG=rv32imcf]
```
Or separately:
```
make gen       # clone ACT4 (if needed) + generate ELFs via Sail reference model
make certify   # compile with Verilator and run all ELFs through the DUT
```

To run only a specific extension:
```
make certify FILTER=Zicntr [CV_CORE_CONFIG=rv32imcf]
```

To generate waveforms (FST) for debugging:
```
make certify WAVES=1 FILTER=Zicntr
gtkwave cv32e40p.fst
```

Results are written to:
`simulation_results/certification_<CV_CORE_CONFIG>_<CERT_PROFILE>/logs/certification_summary.txt`

For example, for the default rv32imc + rvi20:
`simulation_results/certification_rv32imc_rvi20/logs/certification_summary.txt`

Running ACT4 Certification Tests with Questa (vsim)
----------------------------------------------------
Questa provides an independent cross-check for Verilator results. This is particularly
useful for FPU operations (fdiv, fsqrt) where Verilator suppresses UNOPTFLAT warnings
and may mismodel iterative combinatorial feedback paths, producing false failures.

### Questa Prerequisites
- **Questa** (Altera FSE or full edition): `vsim`, `vlog`, `vopt` must be on `$PATH`,
  or override `VSIM`, `VLOG`, `VOPT` variables in the make invocation.
  Default paths point to `/home/marin/altera/25.1std/questa_fse/`.

### Questa Certification Flow

Compile the RTL and testbench once for a given configuration:
```
make questa-compile [CV_CORE_CONFIG=rv32imcf]
```

Then run all ACT4 ELFs through the Questa DUT:
```
make certify-vsim [CV_CORE_CONFIG=rv32imcf]
```

Or generate tests, compile, and run in one step:
```
make gen-certify-vsim [CV_CORE_CONFIG=rv32imcf]
```

To run only a specific extension:
```
make certify-vsim FILTER=Zicntr [CV_CORE_CONFIG=rv32imcf]
```

Results are written to:
`simulation_results/questa_<CV_CORE_CONFIG>_<CERT_PROFILE>/logs/certification_summary.txt`

For example, for rv32imcf + rvi20:
`simulation_results/questa_rv32imcf_rvi20/logs/certification_summary.txt`

The maximum cycle limit per test defaults to 2,000,000 and can be overridden:
```
make certify-vsim CV_CORE_CONFIG=rv32imcf VSIM_MAX_CYCLES=5000000
```

### Known Verilator vs Questa Differences

| Test | Verilator | Questa | Root cause |
|---|---|---|---|
| F-fdiv.s-00 | FAIL | PASS | FPNEw iterative divide path mismodelled by Verilator |
| F-fdiv.s-01 | FAIL | PASS | FPNEw iterative divide path mismodelled by Verilator |
| F-fsqrt.s-00 | FAIL | PASS | FPNEw iterative sqrt path mismodelled by Verilator |
| Zicntr-csrrc-00 | PASS (false) | PASS | See note below |
| Zicntr-csrrs-00 | PASS (false) | PASS | See note below |

**FPU failures (F-fdiv, F-fsqrt):** Verilator mismodels the FPNEw FPU iterative datapath
due to combinatorial feedback loops in the divide/sqrt units. These are Verilator simulation
artefacts, not RTL bugs. Questa is authoritative for FPU tests.

**Zicntr false positive:** CV32E40P resets `mcountinhibit` to `0x1` (counters inhibited) but
Verilator incorrectly simulates it as `0` (counters enabled) due to a Verilator code generation
bug with packed arrays driven by mixed `assign`/`always_ff` in generate for-loops. This causes
Zicntr tests to falsely pass in Verilator when `RVMODEL_BOOT` does not clear `mcountinhibit`.
The CTP fix (`RVMODEL_BOOT` clears `mcountinhibit`) makes both simulators agree. See
`config/cores/cv32e40p/*/rvmodel_macros.h`.
