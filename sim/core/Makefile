###############################################################################
#
# Copyright 2025 Eclipse Foundation
# Copyright 2020 OpenHW Foundation
#
# Licensed under the Solderpad Hardware Licence, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     https://solderpad.org/licenses/
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
###############################################################################
#
# Makefile for the CV_CORE "core" testbench.  Substantially modified from the
# Makefile original for the RI5CY testbench.
#
###############################################################################
#
# Copyright 2019 Claire Wolf
# Copyright 2019 Robert Balas
#
# Permission to use, copy, modify, and/or distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH
# REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF MERCHANTABILITY
# AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR ANY SPECIAL, DIRECT,
# INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES WHATSOEVER RESULTING FROM
# LOSS OF USE, DATA OR PROFITS, WHETHER IN AN ACTION OF CONTRACT, NEGLIGENCE OR
# OTHER TORTIOUS ACTION, ARISING OUT OF OR IN CONNECTION WITH THE USE OR
# PERFORMANCE OF THIS SOFTWARE.
#
# Original Author: Robert Balas (balasr@iis.ee.ethz.ch)
#
###############################################################################

# "Constants"
MAKE           = make
MAKE_DIR       = $(PWD)
MAKE_PATH     := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))
CV32E40P_DV    = $(abspath $(MAKE_PATH)/../../)
CORE_V_VERIF   = $(CV32E40P_DV)/vendor_lib/openhwgroup_core-v-verif
DATE           = $(shell date +%F)
WAVE          ?= 0
WAVES         ?= 0
WAVE_FILE     ?= cv32e40p.fst

CV_CORE       ?= cv32e40p
CV_CORE_LC     = $(shell echo $(CV_CORE) | tr A-Z a-z)
CV_CORE_UC     = $(shell echo $(CV_CORE) | tr a-z A-Z)

#TODO(Ateeb): Temporary, remove next 4 lines after debug
CV_SW_TOOLCHAIN      ?= /opt/riscv
CV_SW_PREFIX         ?= "riscv64-unknown-elf-"
CV_SW_MARCH			 ?= rv32imc_zicsr_zifencei
CV_SW_CC		 	 ?= gcc

SIMULATOR     ?= $(CV_SIMULATOR)
ifneq (${SIMULATOR}, verilator)
  # TODO: should this testbench support other simulators?
  $(warning setting SIMULATOR to verilator (for now))
  SIMULATOR = verilator
endif

# Test-Program directores.
# Relative path is used for Verilator which cannot seem to handle loooong pathnames.
# TODO: find out if that is still true and if not, switch to abs paths.
TEST_PROGRAM_PATH    = $(CV32E40P_DV)/tests/programs/custom
TEST_PROGRAM_RELPATH = ../../tests/programs/custom
# 'Special' test-programs (see $(CV32E40P_DV)/mk/Common.mk)
TEST_FIXED_ELF ?=
TEST_ACT       ?=
ACT_MACROS      = $(CV32E40P_DV)/bsp/act4

# Common output directories
RUN_INDEX               ?= 0
SIM_RESULTS              = simulation_results
SIM_TEST_RESULTS         = $(SIM_RESULTS)/$(TEST)
SIM_RUN_RESULTS          = $(SIM_TEST_RESULTS)/$(RUN_INDEX)
SIM_TEST_PROGRAM_RESULTS = $(SIM_RUN_RESULTS)/test_program
SIM_BSP_RESULTS          = $(SIM_TEST_PROGRAM_RESULTS)/bsp

# Compile compile flags for all simulators
SV_CMP_FLAGS =

# Default "custom test-program"
TEST         ?= hello-world
CFG          ?= default

###############################################################################
# Common Makefiles:
#  -Variables for RTL and other dependencies (e.g. RISCV-DV)
include ../ExternalRepos.mk
#  -Core Firmware and the RISCV GCC Toolchain (SDK)
include $(CV32E40P_DV)/mk/Common.mk

###############################################################################
# verilator configuration
VERILATOR           = verilator
VERI_FLAGS         +=
VERI_TRACE         ?=
VERI_OBJ_DIR       ?= cobj_dir
#VERI_LOG_DIR       ?= cobj_dir/logs
VERI_LOG_DIR       ?= $(SIM_TEST_PROGRAM_RESULTS)

