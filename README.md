# PathForge — Structured Coverage for AI-Powered QA

*A domain-specific language for modeling, executing, and comparing test coverage through explicit system traversal paths.*

---

## The Problem with Traditional QA at Scale

Traditional QA describes **user behavior**. A tester writes a scenario, someone executes it, and its coverage contribution is implicit — buried in prose, impossible to compare across releases, and entirely dependent on the person who wrote it.

This works at small scale. It breaks down when:

- The system under test has **dozens of interacting components** and **hundreds of atomic behaviors**
- Tests are generated or augmented by **AI**, which has no intrinsic notion of what "coverage" means
- You need to answer questions like: *"Did we test this combination before? What changed between this campaign and the last one? Which features are only tested once?"*
- The team rotates and **institutional knowledge lives in people's heads**, not in the test suite

PathForge was designed to solve exactly this. Not by writing better test cases — but by making the underlying system structure the first-class citizen, and deriving coverage from it.

---

## Step 1: Model the System First

Before any test can be generated, PathForge requires a complete structural model of the system under test: **`system.json`**.

This file is the source of truth. It captures:

| Component | What it models |
|---|---|
| **Technical Branches (TB)** | Every discrete capability of the system — each split into atomic testable leaves (TB1.a, TB1.b…) |
| **Parameter Branches (PB)** | Variation dimensions — networks, token types, configuration values, device settings |
| **Root Dependencies (RD)** | The execution environment — physical device models, firmware variants |
| **Canopy Dependencies (CD)** | External integration contexts — wallets, DeFi protocols, governance platforms, NFT marketplaces |
| **Constraints** | Conditions that must hold for a leaf to be reachable (e.g. `BlindSigning=ON`, `forbidden_env: emulator`) |
| **Execution dependencies** | Predecessor relationships between leaves within a path (e.g. TB14.b requires TB14.a to have run first) |
| **Translations** | Human-readable rendering of each leaf in context of its parameters |

The mind map below represents a firmware app modeled in `system.json` — 14 Technical Branches, 112 atomic leaves, 5 Parameter Branches, 4 device contexts, and 4 categories of external integrations.

This is not optional infrastructure. **Without the model, coverage is undefined.** You cannot know what you're missing if you haven't described what exists.

![Firmware App](/assets/img.png)

---

## The DSL

Once the system is modeled, coverage is expressed as **traversal paths** through it.

```
RD → TB → TB.leaf[state] → PB → CD
```

Each node in a path encodes a different dimension of execution:

| Node | Meaning |
|---|---|
| `RD` | Which device / environment this path runs on |
| `TB.leaf` | Which atomic system capability is exercised |
| `[state]` | What settings are active (constraints) |
| `PB` | Which parameter value is used |
| `CD` | Which external integration is present |

**Example:**

```
RD2[Nano S+] · [TxScan=ON] · TB7.e → TB12.c → TB13.a/b/c/d/e → TB4.c · PB3[WETH] · CD2[Compound]
```

This single path encodes:

- **Device:** Ledger Nano S Plus
- **Setting constraint:** Transaction Scan must be ON
- **Feature leaves:** message signing with scan → clear signing with scan → all 5 transaction check leaves → base transaction signing
- **Token parameter:** WETH
- **DeFi context:** Compound

This is more information than most traditional test case descriptions — in one line, with structure that can be parsed, compared, and tracked programmatically.

---

## Risk Weighting: Not All Coverage Is Equal

PathForge does not treat all Technical Branches as equally important. Each TB carries a **risk weight** that reflects:

- How critical the feature is to the user's security or funds
- How many downstream features depend on it
- How frequently it changes across firmware releases
- Whether it has historically been a source of regressions

High-weight TBs (e.g. Transaction Signing, Clear Signing, Blind Signing) anchor the campaign. Their full coverage is non-negotiable. Low-weight TBs can be bundled opportunistically into paths that primarily serve higher-risk areas.

This weighting feeds directly into the **parametric coverage score** — a weighted average that reflects not just "did we touch this branch" but "did we touch the branches that matter most, across the parameter values that matter most."

A campaign can report 100% structural coverage (every leaf touched) while remaining at 65% composite confidence — because high-risk branches are exercised only once, with only one token, on only one device. The weighting system makes this gap visible.

---

## Four Coverage Dimensions

PathForge evaluates campaigns across four dimensions simultaneously:

| Dimension | What it measures | Example gap |
|---|---|---|
| **Structural** | Were all 112 leaves reached at least once? | TB3.c never tested — hardware-only restriction not accounted for |
| **Parametric** | Were high-risk parameter variations exercised? | WETH tested, but USDT (6-decimal) and UNI never exercised |
| **Behavioral** | Were real integration scenarios covered? | Blind signing tested in isolation but never with a DeFi protocol or wallet |
| **Fragility** | How many leaves are tested only once? | 108/112 leaves with zero redundancy — a single flaky test removes coverage |

These four dimensions compose into a **single confidence score** that can be tracked across campaign versions. The score tells you not just whether you have coverage — but whether your coverage would survive a regression.

---

## AI-Ready by Design

This is where PathForge's design becomes strategically important.

AI models can generate test cases fluently. They can write steps, scenarios, and assertions. What they cannot do, without grounding, is:

- Know what system capabilities exist
- Know which combinations are invalid (constraint violations)
- Know what "full coverage" looks like
- Produce output that is comparable across runs

When an AI generates raw test cases, you get text. When it generates **DSL paths against a modeled system**, you get:

- **Verifiable coverage** — every path is validated against the system model
- **Constraint enforcement** — the model rejects paths that violate execution dependencies or setting requirements
- **Explainability** — every generated path can be translated into a human-readable scenario automatically, via the `translations` block in `system.json`
- **Stability** — the path index is deterministic; two different AI runs targeting the same coverage produce comparable, diffable output

