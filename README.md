# Ousterhout Principle: Structure

An agent skill for evidence-backed architectural reviews. Find the complexity callers are forced to manage, put knowledge under the right owner, and recommend the smallest useful change.

**Make callers know less—not every file smaller.**

Inspired by John Ousterhout's *A Philosophy of Software Design*. Independent project; not affiliated with or endorsed by John Ousterhout.

## Install

For a personal Claude Code installation:

```sh
git clone https://github.com/cymkd-simakovicveljko/ousterhout-principle-structure.git \
  ~/.claude/skills/ousterhout-principle-structure
```

If that destination already exists, keep it intact and choose another installation location or deliberately migrate it. Start a new session if the skill does not appear.

The skill is a self-contained [SKILL.md](SKILL.md): no scripts, packages, external skills or credentials are required by the skill itself. Reviewing a project still requires access to its code and the host agent's tools. For other agents supporting `SKILL.md`, install the directory in their documented skill location; invocation syntax and support for frontmatter settings may differ.

## Use

```text
/ousterhout-principle-structure src/payments review
```

| Mode | Behavior |
| --- | --- |
| `review` (default) | Read code and report findings in chat. No edits or generated reports unless requested. |
| `proposal` | Write a requested handoff with evidence, priorities, safety constraints and completion criteria. Specify a destination or use the project's convention. |
| `implement` | Implement an explicitly assigned slice. A broad architectural review is not permission for a rewrite. |

Examples:

```text
/ousterhout-principle-structure src/payments proposal — write docs/payment-structure-proposal.md
/ousterhout-principle-structure src/payments implement — move shared submission logic below the CLI and worker adapters; preserve claim-before-send and uncertain-outcome handling
```

The skill is manually invoked in hosts honoring `disable-model-invocation: true`.

## What it does

1. Maps responsibilities, real entrypoints and governing constraints.
2. Traces callers through implementation and relevant failure/recovery paths.
3. Checks information leakage, ownership, dependency direction, change amplification and discoverability.
4. Recommends the smallest intervention—or no change when the module earns its keep.

Reports follow **Verdict → Keep → Change → Order → Evidence**. Findings need source locations and actual caller evidence. Behavioral uncertainty is marked `[INFERENCE]`; source inspection is not presented as a passing runtime test.

A large parser can be deep. A tiny adapter can be necessary. File size, file count and underscore-prefixed imports are signals to investigate, not architecture scores.

## Sample review

Illustrative example, not a review of a real repository:

> **Verdict:** parsing is deep; submission logic has the wrong owner.
>
> **Keep:** `parse_document(bytes)` hides three formats behind one validated result. Its implementation size is not a reason to split it. Keep the worker adapter that supplies a durable checkpoint.
>
> **Change:** the synchronous submission module imports `_post` from its worker adapter. Both entrypoints need that behavior, so put it below both callers. Preserve claim-before-send and no-blind-resend rules.
>
> **Order:** fix ownership first. No generic submission framework or package-wide move.
>
> **Evidence:** supplied example only. Before changing real code, trace both callers and check that ambiguous submission outcomes cannot cause a second send.

```text
Before: synchronous caller → worker adapter's implementation
After:  synchronous caller → shared implementation ← worker adapter
```

## When not to use it

- A flat, obvious pipeline with little hidden machinery.
- A tiny local fix with no interface or ownership consequence.
- Premature abstraction or unstable requirements: establish the need before deepening the abstraction.

This is a structural design review, not a substitute for correctness, security or performance verification. It follows the target repository's rules and does not authorize live financial or destructive effects.

## Inspiration and license

The guiding ideas—deep modules, information hiding and reducing complexity—come from John Ousterhout's [*A Philosophy of Software Design*](https://web.stanford.edu/~ouster/cgi-bin/book.php). The skill turns those ideas into a practical caller-tracing and review procedure; it is not a reproduction of the book or a claim of official affiliation.

Licensed under [MIT](LICENSE).
