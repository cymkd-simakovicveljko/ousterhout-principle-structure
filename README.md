# Ousterhout Principle: Structure

An agent skill for evidence-backed architectural reviews. Find the complexity callers are forced to manage, put knowledge under the right owner, and recommend the smallest useful change.

**Make callers know less—not every file smaller.**
Language- and framework-independent. Apply it to a library, application or subsystem; the review follows that project's requirements and conventions.


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
/ousterhout-principle-structure src/documents review
```

| Mode | Behavior |
| --- | --- |
| `review` (default) | Read code and report findings in chat. No edits or generated reports unless requested. |
| `proposal` | Write a requested handoff with evidence, priorities, safety constraints and completion criteria. Specify a destination or use the project's convention. |
| `implement` | Implement an explicitly assigned slice. A broad architectural review is not permission for a rewrite. |

Examples:

```text
/ousterhout-principle-structure src/documents proposal — write docs/document-structure-proposal.md
/ousterhout-principle-structure src/documents implement — move shared document-loading logic below the CLI and HTTP adapters; preserve validation and error behavior
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

> **Verdict:** document loading is deep; shared application logic has the wrong owner.
>
> **Keep:** `load_document(path)` hides format detection, decoding and validation behind one result. Its implementation size is not a reason to split it. Keep the HTTP handler that enforces authentication and translates errors.
>
> **Change:** the CLI imports document-loading logic from the HTTP handler. Both entrypoints need that behavior, so put it below both callers. Keep HTTP authentication and response translation in the handler.
>
> **Order:** fix ownership first. No generic document framework or package-wide move.
>
> **Evidence:** supplied example only. Before changing real code, trace both callers and verify that valid and malformed documents retain their expected behavior and HTTP access checks remain enforced.

```text
Before: CLI → HTTP handler's document-loading implementation
After:  CLI → shared document-loading implementation ← HTTP handler
```

## When not to use it

- A flat, obvious pipeline with little hidden machinery.
- A tiny local fix with no interface or ownership consequence.
- Premature abstraction or unstable requirements: establish the need before deepening the abstraction.

This is a structural design review, not a substitute for correctness, security or performance verification. It follows the target repository's rules and does not authorize production mutations or destructive actions.

## Inspiration and license

The guiding ideas—deep modules, information hiding and reducing complexity—come from John Ousterhout's [*A Philosophy of Software Design*](https://web.stanford.edu/~ouster/cgi-bin/book.php). The skill turns those ideas into a practical caller-tracing and review procedure; it is not a reproduction of the book or a claim of official affiliation.

Licensed under [MIT](LICENSE).
