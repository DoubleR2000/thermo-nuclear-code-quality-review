# Thermo-Nuclear Code Quality Review (revised)

An extremely strict, ambitious code-review skill for Claude Code, Cursor and
any agent that reads `SKILL.md` files. Point it at a diff, a branch, a PR or
"the last N PRs to main" and it looks past the diff into the code it touches,
hunting for restructurings that delete whole categories of complexity.

## What it does

The reviewer is told that behaviour must not change and everything else is
fair game. It works through nine rules, in order of weight:

1. **Look for the judo move first.** A reframing that makes branches, flags,
   modes or layers disappear beats any local tidy-up.
2. **Files stay under 1,000 lines.** Big files cannot be navigated by name and
   must be read whole; decompose before growing them.
3. **No special-case branches in shared paths.** A feature `if` in a general
   flow is a design problem, not a nit.
4. **Boring and direct beats clever.** Identity wrappers and magic get deleted,
   not polished.
5. **Tight boundaries.** Unnecessary optionals, escape-hatch types, casts and
   silent fallbacks are questioned. A lookup that cannot answer fails, never
   guesses.
6. **Reuse the canonical helper.** Grep before accepting a new one.
7. **Errors are not swallowed.**
8. **Independent steps should not wait on each other; multi-step writes should
   not stop halfway.**
9. **Review the tests with the same standards.** Name the seam the change
   should be tested at, flag tests that only assert their own fixture, flag
   changes that made the next run harder to verify.

It then reports a short ranked list of findings, each with file, lines, a
one-sentence defect, a concrete remedy and what that remedy removes, and ends
with **APPROVE** or **REQUEST CHANGES**. Working code is not the bar.

## Install

### As a plugin

This repository is also a plugin marketplace. In Claude Code:

```
/plugin marketplace add DoubleR2000/thermo-nuclear-code-quality-review
/plugin install thermo-nuclear-code-quality-review@thermo-nuclear
```

Plugin skills are prefixed with the plugin name, so in a session:

```
/thermo-nuclear-code-quality-review:thermo-nuclear-code-quality-review last 5 PRs to main
```

To get it on every machine at once, add the same repository on claude.ai under
**Customize > Plugins > Add > Add marketplace** and add the plugin there. It is
then saved to your account, and Claude Code downloads it on any machine where
you are signed in with that account.

### As a plain skill

Copy `SKILL.md` into a folder named `thermo-nuclear-code-quality-review` under
your skills directory. For Claude Code, globally:

```
~/.claude/skills/thermo-nuclear-code-quality-review/SKILL.md
```

Then in a session:

```
/thermo-nuclear-code-quality-review last 5 PRs to main
```

Either way, the skill is marked `disable-model-invocation: true`, so it only
runs when you invoke it.

## Credits

- **Original skill: the Cursor team.** This is a derivative of
  [`thermo-nuclear-code-quality-review`](https://github.com/cursor/plugins/blob/3347cbab5b54136f6fba0994c3a01a56f7fb7fca/cursor-team-kit/skills/thermo-nuclear-code-quality-review/SKILL.md)
  in the `cursor/plugins` repository. The ambition, the "code judo" framing,
  the 1,000-line rule, the boundary rules, the ranked output and the approval
  bar are all theirs.
- **Review and suggestions: Matt Pocock.** In a video review of the original
  he ran it on his own codebase (Sandcastle) and rated its findings, then
  suggested the changes this version applies:
  - the original repeats each idea four to six times across its sections;
    collapse it so it is much shorter and the agent can prioritise;
  - "unnecessary sequential orchestration and non-atomic updates" is word
    salad; say it plainly;
  - "does this improve or worsen the local architecture" is meaningless
    unless good and bad are defined; define them;
  - the tone section does not belong in a skill;
  - keep the ambition, the 1k-line rule (files should be navigable by name),
    the questioning of unnecessary optionality, the reuse of existing helpers,
    the ranked output and the approve/reject verdict;
  - the original says nothing about tests, seams or the feedback loop, which
    in his view is the entire point of a maintainable codebase; add it;
  - his run turned up swallowed errors that the original only found by
    accident; name them explicitly.
- **This revision: Riccardo Ronco, with Claude.** Applied the points above,
  added the test-and-seam rule, and generalised the type rule beyond
  TypeScript so it applies to Python and other languages.

## Licence

This revision is released under the MIT licence (see `LICENSE`). The original
`cursor/plugins` repository carried no licence file at the commit referenced
above; the original text is reproduced here in adapted form with attribution,
and the Cursor team retain whatever rights they hold in it.
