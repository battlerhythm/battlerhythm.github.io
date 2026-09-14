---
title: "Practices"
permalink: /practices/
layout: single
author_profile: false
toc: false
---

What I hold myself to when I write or change code -- and what I hand to a coding agent before it starts.

**Scope is deliberately narrow.** Not engineering in general: the judgements that decide the shape of
code. Whether to add an abstraction. Where a boundary goes. What a comment has to carry. How a commit
is cut. These come up on a blank file as much as on a twenty-year-old one -- and the first gate bites
hardest at design time, when you have the least information and the strongest pull toward building
for a future you are only imagining. If a rule here does not fire at one of those moments, it does
not belong on this page: the list has to stay short enough that a person can review it and an agent
can actually hold it.

The rest of this site is the argument behind these; this page is the conclusions. Each rule links
back to the post that earned it.

## The rules

Gates on actions, not a program of work. Nothing here authorises refactoring -- if none of the
triggers fire, none of it applies.

**The output is a file where the next change is cheaper than the last one -- not a cleaner file.**

### Before adding any abstraction -- interface, base class, pattern, layer, new file

1. **Has the change arrived twice?** Once is a fact, twice is an axis, three times is a pattern.
   Abstraction built on one occurrence encodes a guess.
2. **Does a second implementation or a test double exist?** If not, it is cost with no benefit.
   An interface nobody substitutes at is decoration.
3. **Is the code shorter, or at least easier to follow, afterwards?** If it leaves more files and a
   flow that is harder to trace, it failed -- "it's the correct design" is not a justification.
   <https://battlerhythm.github.io/design_patterns/m0-fundamentals/>

Prefer, in this order, before reaching for a pattern:

| Instead of | Use |
|---|---|
| Strategy / Command / single-method interface | a function type (`(A) -> B`) |
| Visitor over a closed hierarchy | `sealed` + exhaustive `when` |
| Inheritance for reuse | interface + `by` delegation |
| Builder | named + default arguments, or `data class` + `copy()` |
| Singleton holding state | a value passed in; `object` only for stateless, I/O-free namespaces |
| Wrapper class for one behavior | an extension function |

<https://battlerhythm.github.io/design_patterns/>

### Before settling on a module boundary

**Can someone use this correctly from the signature, the name, and the interface comment alone?**
If they have to read the body to use it, the boundary is decorative whatever the diagram says.
Depth is functionality divided by interface: prefer the smaller interface over the larger one
covering the same work, and treat a class that is mostly pass-through methods as having none.
<https://battlerhythm.github.io/changing_code/m2-deep-modules/>

### Before changing code that has no tests

1. **State the change in one sentence of observable behavior.** If you cannot, stop -- there is no
   stopping condition and you are rewriting, not refactoring.
2. **List the dependencies on the path of this change and find the ones with no enabling point** --
   a place *outside this file* where you could choose different behavior. The usual offenders are
   clocks, random, environment, global objects, and constructors called inside method bodies.
3. **Check whether a seam already exists** before building one. Another test may have built it.
4. **Cheapest seam that works**, in order: default function-type parameter -> interface as a
   constructor parameter -> `internal` + test source set -> `expect`/`actual`.
   **Never make a class `open` to enable a test.** Test of a good seam: zero production call sites change.
5. **Pin current behavior before touching it.** Assert something you believe is false, run it, paste
   the real value in. Pin the boundary in both directions -- the return value *and* what was sent.
   Label behavior you believe is wrong with a date and a reason, so the pin is not read as a spec.
   <https://battlerhythm.github.io/changing_code/m6-one-page/>

### Before writing a comment, or removing one

**Is this information already somewhere in the code?** If yes, fix the code -- rename, extract -- and
delete the comment. If no, the comment is the only carrier and deleting it loses the information.
Prefer moving a claim into a type, a `require`/`check`, or a named test over stating it in prose.
Never delete an interface comment on the grounds that the implementation can be read.
<https://battlerhythm.github.io/changing_code/m2-comments/>

### Before committing

- **One hat per commit.** A commit either changes behavior or changes structure, never both.
  A `refactor` commit that modifies an existing assertion is a behavior change wearing the wrong label.
- **Pins update in the behavior commit**, never in a structural one -- that is what forces a behavior
  change to declare itself.
- Sequence when both are needed: `refactor` (seam, zero call-site changes) -> `test` (pins, passing
  against unchanged code) -> `fix`/`feat` (the change, with its pin).
- A structural commit's insertions and deletions should roughly balance. They do not when content changed.
  <https://battlerhythm.github.io/changing_code/m5-discipline/>

### Stop

- When the change you came for is **safe**, not when the file is **clean**.
- When the seam you needed **exists**, not when the class is **well-designed**.
- Do not sweep for coverage, and do not refactor code nobody is about to touch -- the benefit is only
  paid when someone next changes it, and they may never.
- Do not delete a comment, an interface, or a defensive check without finding out why it is there.
  Defensive code at a caller is a bug report written by someone who decided not to file one.