# FuseSoC configuration
FUSESOC             = fusesoc
FUSESOC_CORE        = openhwgroup:cv32e40p-dv:cv32e40p_sim:0.1
FUSESOC_TARGET      = $(if $(filter rv32imcf,$(CV_CORE_CONFIG)),sim_fpu,sim)
FUSESOC_WAVES_SUFFIX = $(if $(filter 1,$(WAVES)),_waves,)
FUSESOC_FINAL_TARGET = $(FUSESOC_TARGET)$(FUSESOC_WAVES_SUFFIX)
# Each target gets its own build root to prevent FPU_EN=0/1 collisions
# (Verilator incremental build would silently reuse the wrong binary otherwise)
FUSESOC_BUILD_DIR   = $(CV32E40P_DV)/build/$(FUSESOC_FINAL_TARGET)
# FuseSoC output path: <build_root>/<vlnv>/<target>-verilator/Vtb_top
VERI_EXEC_FUSESOC   = $(FUSESOC_BUILD_DIR)/openhwgroup_cv32e40p-dv_cv32e40p_sim_0.1/$(FUSESOC_FINAL_TARGET)-verilator/Vtb_top

# TB source files for the CV32E40P core
#TBSRC_HOME  := $(CORE_V_VERIF)/$(CV_CORE_LC)/tb
TBSRC_HOME  := $(CV32E40P_DV)/tb
TBSRC_TOP   := $(TBSRC_HOME)/core/tb_top.sv
TBSRC_CORE  := $(TBSRC_HOME)/core
TBSRC_PKG   := $(TBSRC_CORE)/tb_riscv/include/perturbation_defines.sv
TBSRC       := $(TBSRC_CORE)/tb_top.sv \
               $(TBSRC_CORE)/mm_ram.sv \
               $(TBSRC_CORE)/dp_ram.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_random_stall.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_random_interrupt_generator.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_rvalid_stall.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_gnt_stall.sv

#/tb_riscv/riscv_rvalid_stall.sv

RTLSRC_VLOG_TB_TOP	:= $(basename $(notdir $(TBSRC_TOP)))
RTLSRC_VOPT_TB_TOP	:= $(addsuffix _vopt, $(RTLSRC_VLOG_TB_TOP))

#TBSRC_VERI  := $(TBSRC_CORE)/tb_top_verilator.sv
TBSRC_VERI  := $(TBSRC_CORE)/tb_top.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_rvalid_stall.sv \
               $(TBSRC_CORE)/tb_riscv/riscv_gnt_stall.sv \
               $(TBSRC_CORE)/mm_ram.sv \
               $(TBSRC_CORE)/dp_ram.sv
SIM_LIBS    := $(CORE_V_VERIF)/lib/sim_libs

# RTL source files for the CV32E40P core
# DESIGN_RTL_DIR is used by CV_CORE_MANIFEST file
CV_CORE_PKG           := $(CV32E40P_DV)/core-v-cores/$(CV_CORE_LC)
ifeq ($(CV_CORE_CONFIG),rv32imcf)
  CV_CORE_MANIFEST    := $(CV_CORE_PKG)/cv32e40p_fpu_manifest.flist
else
  CV_CORE_MANIFEST    := $(CV_CORE_PKG)/cv32e40p_manifest.flist
endif
export DESIGN_RTL_DIR  = $(CV_CORE_PKG)/rtl

check:
	@echo "$(BANNER)"
	@echo "* Generated variables:"
	@echo "$(BANNER)"
	@echo "CV32E40P_DV      = $(CV32E40P_DV)"
	@echo "CORE_V_VERIF    = $(CORE_V_VERIF)"
	@echo "TEST_FLAGS_MAKE = $(TEST_FLAGS_MAKE)"
	@echo "TEST_FLAGS created by $(YAML2MAKE):"
	@cat $(TEST_FLAGS_MAKE)
	@echo "CFG_FLAGS_MAKE = $(CFG_FLAGS_MAKE)"
	@echo "CFG_FLAGS created by $(CFGYAML2MAKE):"
	@cat $(CFG_FLAGS_MAKE)
	@echo "CLONE_CV_CORE_CMD = $(CLONE_CV_CORE_CMD)"

# Shorthand rules for convience
CV_CORE_pkg: clone_rtl

tbsrc_pkg: $(TBSRC_PKG)

tbsrc: $(TBSRC)

###############################################################################
# The sanity rule runs whatever is currently deemed to be the minimal test that
# must be able to run (and pass!) prior to generating a pull-request.
.DEFAULT_GOAL := sanity

.PHONY: sanity
sanity:
	make test TEST=hello-world

all: clean_all sanity

###############################################################################
# Verilator

# First test if the user wants to to wave dumping. Nobody ever remembers if it's
# "WAVE" or "WAVES", so will work. We don't support VCD since the files are huge.
ifneq (${WAVE}, 0)
  WAVES = 1