The DSL acts as a **stable intermediate representation** between the AI's generative output and the human-readable test campaign. This is essential: without it, AI-generated QA is a black box. With it, coverage is auditable at every level.

This matters especially for security-critical products, where *"the AI said it was covered"* is not an acceptable audit response.

---

## Campaigns as Structured Data

A test campaign in PathForge is not a document — it is a **structured data artifact**.

```
P01  Nano X    [default]       TB4.c/h → TB12.a/e/f/h/i   · USDC   · Uniswap
P02  Nano S+   [default]       TB4.c/h → TB12.b/d/g        · UNI    · 1inch
P03  Stax      [default]       TB8.a–n → TB4.c             · DAI    · Aave
...
```

Each row is:

- **Parseable** — every node maps to a defined entity in `system.json`
- **Renderable** — the translation layer converts it into human-readable steps automatically
- **Comparable** — two campaigns can be diffed structurally, not just textually

---

## Diffable Coverage: A Historical Record

Because campaigns are structured, they accumulate into a **historical record of coverage**.

Campaign versions (v1, v2, v3…) can be compared programmatically:

### What a diff reveals

```
Campaign v4 → Campaign v5
```

**Added coverage:**
```
+ P01  TB12.e  (proxy contract resolution — previously missing)
+ P03  TB8.f/g/h/j/l/m/n  (7 new EIP-712 leaves — incomplete in v4)
```

**Removed coverage:**
```
- P22  TB4.h · PB1[Base]  (Base L2 no longer tested after path consolidation)
```

**Modified paths:**
```
~ P10  PB3[DAI] → PB3[WETH]  (token parameter changed — DAI-specific behaviors now uncovered)
```

**Coverage trend:**

| Campaign | Composite score | Paths |
|---|---|---|
| v2 | 63% | 20 paths |
| v3 | 77% | 28 paths |
| v4 | 80% | 28 paths — explicit RD distribution |
| v5 | 78% | 20 paths — swap-focused, intentional parametric tradeoff |

This is coverage as version control. You can see:

- Which features gained test coverage and when
- Which regressions were introduced by campaign restructuring
- Whether confidence is trending up or down across releases
- Where to focus the next campaign to maximize ROI

No traditional QA methodology produces this naturally. It requires the underlying structure that PathForge enforces from the start.

---

## Minimum Path, Maximum Coverage

One of PathForge's core optimization objectives is: **cover the most system behaviors in the fewest paths**.

This is a combinatorial problem. Given 112 leaves with constraints and dependencies, find the minimum set of paths that achieves 100% structural coverage, respects all constraints, and maximizes parametric breadth.

The DSL makes this problem tractable:

- Leaves with compatible constraints can be **bundled** into a single path
- Leaves with conflicting constraints (e.g. `BlindSigning=ON` vs `BlindSigning=OFF`) **must** occupy separate paths
- Execution dependencies (TB14.b requires TB14.a) constrain ordering within a path
- High-risk leaves anchor paths; low-risk leaves fill in opportunistically

The result is a campaign that is **dense by design** — not because someone was clever, but because the system model makes the structure of the problem explicit.

v5 of the Ethereum campaign covers 112 leaves across 20 paths — an average of 5.6 leaf-equivalents per path. A traditional approach for the same system would generate 50–100 scenarios with far lower effective coverage density.

---

## Why This Approach Is Necessary

The case for structured coverage modeling becomes unavoidable as three trends converge:

**1. System complexity grows faster than QA bandwidth**
A system with 112 leaves, 4 devices, 5 token types, and 6 DeFi integrations has a theoretical test space in the tens of thousands. No team can explore it manually. Without a model, no AI can explore it reliably either.

**2. AI generates volume, not coverage**
An AI given a prompt will produce test cases. Without grounding in a system model, it will produce test cases that feel comprehensive but leave systematic gaps — especially at integration boundaries and constraint intersections, which are precisely where regressions happen.

**3. Compliance and audit requirements demand traceability**
For security-critical products, "we tested it" is not enough. You need to answer: *what did you test, against which system version, with which configurations, on which devices?* PathForge makes this answerable at every level — from the campaign index down to individual leaf coverage.

---

## Key Comparison

| | Traditional QA | PathForge |
|---|---|---|
| Unit of work | Test case / scenario | Traversal path |
| Coverage | Implicit, inferred | Explicit, structural |
| AI output | Raw steps (opaque) | Validated paths (auditable) |
| Duplication | Common — same path, different words | Eliminated by design |
| Comparison across versions | Requires human review | Diffable programmatically |
| Gap detection | Requires expert knowledge | Visible in the index |
| Parameter expansion | Requires new test cases | Same path, new PB value |
| Confidence score | Subjective | Multi-dimensional, versioned |
| Minimum path problem | Unsolved | Optimized by constraint model |

---

## Summary

PathForge is built on a single conviction: **you cannot manage coverage you cannot see**.

The process is:

1. **Model the system** — every capability, dependency, constraint, and integration context in `system.json`
2. **Express coverage as traversal paths** — structured, parseable, constraint-validated
3. **Weight risks** — not all coverage contributes equally; the model knows which branches matter most
4. **Generate and optimize campaigns** — minimum paths, maximum structural and parametric coverage
5. **Track coverage over time** — diffable campaigns, versioned confidence scores, explicit gap history

The result is a QA system where coverage is a property of the **model**, not the **team**. It survives staff rotation. It bounds AI generation. It produces an audit trail. And it scales with the system — because as new features are added to `system.json`, the coverage gap is immediately visible in the next campaign diff.

> Quality is encoded in traversal — not lost in test cases.

---

*Built with PathForge · Root-to-Canopy Quality Mapping Methodology*