---

## Notes

### How an agent gets this

Three ways, in order of reliability.

**1. A file the agent always loads.** Put the rules where the table below says and they are in
context before you type anything -- nothing to remember, nothing to repeat, and they survive
`/compact` because loaded files are re-injected. This is the default.

**2. Paste the block.** When you cannot write to config -- someone else's machine, a web chat --
paste the rules as the first message. Works everywhere, needs no network, and you know it loaded.

**3. Tell the agent to fetch it.** *"Read battlerhythm.github.io before you start"* works when the
agent has a web tool and the domain is reachable: the home page points here. It is the fallback,
not the default. You repeat it every session, corporate egress may block a personal domain, and
instructions given in conversation fade as the context fills in a way a loaded file does not. Add
*"if you cannot fetch it, say so"* -- otherwise you will not know which case you are in.

**What does not work is a URL sitting in a config file.** Instruction files are loaded as text; a
link inside one is just a string. The agent would have to *decide* to fetch it, and it will not make
that decision at the moment it is about to add an interface. A gate that only fires when invoked is
not a gate.

At a company there is a further reason to paste rather than link, and it is not technical. A
personal domain referenced from a company repository makes your site's availability a dependency of
their tooling, and makes content you can change unilaterally into their engineering guidance.
**Put the text in a pull request instead**, where it gets reviewed like any other policy -- and check
first whether the team already has standards this would contradict, because contradictory rules
are worse than no rules.

### Where to put it, by scope

| Scope | Location |
|---|---|
| **Every project on your machine** | `~/.claude/rules/practices.md` -- applies to all projects, loads before project rules. Nothing to do when you start a new repo |
| Same, single file | `~/.claude/CLAUDE.md` |
| Same, Codex | `~/.codex/AGENTS.md` |
| Same, Cursor | User Rules in settings |
| **One repository / a team** | `AGENTS.md` at the root, committed |
| **A whole organisation** | Managed policy `CLAUDE.md` -- macOS `/Library/Application Support/ClaudeCode/CLAUDE.md`, Linux `/etc/claude-code/CLAUDE.md`. Deployed by IT, cannot be excluded by users |


The user-level row is the one that answers "what do I do when I start a new project": nothing.

Claude Code reads `CLAUDE.md`, not `AGENTS.md`. Keep one real file and point the other at it:

```bash
git mv CLAUDE.md AGENTS.md
ln -s AGENTS.md CLAUDE.md
```

On Windows a symlink needs Administrator or Developer Mode, so put `@AGENTS.md` as the first line of
`CLAUDE.md` instead -- the import loads at session start, and Claude-specific additions can follow it.

Two practical notes. User-level config is **machine-local**, so a new laptop means doing it again --
which is the argument for this page being the canonical copy. And a symlink in a project's
`.claude/rules/` pointing outside the working directory is treated as an external import and prompts
for approval; keeping shared rules in `~/.claude/rules/` avoids that entirely.

### Rules are context, not enforcement

Instruction files shape behavior; they do not guarantee it. Anthropic's own documentation is explicit
that Claude "treats them as context, not enforced configuration," and recommends a **hook** for
anything that must fire at a fixed point such as a commit.

Exactly one rule on this page is mechanically checkable, and it is the one worth enforcing: *a
`refactor` commit that removes or modifies an existing assertion is a behavior change wearing the
wrong label.* As a `commit-msg` hook:

```sh
#!/bin/sh
case "$(sed -n '1p' "$1")" in refactor*) ;; *) exit 0 ;; esac

TESTS='*Test.kt *Tests.kt *Test.java *_test.go *_test.py *test_*.py *.test.ts *.spec.ts *Tests.swift'

removed=$(git diff --cached -U0 -- $TESTS 2>/dev/null   | grep '^-' | grep -v '^---'   | grep -E '(assert|Assert|expect\(|XCTAssert|verify\()'   | cut -c1-100 | head -5)

[ -z "$removed" ] && exit 0

echo "" >&2
echo "  A 'refactor' commit removes or changes existing assertions:" >&2
printf '%s\n' "$removed" | sed 's/^/      /' >&2
echo "  If structure moved and behavior did not, no assertion had to move. Split the commit." >&2
echo "  Mechanical reason (rename sweep)?  git commit --no-verify" >&2
exit 1
```

Install with `cp commit-msg .git/hooks/commit-msg && chmod +x .git/hooks/commit-msg`. It passes a
refactor that only touches production code, one that only *adds* pins, and any commit not labelled
`refactor`; it stops the case it is meant to stop, across kotlin.test, JUnit, pytest, jest and XCTest
conventions.

### Keep project facts out of it

This block is general — it says nothing about any particular codebase. Build commands,
architecture, and invariants belong in the same file but in their own sections, and anything with a
date or a commit id belongs in a status document instead. Contradictory instructions are the main
failure mode of these files: when two rules conflict, the agent picks one arbitrarily.