endif
ifneq (${WAVES}, 0)
  VERI_TRACE      = --trace-fst
  VERI_RUN_FLAGS  = +waves
  VERI_WAVE_CMD   = +wave_file=$(SIM_RUN_RESULTS)/$(WAVE_FILE)
endif

verilate: CV_CORE_pkg
	@echo "$(BANNER)"
	@echo "* Compiling CORE TB and CV32E40P with Verilator via FuseSoC (target=$(FUSESOC_FINAL_TARGET))"
	@echo "$(BANNER)"
	$(FUSESOC) --cores-root=$(CV32E40P_DV)/sim/core \
	           run --no-export --target=$(FUSESOC_FINAL_TARGET) \
	           --build-root=$(FUSESOC_BUILD_DIR) \
	           --setup --build $(FUSESOC_CORE)
	mkdir -p $(SIM_RESULTS)
	mkdir -p $(SIM_TEST_RESULTS)
	cp $(VERI_EXEC_FUSESOC) $(SIM_TEST_RESULTS)/verilator_executable

test: verilate $(TEST_PROGRAM_PATH)/$(TEST)/$(TEST).hex
	@echo "$(BANNER)"
	@echo "* Running with Verilator: logfile in $(SIM_TEST_RESULTS)/$(TEST).log"
	@echo "$(BANNER)"
	mkdir -p $(VERI_LOG_DIR)
	$(SIM_TEST_RESULTS)/verilator_executable \
		$(VERI_RUN_FLAGS) \
		$(VERI_WAVE_CMD) \
		+verbose \
		+test_program=$(TEST_PROGRAM_RELPATH)/$(TEST)/$(TEST).hex \
		| tee $(VERI_LOG_DIR)/$(TEST).log

# verilator specific cleanup
verilator-clean:
	if [ -d $(SIM_RESULTS) ]; then rm -r $(SIM_RESULTS); fi
	if [ -d $(VERI_OBJ_DIR) ]; then rm -r $(VERI_OBJ_DIR); fi
	if [ -d $(FUSESOC_BUILD_DIR) ]; then rm -r $(FUSESOC_BUILD_DIR); fi
	if [ -e memory_dump.bin ]; then rm memory_dump.bin; fi
	if [ -e cv32e40p_tb.vcd ]; then rm cv32e40p_tb.vcd; fi


###############################################################################
# CV_CORE RTL dependencies

clone_rtl:
	@echo "$(BANNER)"
	@echo "* Cloning CV32E40P RTL model"
	@echo "$(BANNER)"
	$(CLONE_CV_CORE_CMD)

###############################################################################
# clean up simulation results
clean-sim-results:
	rm -rf $(SIM_RESULTS)

# clean up toolchain generated files
clean-test-programs:
	find $(CV32E40P_DV)/tests/programs -name "*.o"       -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "*.hex"     -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "*.elf"     -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "*.map"     -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "*.readelf" -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "*.objdump" -exec rm {} \;
	find $(CV32E40P_DV)/tests/programs -name "corev_*.S" -exec rm {} \;

clean-all-targets: clean-dist clean-cloned clean-sim-results clean-test-programs verilator-clean

clean-dist:
	rm -rf riscv-fesvr riscv-isa-sim

clean-cloned:
	rm -rf $(CV_CORE_PKG) $(CORE_V_VERIF)

clean-all-banner:
	@echo "$(BANNER)"
	@echo "* Delete all generated files (yes, we mean ALL)"
	@echo "$(BANNER)"

clean_all: clean-all-banner clean-all-targets
#endend

###############################################################################
# ISA configuration: rv32imc (FPU=0) or rv32imcf (FPU=1)
CV_CORE_CONFIG ?= rv32imc

ifeq ($(CV_CORE_CONFIG),rv32imcf)
  CERT_FPU_EN      = 1
  CERT_EXTENSIONS  = I,M,C,Zca,Zcf,F,Zicsr,Zifencei
  CERT_CONFIG_NAME = cv32e40p_v1.8.3_rv32imcf
else
  CERT_FPU_EN      = 0
  CERT_EXTENSIONS  = I,M,C,Zca,Zicsr,Zifencei
  CERT_CONFIG_NAME = cv32e40p_v1.8.3_rv32imc
endif

###############################################################################
# certification test-program directories
certification_PROGRAM_PATH    = $(ACT4_PKG)/work/$(CERT_CONFIG_NAME)/elfs
certification_PROGRAM_RELPATH = ../../vendor_lib/riscv-arch-test/act4/work/$(CERT_CONFIG_NAME)/elfs

