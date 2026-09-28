---
name: ousterhout-principle-structure
description: Assess module depth, information hiding and structural complexity; recommend the smallest evidence-backed improvement.
disable-model-invocation: true
license: MIT
argument-hint: "[path or subsystem] [review | proposal | implement]"
---

# Ousterhout principle: structure

> The best modules are deep. They allow a lot of functionality to be accessed through a simple interface.
> — John Ousterhout

Make callers know less. Do not optimize for smaller files or more layers.

## Invocation and scope

Default to a **read-only review** of the named path or subsystem. If no scope is supplied, use the current conversation's explicit architectural target; if none exists, ask which area to review. Resolve the actual code path from the repository rather than assuming the user's spelling is exact.

- **review:** inspect and report in chat; no edits, report files or generated diagrams unless requested.
- **proposal:** write a handoff only when requested, using the requested destination or existing repository convention. Preserve findings, evidence, uncertainties, priorities, safety constraints and completion criteria. This mode does not authorize implementation.
- **implement:** change only the explicitly assigned behavior or structural slice. A broad review invocation is not permission for a package-wide refactor. If no slice is assigned, first deliver a bounded recommendation.

Follow the repository's instructions, documented requirements, design decisions and actual framework constraints. Distinguish current requirements, proposed changes and historical evidence. Inspect local changes before modifying files; preserve unrelated work.

## Vocabulary

Use these terms consistently when explaining a finding:

- **Module:** a function, class or cohesive package with an interface and an implementation.
- **Interface:** the contract and knowledge required to use a module correctly, not just its method signatures.
- **Implementation:** the mechanisms behind that contract.
- **Depth:** how much useful behavior a module provides relative to what callers must understand.
- **Seam:** a place where an implementation can vary without rewriting its callers.
- **Adapter:** code connecting a caller or external protocol to that seam.
- **Leverage:** difficult behavior implemented once and reused by real callers.
- **Locality:** related policy, changes and verification concentrated under one owner.

## Avoid When

- The task is a flat, obvious tool or pipeline with little hidden machinery; preserve its direct flow.
- The task is a tiny local fix with no interface or ownership consequence; make the local fix.
- The real problem is premature abstraction or unstable requirements rather than interface depth; establish the need before deepening the abstraction.

## Guiding test

Can a caller ask for a meaningful outcome without understanding how the module achieves it?

The interface includes more than signatures: ordering, invariants, failure modes, configuration, provenance, retry obligations and performance assumptions are also caller knowledge.

| Deep | Shallow |
| --- | --- |
| Substantial behavior behind a small interface | Interface nearly as complex as the behavior |
| Owns difficult knowledge | Makes callers coordinate difficult knowledge |
| Changes remain local | One policy change spreads across callers |

A large cohesive implementation can be deep. A tiny adapter can be necessary. Internal helper seams do not all belong in the external interface.

## Review procedure

### 1. Map responsibilities and constraints

Read relevant design documentation where available and map the actual code. Identify real entrypoints, outcomes, state owners, integrations and recovery paths where applicable. Separate essential complexity from accidental structure.

Essential complexity can include numerical precision, authorization, privacy, concurrency, data integrity, resource lifetimes, cancellation and uncertain external effects. Hiding it means owning it inside the right module, not deleting its safeguards.

Complete when the scope, governing behavior and representative entrypoints are known. Do not scan unrelated subsystems just to enlarge the report.

### 2. Trace behavior from caller to outcome

Trace representative procedures through real callers, including a failure/recovery path when relevant. Use language-server references where available. Inspect bodies, not only filenames, imports or declarations.

For each candidate module, establish:

- What outcome the caller requests.
- What the caller must assemble or know first.
- What behavior and policy the implementation actually hides.
- Which other callers benefit and where the same knowledge is repeated.
- What happens on retry, interruption, stale state or partial success.

Complete when every proposed finding has a concrete caller and implementation path. A component that supports a capability is not proof its callers use it.

### 3. Diagnose depth and ownership

Look for evidence of:

- **Information leakage:** callers share representation, positional encoding, selection rules or sequencing knowledge.
- **Misplaced ownership:** shared authentication under one feature, common validation under one unrelated operation, or reusable logic inside one entrypoint.
- **Inverted adapter dependencies:** reusable behavior imports its network, background-job or command-line adapter rather than both entrypoints using the underlying implementation.
- **Change amplification:** a writer and reader must change together for one policy or representation adjustment.
- **Discoverability (unknown unknowns):** can a maintainer identify which modules a change affects, or are important dependencies and side effects hidden? Hide implementation details—not consequences callers need to understand.
- **Pass-through interfaces:** extra configuration or wrappers add concepts without hiding meaningful behavior.
- **Fragmented cohesion:** one capability is scattered across technical file categories with no clear owner.
- **Integration gaps:** a deep implementation exists but the actual caller omits required context or never reaches it.

