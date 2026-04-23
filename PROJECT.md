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

The document (`w1-vanilla.tex`) currently uses the vanilla template structure. Below tracks which sections have been adapted to describe **SP1** vs. still containing **vanilla template** content.

| Section | Title | Status | Notes |
|---------|-------|--------|-------|
| Overview | Overview | Template | Still describes vanilla zkVM |
| 1 | Execution Model | Template | Generic RISC-V execution model |
| 1.1 | Program, Execution, and VM State | Template | |
| 1.2 | Supported Instructions | Template | Lists generic ops; needs SP1-specific instruction set |
| 2 | Segmentation | Template | |
| 2.1 | Execution Segments | Template | |
| 2.2 | Segment layout | Template | Generic chips (Keccak, Poseidon, range check) |
| 3 | Correct execution of a program | Template | |
| 3.1 | Predicates for individual operations | Template | |
| 3.2 | Correct execution of a single step | Template | |
| 3.3 | Delegating expensive operations using a Bus | **Partial** | Bus predicates are template; Local/Global interaction scopes added (SP1-specific) |
| 3.4 | Committed memory | Template | Uses Merkle tree commitment; needs SP1's actual approach |
| 4 | Proof Architecture | Template | |
| 4.1 | Topology | Template | Vanilla recursion tree (inner/segment/convert/combine/embed) |
| **4.2** | **`core` circuits** | **SP1-specific** | **Chip cluster architecture, shard relation, 4 cluster types (core, core+P, precompile-P, mem-boundary)** |
| 4.3 | `segment` circuit | Template | |
| 4.4 | `convert` circuit | Template | |
| 4.5 | `combine` circuit | Template | |
| 4.6 | `embed` circuit | Template | |
| **5** | **Proof system instantiations** | **In progress** | **SP1-specific content started** |
| 5.1 | LogUp + GKR Details | TODO | `TODO: Ask Min about the LogUp+GKR business` |
| 5.2 | Zerocheck Details | In progress | Has SP1-specific zerocheck description |

## SP1-Specific Content Written So Far

### Section 4.2: `core` circuits
SP1's chip cluster architecture, including:
- **Shard relation** $\relation_{\mathrm{shard}}^{(C)}$: formal relation for shard proofs (chip constraints, local LogUp balance, global cumulative sum)
- **4 cluster types**: base core, extended core (core+P), deferred precompile (precompile-P), memory boundary (mem-boundary)
- Detailed listing of chips in each cluster type
- Inter-shard communication via global accumulation bus (\texttt{Global} table)

### Section 3.3: Local/Global interaction scopes
- **Local** interactions balance within a shard (LogUp + GKR)
- **Global** interactions accumulate across shards via septic curve points ($\sigma \in E(\mathbb{F})$), checked at the end via $\sum_i \sigma_i = \mathcal{O}$

### Section 5: Proof system instantiations
Key concepts introduced:

- **Hypercube proof system**: SP1's proof system framework
- **Jagged polynomial commitment scheme**: sparse PCS for the table layout with varying-height tables
- **BaseFold**: internal dense PCS used within jagged PCS
- **KoalaBear field**: 32-bit field used by SP1
- **Table cluster architecture**: tables with ~2^12 columns, padded to height 2^22
- **Protocol steps**: (1) multilinear commitment, (2) LogUp+GKR, (3) zerocheck sumcheck, (4) batch PCS evaluation
- **Padding mitigations**: padding rows contribute easily computable terms to sumchecks
- **Verifier as arithmetic circuit**: independent of individual table heights, ~2^18 possible circuits

### Known TODOs in Section 5
- `{\color{red} TODO: Ask Min about the LogUp+GKR business.}` (Sec 5.1)
- `{\color{red} maybe $\pm 1$}` note about GKR round count (Sec 5)
- Item 2 in the jagged PCS enumeration is incomplete (sentence cut off mid-thought)
- The `\widehat{\mathrm{geq}}` function and zerocheck batching are partially described

## Open Questions / Future Work Notes

- **Septic curve formalization**: Section 4.2 now references the "septic curve" (degree-7 elliptic curve over the base field) used for global cumulative sums (`SepticDigest` in code). Currently introduced informally as $\sigma \in E(\mathbb{F})$. Needs decisions:
  - Should we formally define the curve equation and the accumulation mechanism in this paper, or defer to W2?
  - The `SepticDigest` stores $(x, y)$ coordinates. The accumulation is additive (EC point addition). Worth describing the concrete curve parameters and why degree 7 was chosen.
  - The completeness check $\sum_i \sigma_i = \mathcal{O}$ happens in `complete.rs:147`. This should be formalized in the recursion relations (Sec 4 shrink/wrap circuits, once written for SP1).
  - Key code locations: `~/sp1/crates/hypercube/src/septic_curve.rs`, `~/sp1/crates/hypercube/src/septic_digest.rs`, `~/sp1/crates/hypercube/src/air/public_values.rs` (line 119: `global_cumulative_sum: SepticDigest<T>`)

## Key SP1 Architecture Concepts (for reference when writing)

These are high-level pointers into the SP1 codebase (`~/sp1/`) to guide section writing:

- **Core VM**: `~/sp1/core/` and `~/sp1/zkvm/`
- **Prover**: `~/sp1/prover/`
- **Recursion**: `~/sp1/recursion/`
- **Crates (chips, etc.)**: `~/sp1/crates/`

## Template Instructions (from EF)

The vanilla template includes teal `instruction` boxes indicating where teams should customize. These appear in:
- Sec 2.2 (Segment layout): "Teams should describe how their bus consistency argument works"
- Sec 3.1 (Predicates): "Teams should list all operations supported by their zkVM"
- Sec 3.2 (Step predicate): "Teams should define the step predicate for their architecture"
- Sec 3.3 (Bus delegation): "Teams should describe how expensive operations are delegated"
- Sec 3.4 (Committed memory): "Teams should describe how memory is committed"
- Sec 4 (Proof architecture): "Teams should add footnotes linking to soundcalc entries"
- Sec 5 (Proof system): "Teams should explain how the relations are proven using AIR / IOPs / poly-IOPs"