###############################################################################
# certification test target (run ALL ELFs under compliance/rv32, quiet + summary)
.PHONY: gen
gen: $(ACT4_PKG)
	@echo "* Cleaning ACT4.0 directory..."
	@$(MAKE) -C $(ACT4_PKG) clean
	@echo "* Generating ACT4.0 tests for $(CERT_CONFIG_NAME)..."
	@$(MAKE) -C $(ACT4_PKG) \
		EXTENSIONS=$(CERT_EXTENSIONS) \
		CONFIG_FILES=config/cores/cv32e40p/$(CERT_CONFIG_NAME)/test_config.yaml
	@echo "* Tests generated in: $(certification_PROGRAM_PATH)"

.PHONY: certify
certify: TEST = certification_$(CV_CORE_CONFIG)
certify: verilate
	@echo "$(BANNER)"
	@echo "* Running ALL compliance ELFs with Verilator"
	@echo "$(BANNER)"
	@set -e; \
	mkdir -p "$(VERI_LOG_DIR)"; \
	SUMMARY="$(VERI_LOG_DIR)/certification_summary.txt"; \
	echo "RESULT  TEST" > "$$SUMMARY"; \
	echo "------  ----" >> "$$SUMMARY"; \
	ROOT="$(certification_PROGRAM_PATH)/rv32i"; \
	ELFS="$$(find -L "$$ROOT" \( -type f -o -type l \) -a \( -iname '*.elf' -o -iname '*.elf32' \) 2>/dev/null | sort)"; \
	if [ -z "$$ELFS" ]; then \
	  echo "No .elf files found under $$ROOT"; \
	  echo "Debug listing (2 levels):"; \
	  find "$$ROOT" -maxdepth 2 -print || true; \
	  exit 2; \
	fi; \
	PASS=0; FAIL=0; TOTAL=0; \
	for ELF in $$ELFS; do \
	  TOTAL=$$((TOTAL+1)); \
	  REL="$${ELF#$$ROOT/}"; \
	  BASE="$${ELF%.*}"; \
	  HEX="$$BASE.hex"; \
	  LOG="$(VERI_LOG_DIR)/$${REL%.*}.log"; \
	  mkdir -p "$$(dirname "$$LOG")"; \
	  if [ ! -f "$$HEX" ] || [ "$$ELF" -nt "$$HEX" ]; then \
	    riscv64-unknown-elf-objcopy -O verilog "$$ELF" "$$HEX"; \
	  fi; \
	  ENTRY=$$(riscv64-unknown-elf-readelf -h "$$ELF" | awk '/Entry point/{print $$NF}'); \
	  set +e; \
	  "$(SIM_TEST_RESULTS)/verilator_executable" \
	    $(VERI_RUN_FLAGS) \
	    $(VERI_WAVE_CMD) \
	    "+test_program=$$HEX" \
	    "+boot_addr=$$ENTRY" \
	    > "$$LOG" 2>&1; \
	  RC=$$?; \
	  set -e; \
	  if [ $$RC -eq 0 ] && grep -q "ALL TESTS PASSED" "$$LOG"; then \
	    echo "PASS    $$REL" | tee -a "$$SUMMARY"; \
	    PASS=$$((PASS+1)); \
	  else \
	    echo "FAIL    $$REL" | tee -a "$$SUMMARY"; \
	    FAIL=$$((FAIL+1)); \
	  fi; \
	done; \
	echo "" | tee -a "$$SUMMARY"; \
	echo "TOTAL=$$TOTAL PASS=$$PASS FAIL=$$FAIL" | tee -a "$$SUMMARY"; \
	echo "Summary saved to: $$SUMMARY"; \
	true

.PHONY: gen-certify
gen-certify: gen certify

###############################################################################
# Questa (vsim) certification flow
# Useful to cross-check Verilator results, e.g. to rule out UNOPTFLAT-related
# mismodelling for iterative FPU operations (fdiv, fsqrt).
#
# Usage:
#   make questa-compile [CV_CORE_CONFIG=rv32imcf]   # compile once
#   make certify-vsim   [CV_CORE_CONFIG=rv32imcf]   # run all ELFs
#   make gen-certify-vsim [CV_CORE_CONFIG=rv32imcf] # gen + compile + run

VSIM        ?= /home/marin/altera/25.1std/questa_fse/linux_x86_64/vsim
VLOG        ?= /home/marin/altera/25.1std/questa_fse/linux_x86_64/vlog
VOPT        ?= /home/marin/altera/25.1std/questa_fse/linux_x86_64/vopt

VSIM_WORK_DIR    = $(SIM_RESULTS)/questa_$(CV_CORE_CONFIG)
VSIM_LOG_DIR     = $(VSIM_WORK_DIR)/logs
VSIM_OPT_TOP     = tb_top_opt

