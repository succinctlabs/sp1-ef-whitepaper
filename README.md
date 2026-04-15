# EF zkVM Security Sprint

Resources and reference materials for the [whitepaper deliverables](https://zkevm.ethereum.foundation/blog/cryptography-research-update) of the [EF zkVM Security Sprint](https://blog.ethereum.org/2025/12/18/zkevm-security-foundations).

## Milestones

| Milestone | Deadline | Scope |
|-----------|----------|-------|
| **W1** | May 1, 2026 | Architecture overview: segmentation and recursive structure |
| W2 | Sep 1, 2026 | Architecture details: buses, memory, instruction fetching, in-depth recursive structure |
| W3 | Dec 1, 2026 | Security argument: proof sketch that the zkVM is an argument of knowledge |

Full requirements: [`resources/zkvm_architecture_whitepaper_details.pdf`](resources/zkvm_architecture_whitepaper_details.pdf).

## Contents

- [`vanillaVM-W1/`](vanillaVM-W1) — reference W1 deliverable illustrating expected scope and rigor. See [`w1-sample.pdf`](vanillaVM-W1/w1-sample.pdf) for the compiled output, or `w1-vanilla.tex` for the source.
- [`resources/`](resources) — whitepaper requirements and supporting documents.

> The vanilla zkVM is **illustrative only** — not necessarily implementable, efficient, or sound. Use it as a structural and notational guide.

## Related

- [soundcalc](https://github.com/ethereum/soundcalc) — security evaluation of inner-segment arguments.
