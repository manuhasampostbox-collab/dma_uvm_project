[README_dma_uvm.md](https://github.com/user-attachments/files/34834590/README_dma_uvm.md)
# dma_uvm_project
UVM testbench for a SystemVerilog DMA controller, built with pyuvm and cocotb on Verilator, with a scoreboard, constrained-random sequences and functional coverage.
# dma-uvm-verification

A UVM testbench for a memory-to-memory DMA controller (SystemVerilog RTL, APB register interface),
written with **pyuvm** (the Python implementation of UVM) on **cocotb**, simulated with Verilator.

## UVM structure
    test -> env -> agent (sequencer, driver, monitor) + scoreboard
- **Sequence items and sequences:** `DmaItem`, `RandomSeq` (constrained-random valid transfers),
  `ErrorSeq` (directed illegal configurations).
- **Driver:** fills the source memory, programs the DMA over APB, waits for completion.
- **Monitor:** watches the APB bus, snapshots the memory before and after each transfer, and
  publishes an observation through an analysis port.
- **Scoreboard:** an independent reference model checks the final status, the whole memory image
  (not only the destination), and the interrupt output for every transfer.
- **Functional coverage:** transfer length bins, result kinds (ok, zero length, unaligned, out of
  range), IRQ enable, and overlapping regions. Tests fail on coverage holes.
- **Tests:** `SmokeTest`, `ErrorTest`, `RandomTest`, `RegressionTest`.

## Run
    pip install verilator cocotb pyuvm      # needs g++ and make
    python3 run_uvm.py                      # all four tests, or: python3 run_uvm.py RandomTest

Result when I ran it: all four tests passed (up to 86 transfers checked per test, 100% coverage
of the defined bins, 0 scoreboard errors).

## Do the tests catch bugs?
Yes. Changing the DMA's last-word condition so it copies one word too few made the
RegressionTest fail with memory and status mismatches. I restored the correct RTL afterwards.

## Limits
This is UVM methodology in Python (pyuvm), not the SystemVerilog UVM library, because the
open-source simulator used here does not run the SystemVerilog UVM library. No RAL, no
virtual sequences or multiple agents, and the design is simulated only (no FPGA or silicon).