VLOG_FLAGS  = -sv -work $(VSIM_WORK_DIR) \
              +define+FPU_EN=$(CERT_FPU_EN) \
              -suppress 2583
VOPT_FLAGS  = -work $(VSIM_WORK_DIR) \
              +acc -o $(VSIM_OPT_TOP)
VSIM_MAX_CYCLES ?= 2000000
VSIM_FLAGS  = -work $(VSIM_WORK_DIR) \
              -batch -quiet \
              -sv_seed 0 \
              +nowarnTSCALE \
              -suppress 8386 \
              -suppress 3009 \
              -do "run -all; quit -f" \
              +maxcycles=$(VSIM_MAX_CYCLES)

.PHONY: questa-compile
questa-compile: CV_CORE_pkg
	@echo "$(BANNER)"
	@echo "* Compiling CORE TB and CV32E40P with Questa (CV_CORE_CONFIG=$(CV_CORE_CONFIG))"
	@echo "$(BANNER)"
	mkdir -p $(VSIM_WORK_DIR)
	$(VLOG) $(VLOG_FLAGS) -f $(CV_CORE_MANIFEST)
	$(VLOG) $(VLOG_FLAGS) $(TBSRC_PKG)
	$(VLOG) $(VLOG_FLAGS) $(TBSRC_VERI)
	$(VOPT) $(VOPT_FLAGS) \
		-G FPU_EN=$(CERT_FPU_EN) \
		tb_top

.PHONY: certify-vsim
certify-vsim: TEST = certification_$(CV_CORE_CONFIG)
certify-vsim:
	@echo "$(BANNER)"
	@echo "* Running ALL compliance ELFs with Questa (CV_CORE_CONFIG=$(CV_CORE_CONFIG))"
	@echo "$(BANNER)"
	@set -e; \
	mkdir -p "$(VSIM_LOG_DIR)"; \
	SUMMARY="$(VSIM_LOG_DIR)/certification_summary.txt"; \
	echo "RESULT  TEST" > "$$SUMMARY"; \
	echo "------  ----" >> "$$SUMMARY"; \
	ROOT="$(certification_PROGRAM_PATH)/rv32i"; \
	ELFS="$$(find -L "$$ROOT" \( -type f -o -type l \) -a \( -iname '*.elf' -o -iname '*.elf32' \) 2>/dev/null | sort)"; \
	if [ -z "$$ELFS" ]; then \
	  echo "No .elf files found under $$ROOT"; \
	  echo "Debug listing (2 levels):"; \
	  find "$$ROOT" -maxdepth 2 -print || true; \
	  exit 2; \
	fi; \
	PASS=0; FAIL=0; TOTAL=0; \
	for ELF in $$ELFS; do \
	  TOTAL=$$((TOTAL+1)); \
	  REL="$${ELF#$$ROOT/}"; \
	  BASE="$${ELF%.*}"; \
	  HEX="$$BASE.hex"; \
	  LOG="$(VSIM_LOG_DIR)/$${REL%.*}.log"; \
	  mkdir -p "$$(dirname "$$LOG")"; \
	  if [ ! -f "$$HEX" ] || [ "$$ELF" -nt "$$HEX" ]; then \
	    riscv64-unknown-elf-objcopy -O verilog "$$ELF" "$$HEX"; \
	  fi; \
	  ENTRY=$$(riscv64-unknown-elf-readelf -h "$$ELF" | awk '/Entry point/{print $$NF}'); \
	  ENTRY_HEX=$${ENTRY#0x}; \
	  set +e; \
	  $(VSIM) $(VSIM_FLAGS) \
	    $(VSIM_OPT_TOP) \
	    "+test_program=$$HEX" \
	    "+boot_addr=$$ENTRY_HEX" \
	    > "$$LOG" 2>&1; \
	  RC=$$?; \
	  set -e; \
	  if [ $$RC -eq 0 ] && grep -q "ALL TESTS PASSED" "$$LOG"; then \
	    echo "PASS    $$REL" | tee -a "$$SUMMARY"; \
	    PASS=$$((PASS+1)); \
	  else \
	    echo "FAIL    $$REL" | tee -a "$$SUMMARY"; \
	    FAIL=$$((FAIL+1)); \
	  fi; \
	done; \
	echo "" | tee -a "$$SUMMARY"; \
	echo "TOTAL=$$TOTAL PASS=$$PASS FAIL=$$FAIL" | tee -a "$$SUMMARY"; \
	echo "Summary saved to: $$SUMMARY"; \
	true

.PHONY: gen-certify-vsim
gen-certify-vsim: gen questa-compile certify-vsim
