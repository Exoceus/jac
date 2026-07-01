# JAC — Jatin's Accelerator Chip

An FPGA-based hardware accelerator, built from first principles, that runs
**MNIST digit classification (inference first, training later)** on a
**PYNQ-Z2 board (Xilinx Zynq XC7Z020 SoC)**.

This file is the single source of truth for the plan and progress. It is
git-tracked and outlives any individual working session. Update the **Session
Log** every time we make progress.

---

## North Star

> Strip away the ML vocabulary and a neural network — forward pass *and* the
> backprop that trains it — is almost entirely **matrix multiply-accumulate**.
> Build a fast, correct matrix multiplier in the fabric, keep it fed with data,
> and you have ~90% of an AI accelerator. Everything else is garnish.

**Milestone ordering:** get *inference* of a tiny pre-trained MNIST model
working end-to-end first; add the *training* (backward pass + weight update)
onto the same compute core only after that works.

---

## The Hardware (what we're targeting)

- **Board:** PYNQ-Z2 (TUL `1M1-M000127DVB`)
- **Chip:** Xilinx Zynq **XC7Z020** SoC — two halves on one die:
  - **PS (Processing System):** dual-core ARM Cortex-A9 + DDR3 controller,
    512 MB DDR3. Boots Linux + Python (PYNQ). This orchestrates.
  - **PL (Programmable Logic):** the FPGA fabric where JAC lives.
    - ~53K LUTs, ~106K flip-flops
    - **220 DSP slices** ← the compute budget (each = one hardware MAC)
    - ~4.9 Mb (~600 KB) BRAM ← tiny; data movement is a first-class problem

---

## How We Work (the tutor contract — read this every session)

- **Jatin builds; the tutor does not.** No solution HDL/RTL is ever written for
  him. Guidance, questions, reading his code, running his sims, reading
  waveforms/logs — yes. Writing the design — no.
- **Socratic + first principles.** New concept → *why it exists / what problem
  it solves* before *how*. Confirm the "why" landed before moving on.
- **Simulation first.** Learn and verify in a fast sim loop; only touch Vivado
  and the real board when a milestone demands it.
- **Every build target has a "Definition of Done"** and is checked against a
  numpy/PyTorch reference.

---

## Locked Decisions (don't re-litigate without a reason)

| Decision | Choice | Why |
|---|---|---|
| Language | **SystemVerilog** | Fewer footguns than Verilog; native to the Xilinx/Zynq world; most tutorials |
| First sim toolchain | **Icarus Verilog + Surfer** | Seconds-long feedback loop; full signal visibility; no 100 GB install. NOTE: GTKWave's prebuilt cask is broken on this macOS — use Surfer (native ARM build or browser at app.surfer-project.org) as the VCD viewer instead. |
| Vivado | **Deferred** to Phase 4 | Needed only to target the real chip; painful to learn HDL inside |
| HLS (C++ → hardware) | **Rejected** | Hides the exact things we're here to learn |
| Number format | **Fixed-point** (Q-format TBD in 0.2) | Integer arithmetic in hardware; float is expensive on this fabric |
| First model | **Small MLP** (e.g. 784→hidden→10) | Pure matmul + ReLU + argmax; no convolutions to start |

---

## Curriculum

Legend: `[ ]` not started · `[~]` in progress · `[x]` done

### Phase 0 — Foundations (simulation only, no board)
- [~] **0.1 HDL mind-shift.** Combinational vs sequential, the clock, registers.
  Install toolchain; simulate a tiny registered design; read it in GTKWave.
  *DoD:* can write + simulate a clocked counter and explain every waveform edge.
- [ ] **0.2 Fixed-point.** Pick a Q-format for MNIST pixels/weights; understand
  bit-growth on multiply and how to round/saturate.
  *DoD:* can hand-encode a value, multiply two fixed-point numbers, and predict
  the result's format and rounding.
- [ ] **0.3 The MAC unit.** Single multiply-accumulate with an accumulator
  register + testbench.
  *DoD:* MAC computes a dot product over N cycles; matches a numpy reference.

### Phase 1 — One neuron (dot-product engine)
- [ ] **1.1 Streaming dot product** over an MNIST-length vector.
- [ ] **1.2 Accumulator width / saturation** — don't overflow.
- [ ] **1.3 ReLU** (combinational, trivial — a good confidence win).
  *DoD:* one neuron's output for a real input vector, matches numpy.

### Phase 2 — One layer (matrix-vector multiply)
- [ ] **2.1 Many neurons** — reuse the MAC hardware across output neurons
  (the scheduling problem).
- [ ] **2.2 BRAM storage** for weights/activations; memory layout.
- [ ] **2.3 Control FSM** that sequences the whole matvec.
  *DoD:* a full fully-connected layer forward pass in sim, matches numpy.

### Phase 3 — Full inference network (still sim)
- [ ] **3.1 Chain layers** into the MLP.
- [ ] **3.2 argmax** for classification (cheaper than full softmax).
- [ ] **3.3 Fixed-point accuracy** vs the float reference — measure the drop.
  *DoD:* full MNIST inference in sim classifies test images within an agreed
  accuracy margin of the float model.

### Phase 4 — Onto the real chip (Vivado + PYNQ)
- [ ] **4.1 Vivado + Zynq PS/PL split + AXI basics.**
- [ ] **4.2 Package JAC as an AXI peripheral**; DMA data from DDR.
- [ ] **4.3 Drive it from Python via PYNQ** — classify a real image on hardware.
- [ ] **4.4 Measure** latency/throughput vs ARM-only software.
  *DoD:* a PYNQ notebook sends an MNIST image to the fabric, gets the right
  digit back.

### Phase 5 — Stretch: training on-chip
- [ ] Backward pass, gradients, weight update on the same compute core.
  The original dream. Scope after Phase 4 lands.

---

## Current Focus

**Phase 0.1.** Immediate actions:
1. Install `icarus-verilog` (done, v13) + a VCD viewer (Surfer — GTKWave is broken on this macOS).
2. Answer the "three MAC questions" (multiply = which logic type? what holds the
   accumulator across steps? what enforces multiply-then-add ordering when both
   units exist simultaneously?).
3. Next tutor step after that: first tiny build target in simulation.

---

## Parking Lot / Open Questions
- Exact Q-format (fractional bits) for MNIST — decide in 0.2.
- Hidden-layer size for the MLP — trade accuracy vs fabric resources.
- Where the pre-trained weights come from (train in PyTorch on laptop, export).

---

## Session Log (append-only, newest at bottom)

### 2026-06-30 — Session 1
- Identified the board: PYNQ-Z2 / Zynq XC7Z020. Established the north star
  (it's all matmul), the PS/PL split, and the sim-first / inference-first plan.
- Calibrated Jatin: strong software + ML (has implemented backprop by hand),
  **new to HDL**, no tools installed yet.
- Locked the decisions in the table above. Created this roadmap + Claude memory.
- Handed off: install toolchain + answer the three MAC reasoning questions.
