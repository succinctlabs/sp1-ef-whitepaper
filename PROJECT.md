# SP1 zkVM Architecture Whitepaper — Project Tracker

## Overview

This project produces an architecture whitepaper for the **SP1 zkVM** as part of the [EF zkVM Security Sprint](https://blog.ethereum.org/2025/12/18/zkevm-security-foundations). The paper must document SP1's design with enough precision for independent security review.

- **Grant requirements**: `resources/zkvm_architecture_whitepaper_details.pdf`
- **Reference template**: `vanillaVM-W1/` (vanilla zkVM example — structural/notational guide only, not meant to be implementable or sound)
- **SP1 source code**: `~/sp1/`

## Milestones

| Milestone | Deadline | Scope | Status |
|-----------|----------|-------|--------|
| **W1** | May 1, 2026 | Architecture overview: segmentation and recursive structure | **In progress** |
| W2 | Sep 1, 2026 | Architecture details: buses, memory, instruction fetching, in-depth recursive structure | Not started |
| W3 | Dec 1, 2026 | Security argument: proof sketch that the zkVM is an argument of knowledge | Not started |

## W1 Requirements (from EF spec)

The W1 deliverable must cover:

1. **Execution model (Sec 1–2 in template)**
   - Notion of program, program execution, VM state
   - List of supported instructions
   - Statement about program execution to be asserted in the final proof

2. **Segmentation (Sec 2 in template)**
   - How execution is split into segments
   - Whether segments are further partitioned into chips
   - Whether some instructions are precompiles proven separately

3. **Interactions (Sec 3 in template)**
   - Memory handling w.r.t. segmentation (local-global, ROM-RAM, etc.)
   - Consistency argument type (permutation, lookup/logup, etc.) and randomness source
   - Bus organization and consistency enforcement (list buses, argument types)

4. **Individual proofs (Sec 4 in template)**
   - How circuits are created from segments and chip groups
   - Circuit types (by size, security level, proof size) and how many of each
   - Commitment schemes and argument types (FRI-based, lookup-based, etc.)
   - Additional subprotocols (e.g., lookup arguments for range checks)

5. **Recursion structure (Sec 4 in template)**
   - Recursion topology (linear chain, tree, configurable fan-in)
   - What each recursion step proves (NP relation of each node)
   - Mapping between recursion nodes and soundcalc circuit instances

## File Structure

```
vanillaVM-W1/
  w1-vanilla.tex    — Main LaTeX source (currently being edited)
  macros.tex         — Shared macros and notation
  w1-sample.pdf      — Compiled reference output
resources/
  zkvm_architecture_whitepaper_details.pdf — EF requirements doc
```

## Editing Status

The document (`w1-vanilla.tex`) has been largely adapted to describe **SP1**. The vanilla template `instruction` boxes have all been removed. The title is now "SP1 zkVM Architecture".

| Section | Title | Status | Notes |
|---------|-------|--------|-------|
| Overview | Overview | **SP1-specific** | RV64IM, chip clusters, LogUp+GKR, septic curve, Hypercube/BaseFold/KoalaBear, normalize/compose/deferred |
| 1 | Execution Model | **SP1-specific** | |
| 1.1 | Program, Execution, and VM State | **SP1-specific** | RV64IM ELF binary, 64-bit PC, 32 registers (x0–x31, 64-bit), x0 hardwired to zero |
| 1.2 | Supported Instructions | **SP1-specific** | RV64IM + categorized precompiles (hash, EC, field arith, big-int, system) via ecall |
| 2 | Segmentation | **SP1-specific** | |
| 2.1 | Execution Shards | **SP1-specific** | Trace-area-based sharding (element threshold 2^28+2^27, height threshold 2^22) |
| 2.2 | Shard layout | **SP1-specific** | Chip cluster model, forward-references Sec 4.2 |
| 3 | Correct execution of a program | **SP1-specific** | |
| 3.1 | Predicates for individual operations | **SP1-specific** | ALU, memory loads/stores, control flow, precompiles via ecall |
| 3.2 | Correct execution of a single step | **SP1-specific** | Step predicate as disjunction over all opcodes |
| 3.3 | Chip interactions and the bus | **SP1-specific** | Send/receive model, LogUp argument, Local vs Global scopes, septic curve |
| 3.4 | Memory consistency | **SP1-specific** | Global-bus-based memory (no Merkle trees), MemoryGlobalInit/Final, timestamps, MemoryLocal |
| 4 | Proof Architecture | **SP1-specific** | |
| 4.1 | Topology | **SP1-specific** | SP1 recursion DAG (core/normalize/compose/deferred), VK Merkle tree, programs vs. verifiers |
| **4.2** | **`core` circuits** | **SP1-specific** | Chip cluster architecture, shard relation, 4 cluster types (core, core+P, precompile-P, mem-boundary) |
| **4.3** | **`normalize` circuit** | **SP1-specific** | 1-to-1 shape normalization, shard proof verification, aggregation checks, completeness conditions |
| **4.4** | **`compose` circuit** | **SP1-specific** | N-to-1 aggregation (N≤4), continuity checks, cumulative sum accumulation, VK authentication |
| **4.5** | **`deferred` circuit** | **SP1-specific** | 1-to-1 deferred sub-computation wrapper, VK Merkle verification, deferred digest update |
| **5** | **Proof system instantiations** | **In progress** | SP1-specific content started; some TODOs remain |
| 5.1 | LogUp + GKR Details | **TODO** | `TODO: Ask Min about the LogUp+GKR business` |
| 5.2 | Zerocheck Details | **In progress** | Has SP1-specific zerocheck description with geq function and batching |

## SP1-Specific Content Written So Far

### Overview
SP1-specific introduction: RV64IM, chip clusters, LogUp+GKR, septic curve accumulation, Hypercube/BaseFold/KoalaBear, normalize/compose/deferred recursion pipeline.

### Section 1: Execution Model
- RV64IM ELF binary program format
- 64-bit PC, 32 registers (x0–x31, 64-bit each, x0 hardwired to zero)
- Byte-addressed writable memory, immutable code

### Section 1.2: Supported Instructions
- Full RV64IM instruction set
- Categorized precompiles invoked via ecall with syscall code in register t0:
  hash (SHA-256, Keccak, Poseidon2), elliptic curve (Ed25519, Secp256k1, Secp256r1, BN-254, BLS12-381),
  field arithmetic (Fp/Fp2 for BN-254/BLS12-381), big-integer (uint256, u256×2048), system (halt, I/O, commit)

### Section 2: Segmentation (Sharding)
- Trace-area-based sharding with configurable thresholds (element threshold 2^28+2^27, height threshold 2^22)
- Chip cluster model: each shard proven by a chip cluster with execution trace, precompile, and infrastructure chips

### Section 3: Correct Execution
- Operation predicates grouped by category (ALU, memory, control flow, precompiles)
- Send/receive bus model with LogUp argument (proved via GKR)
- Local scope (balance within shard) vs Global scope (septic curve accumulation across shards)
- Memory consistency via global bus: MemoryGlobalInit/MemoryGlobalFinal chips, timestamp-based interactions, no Merkle trees

### Section 4.1: Topology
- Recursion DAG figure with 4 circuit types: core → normalize → compose/deferred
- Table of circuit types with relations
- Programs vs. verifiers distinction (recursive proof aggregation embeds verifier as program)
- VK Merkle tree: ~181k leaves, one per (circuit_type, shape); compose/deferred verify Merkle membership

### Section 4.2: `core` circuits
- Shard relation $\mathcal{R}_{\mathrm{shard}}^{(C)}$: chip constraints, local LogUp balance, global cumulative sum
- 4 cluster types: base core, extended core (core+P), deferred precompile (precompile-P), memory boundary (mem-boundary)
- Detailed listing of chips in each cluster type
- Inter-shard communication via global accumulation bus (Global table)

### Section 4.3: `normalize` circuit
- 1-to-1 shape normalization to compress_shape
- Shard proof verification via Fiat–Shamir challenger
- Aggregation checks: first-shard PC validation, global cumulative sum accumulation, public values propagation, completeness conditions
- Formal relation $\mathcal{R}_{\mathrm{norm}}$

### Section 4.4: `compose` circuit
- N-to-1 recursion (1 ≤ N ≤ 4)
- Sequential continuity, program identity, cumulative sum accumulation
- VK root consistency checks
- Formal relation $\mathcal{R}_{\mathrm{compose}}$

### Section 4.5: `deferred` circuit
- 1-to-1 wrapper for deferred sub-computation proofs
- VK Merkle verification, proof verification, deferred digest update via Poseidon-2
- Formal relation $\mathcal{R}_{\mathrm{def}}$

### Section 5: Proof system instantiations
Key concepts introduced:
- Hypercube proof system framework
- Jagged polynomial commitment scheme (sparse PCS for varying-height tables)
- BaseFold (internal dense PCS)
- KoalaBear field (32-bit)
- Table cluster architecture (~2^12 columns, padded to height 2^22)
- Protocol steps: (1) multilinear commitment, (2) LogUp+GKR, (3) zerocheck sumcheck, (4) batch PCS evaluation
- Padding mitigations: padding rows contribute easily computable terms to sumchecks
- Verifier as arithmetic circuit: independent of individual table heights, ~2^18 possible circuits
- Zerocheck details: constraint batching, geq function for padding handling

### Known TODOs
- `{\color{red} TODO: Ask Min about the LogUp+GKR business.}` (Sec 5.1)
- `{\color{red} maybe ±1}` note about GKR round count (Sec 5)
- Item 2 in the jagged PCS enumeration is incomplete (sentence cut off mid-thought)
- The `\widehat{\mathrm{geq}}` function and zerocheck batching are partially described

### Architectural decisions
- **Shrink/wrap circuits omitted**: The shrink (field reduction) and wrap (SNARK wrapping for on-chain verification) circuits exist in SP1 but are omitted from this paper as out of scope for W1.

## Open Questions / Future Work Notes

- **Septic curve formalization**: Section 3.3 and 4.2 reference the "septic curve" (degree-7 elliptic curve over the base field) used for global cumulative sums (`SepticDigest` in code). Currently introduced informally as $\sigma \in E(\mathbb{F})$. Needs decisions:
  - Should we formally define the curve equation and the accumulation mechanism in this paper, or defer to W2?
  - The `SepticDigest` stores $(x, y)$ coordinates. The accumulation is additive (EC point addition). Worth describing the concrete curve parameters and why degree 7 was chosen.
  - The completeness check $\sum_i \sigma_i = \mathcal{O}$ is enforced in the normalize and compose circuits when `is_complete=1`.
  - Key code locations: `~/sp1/crates/hypercube/src/septic_curve.rs`, `~/sp1/crates/hypercube/src/septic_digest.rs`, `~/sp1/crates/hypercube/src/air/public_values.rs` (line 119: `global_cumulative_sum: SepticDigest<T>`)

- **LogUp + GKR details**: Section 5.1 is currently a TODO placeholder. Need input from Min on the LogUp+GKR protocol specifics.

- **Jagged PCS description**: Item 2 in the jagged PCS enumeration (Sec 5) is incomplete — the sentence about evaluation proof for a point $\eta = \eta_n || \eta_k$ cuts off mid-thought.

## Key SP1 Architecture Concepts (for reference when writing)

These are high-level pointers into the SP1 codebase (`~/sp1/`) to guide section writing:

- **Core VM**: `~/sp1/crates/core/executor/` (opcode.rs, state.rs, register.rs, syscall_code.rs)
- **Machine/chips**: `~/sp1/crates/core/machine/src/riscv/mod.rs` (chip enumeration), `~/sp1/crates/core/machine/src/memory/` (memory chips)
- **Prover**: `~/sp1/crates/prover/` (recursion.rs, shapes.rs)
- **Recursion circuits**: `~/sp1/crates/recursion/circuit/src/machine/` (core.rs=normalize, compress.rs=compose, deferred.rs, vkey_proof.rs)
- **Proof system**: `~/sp1/crates/hypercube/` (septic_curve.rs, septic_digest.rs)
- **Sharding config**: `~/sp1/crates/core/executor/src/opts.rs` (ELEMENT_THRESHOLD, HEIGHT_THRESHOLD)

## Template Instructions (from EF)

All template `instruction` boxes have been removed and replaced with SP1-specific content:
- ~~Sec 2.2 (Segment layout)~~: Replaced with chip cluster description
- ~~Sec 3.1 (Predicates)~~: Replaced with SP1 operation categories
- ~~Sec 3.2 (Step predicate)~~: Replaced with SP1 step predicate
- ~~Sec 3.3 (Bus delegation)~~: Replaced with send/receive bus model, Local/Global scopes
- ~~Sec 3.4 (Committed memory)~~: Replaced with global-bus-based memory
- ~~Sec 4 (Proof architecture)~~: SP1-specific topology, relations, and circuit descriptions
- ~~Sec 5 (Proof system)~~: SP1-specific Hypercube proof system description (partially complete)
