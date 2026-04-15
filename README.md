# Vanilla zkVM Architecture — W1 Deliverable Template

This folder contains a reference example to help teams prepare their **W1 deliverable** as part of the [EF zkVM Security Sprint](https://zkevm.ethereum.foundation/blog/cryptography-research-update).

## Background

The EF Cryptography Research team has published [detailed requirements](https://crypto.ethereum.org/docs/zkvm_architecture_whitepaper_details.pdf) for a zkVM architecture whitepaper, to be delivered across three milestones aligned with the sprint timeline:

| Milestone | Deadline | Scope |
|-----------|----------|-------|
| **W1** | **May 1, 2026** | Architecture overview: segmentation and recursive structure |
| W2 | Sep 1, 2026 | Architecture details: buses, memory, instruction fetching, in-depth recursive structure |
| W3 | Dec 1, 2026 | Security argument: proof sketch that the zkVM is an argument of knowledge |

## Contents

| File | Description |
|------|-------------|
| `w1-vanilla.tex` | Reference W1 document describing a "Vanilla zkVM Architecture" |
| `macros.tex` | LaTeX macros and notation used in the document |

## About the Vanilla Document

`w1-vanilla.tex` is a **reference example** illustrating the expected scope and level of rigor for a W1 submission. It was authored by the EF Cryptography Research team (George Kadianakis, Dmitry Khovratovich, Benedikt Wagner, Arantxa Zapico) and is dated April 8, 2026.

> **Note:** This document is intentionally illustrative. The described zkVM is not necessarily implementable, efficient, or even sound. It serves as a structural and notational guide. Submitted documents should provide additional detail where applicable.

The document covers all sections expected in a W1 document:

1. **Execution Model** — Defines the program, VM state (program counter, registers, byte-addressed memory), execution trace, and the RV+ (RISC-V extended with Keccak and Poseidon precompiles) instruction set.

2. **Segmentation** — Explains how the execution trace is divided into fixed-length segments, each represented as a *segment trace* plus specialized *chips* (Keccak, Poseidon, range check), connected via an interaction *bus*.

3. **Correct Execution** — Formalizes per-operation predicates, the step predicate (inline vs. bus-deferred verification), and memory commitment via Merkle trees.

4. **Proof Architecture** — Describes the full circuit topology:
   - **Inner circuits** (`inner-step`, `inner-keccak`, `inner-poseidon`, `inner-range`) — leaf-layer proofs for one segment.
   - **`segment`** — recursively verifies all four inner proofs via a shared bus commitment.
   - **`convert`** — 1-to-1 normalization circuit wrapping a segment proof.
   - **`combine`** — 2-to-1 recursive circuit for binary aggregation of segment proofs.
   - **`embed`** — produces the final proof attesting to the full program execution.

5. **Proof System Instantiations** — Placeholder section indicating where teams should describe their AIR / IOP / poly-IOP reductions and link to their [soundcalc](https://github.com/ethereum/soundcalc) circuit entries.

## How to Use

1. Compile `w1-vanilla.tex` with a LaTeX distribution that includes the packages listed in its preamble (`tikz`, `tcolorbox`, `pgfplots`, `siunitx`, etc.).
2. Read the document alongside the [full whitepaper requirements](https://crypto.ethereum.org/docs/zkvm_architecture_whitepaper_details.pdf).
3. Use the structure and notation as a template for your own W1 submission. Pay particular attention to the teal **instruction boxes** throughout the document — they highlight exactly what teams are expected to fill in.

## Further Reading

- [zkEVM Security Sprint: February Update](https://zkevm.ethereum.foundation/blog/cryptography-research-update) — announcement of the whitepaper guidelines and W1–W3 milestones.
- [Full whitepaper requirements (PDF)](https://crypto.ethereum.org/docs/zkvm_architecture_whitepaper_details.pdf)
- [soundcalc](https://github.com/ethereum/soundcalc) — tool for security evaluation of inner-segment arguments.
- [zkVM Standards](https://github.com/ethereum/zkvm-standards)
