---
name: thermo-nuclear-code-quality-review
description: Extremely strict maintainability review of a diff, branch or set of PRs. Hunts for restructurings that delete complexity, files that grow past 1k lines, special-case branching, loose type boundaries, and tests that do not prove the change. Use for a thermonuclear review, deep code quality audit, or especially harsh maintainability review.
---

# Thermo-Nuclear Code Quality Review

Review `$ARGUMENTS` (a branch, PR number, commit range or "last N PRs to main";
default: the current branch's changes against main).

Behaviour must not change. Everything else is fair game. Start from the diff,
then follow it into the code it touches: the question is never "is this diff
tidy" but "is the codebase better or worse for it, and what would make it
dramatically simpler".

## What "better" means

A change makes the codebase better when a reader must hold FEWER concepts,
branches and layers in their head to follow it; when logic sits in the module
that already owns that concept; when a file can be found and understood by its
name alone; and when the next change is easier to make and easier to prove.
Anything that moves in the other direction is a regression, even if it works.

## The rules

1. **Look for the judo move first.** Before commenting on anything local, ask
   whether a different framing would make whole branches, flags, modes, helpers
   or layers disappear. Prefer the version that feels inevitable in hindsight.
   A refactor that moves complexity around without deleting any concept is not
   an improvement.

2. **Files stay under 1,000 lines.** A file is navigated by its name; a giant
   file must be read whole to find anything in it. If the change pushes a file
   over the line, the decomposition should have come first. Waive only with a
   stated structural reason and a file that is still clearly one thing.

3. **No special-case branches in shared paths.** A new `if` for one feature
   inside a general flow, a one-off boolean or nullable mode, an edge case
   handled in the middle of an already busy function: these are design
   problems, not nits. The remedy is to reframe the model so the case is the
   default flow, or to move it behind its own abstraction.

4. **Boring and direct beats clever.** Flag magic, generic mechanisms that hide
   a simple data shape, identity wrappers, and pass-through helpers that add
   a hop without adding clarity. Delete the layer rather than polish it.

5. **Tight boundaries.** Question every optional parameter, loose or escape
   hatch type (`any`, `unknown`, `Any`, `object`), cast, and silent fallback
   default. If the value is always present, it is required. If a branch exists
   only to paper over an unclear invariant, make the invariant explicit
   instead. A lookup that cannot answer should fail, not guess.

6. **Reuse the canonical helper.** Grep before accepting a new helper. A near
   duplicate of an existing utility, or logic placed in the wrong package when
   a clear home exists, is architectural drift.

7. **Errors are not swallowed.** A try block that returns false, an empty
   catch, a log-and-continue on a path that should stop: flag them as
   correctness problems, not style.

8. **Independent steps should not wait on each other, and multi-step writes
   should not stop halfway.** Two unrelated calls run one after the other for
   no reason: ask whether they should run together. A sequence of updates
   that can fail in the middle and leave state half-applied: ask for one
   atomic write. Do not chase micro-optimisations beyond this.

9. **Review the tests with the same standards.** For every meaningful change:
   - Name the seam it should have been tested at. If the test bypasses that
     boundary and pokes internals, it is not proof of the boundary.
   - Flag tests that assert a property of their own fixture and never call
     the function that owns the behaviour.
   - Flag a change that made the code harder to test, or that left the next
     run with no fast way to check it. A codebase that is easy to change
     but hard to verify is not easy to change.
   - A missing test for a new branch is a finding, ranked with the branch.

## Output

Order findings by weight, and keep the list short: a few high-conviction
findings beat a long list of cosmetic notes.

1. Structural regressions (rules 2, 3, 4, 6)
2. Missed judo moves: a visible path that would delete whole categories of complexity (rule 1)
3. Boundary and type-contract problems (rule 5), swallowed errors (rule 7)
4. Test and seam problems (rule 9)
5. Orchestration and atomicity (rule 8)
6. Legibility

For each finding give: file and lines, what is wrong in one sentence, the
concrete remedy (delete X, move Y into Z, split file A into B and C, make P
required), and what it removes.

Sample phrasing, so the language stays sharp:
- "this pushes the file past 1k lines; decompose first."
- "this adds a special-case branch to a shared flow; reframe so the branch disappears."
- "this test asserts its own fixture; it never calls the function it claims to cover."

## Verdict

End with **APPROVE** or **REQUEST CHANGES**. Working code is not the bar.
Request changes when any of these is present and unjustified:

- a visible judo move that the change did not take
- a file pushed past 1,000 lines
- new special-case branching in a shared path
- an unnecessary wrapper, cast, optional or silent fallback
- a duplicated helper or logic in the wrong layer
- a change with no test at its seam, or a test that proves nothing

Say plainly if the codebase is messier than before the change.