Challenge each suspicion before reporting it:

- Does the split preserve a transaction, durable checkpoint, independent recovery, authorization check or protocol requirement?
- Is a repeated check necessary revalidation after mutable state changed?
- Is private sharing local to one cohesive module, or crossing unrelated responsibilities?
- Would deleting the module remove ceremony, or push complexity back into callers?
- Is the symptom demonstrated, inferred from source, or merely hypothetical?

File counts, line counts, import counts and private names are navigation signals, not architecture scores. Do not label every small file shallow or every large file a god module. Report strengths worth preserving alongside weaknesses; no quota of findings.

### 4. Choose the smallest useful intervention

Use this order: no change if the module earns its keep; reuse an existing owner; move misplaced shared behavior; consolidate repeated knowledge; narrow caller inputs; only then introduce a new module where a real seam needs an owner.

For each recommendation specify:

1. Caller knowledge before and after.
2. The single owner of the hidden policy or representation.
3. Actual affected callers and the change in dependency direction.
4. Preserved invariants and compatibility risks for existing callers, saved data, protocols or execution histories, where applicable.
5. The smallest check that would detect a broken contract.

Prefer boring changes. Avoid generic orchestration frameworks, speculative repositories, universal facade modules and folder-only rearrangements. Preserve meaningful type distinctions and explicit runtime validation. Sharing a mechanism does not mean its callers share the same policy.

When a behavioral gap depends on an unspecified requirement, name the missing decision rather than inventing it. Continue independent structural work only if it is within the assigned scope.

Complete when each recommendation explains a concrete reduction in caller burden and a safe, bounded way to prove it.

## Output

Lead with an honest verdict. Use this structure, sized to the scope:

1. **Verdict:** where depth exists and where complexity escapes.
2. **Keep:** concrete deep modules and why they earn their interfaces.
3. **Change:** ranked findings with source path/symbol/line, actual caller evidence, consequence and minimum remedy.
4. **Order:** first useful change, what to defer, and explicit non-goals.
5. **Evidence:** inspection/commands/scenarios actually performed and their limits.

Label source-derived behavioral uncertainty **[INFERENCE]**. Do not invent numeric quality scores, runtime results or production-readiness claims. Prefer code references and a small before/after dependency sketch over generic architecture advice. Recommend no change when the evidence supports it.

For a requested proposal, also include the reviewed revision and local-change caveats, relevant references, requirements versus proposals, goals, non-goals, compatibility requirements, per-slice completion criteria and a copyable next-agent assignment. Link existing policy rather than cloning an entire repository instruction set. A handoff records an executable next step, not implicit approval of every suggested change.

## Implementation and verification

When implementation is explicitly assigned, trace all affected callers before editing; use symbol-aware refactoring where available. Migrate internal callers cleanly and remove obsolete glue. Where applicable, preserve saved data, external contracts and durable execution histories with an explicit compatibility decision rather than treating them as internal imports.

Prove behavior through the caller's interface, including affected error and recovery paths where applicable. Keep focused regression tests for plausible bugs; do not add tests that assert source layout, method forwarding or mock echoes. Run repository-required checks and report exact commands/results. For a read-only assessment, source evidence is sufficient for structural claims; use a bounded scenario before claiming a runtime defect. Use isolated resources for checks; a structural review does not authorize production mutations or destructive actions.

## Calibration examples

- `load_document(path)` handles format detection, decoding and validation behind one interface: preserve it even if its implementation is large.
- A short HTTP handler enforces authentication and translates errors: thinness alone does not justify removing it.
- A writer and reader independently interpret `[version, flags, payload, ...]`: give that format one owner and preserve compatibility with existing data.
- A CLI imports shared application logic from an HTTP handler: move the shared behavior below both entrypoints while preserving their distinct responsibilities.
- Many files but no demonstrated caller burden: do not propose a package-wide move. Find the actual knowledge leak first.

## Inspiration

Inspired by John Ousterhout’s *A Philosophy of Software Design*. This is an independent practical review workflow, not an official skill or an endorsement by John Ousterhout.
