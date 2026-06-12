# fable5

A Claude Code skill that compiles your plain-words request into a prompt
engineered for the Claude Fable 5 model.

Messy prompt in — any language, numbered fragments, pasted snippets,
"like it was before" references — one English, copy-ready, runnable
prompt out, built the way Anthropic's official
[Claude Fable 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5)
says this model wants to be briefed: full task up front, the reason
behind it, testable done-criteria, explicit boundaries, and *only* the
behavioral clauses the task actually needs.

The guide's core thesis: Fable 5 converts *well-specified tasks given up
front* into first-shot correct results. Most people don't write
well-specified tasks. This skill writes them for you, from what you said.

## What it does

1. **Recon before questions.** Reads the repo, memory, and conversation
   first. Vague references get resolved into anchors — "as it was before
   your last deploy" becomes a commit hash, "remove this line" becomes a
   `file:line` with quoted text. It is forbidden from asking anything a
   `grep` could answer.
2. **One gap interview, only if needed.** Up to 4 questions in a single
   interactive card, each mapped to something only you know: the goal and
   who it's for, what "done" means, what must not change, genuine taste
   forks. Fully-specified input skips this entirely.
3. **Compiles the prompt** on a seven-part skeleton:

   | Part | What goes in |
   | --- | --- |
   | Context & why | "Working on X for Y; they need Z. With that in mind:" |
   | Task | your numbered requirements, made concrete and testable |
   | Done means | checks a tool can run — commands, measurements, viewports |
   | Boundaries | what must NOT change + no-speculative-work clause |
   | Process clauses | **only the rows your task triggers** (see below) |
   | Reporting | outcome-first summary style, for long runs only |
   | Operator footer | a recommended effort level, for you, not the model |

   The clause library maps task signals to the guide's patterns —
   ambiguity → act-when-enough-info; code edits → surgical changes; long
   runs → evidence-grounded progress; autonomy → checkpoint rules; UI →
   viewport-matrix verification. Dumping all of them into every prompt is
   an anti-pattern: the guide is explicit that over-prescription degrades
   Fable 5 output, so clause *selection* is the skill's actual judgment.
4. **Hands off with one question.** The prompt prints in a copyable
   block with its assumptions listed, then: run it now in this session,
   or keep the prompt. Nothing executes before you answer.

Hard guarantees: every fragment of your input survives into the output
(your "MUST" stays a MUST), the produced prompt never asks the model to
echo its internal reasoning (a Fable 5 refusal trigger), and the output
is always English regardless of input language.

## Example

Raw input:

```
dev2: 1. all pages markup must be (as it was before your last deploy)
a viewport-locked layout; 2. /home: next launch card - Bosnia &
Herzegovina MUST be 2 lines on all screens; 3. /markets/champion - top
part incl tabs MUST be locked and never scroll, table scrolls inside...
```

Compiled output (abridged):

```
Context: Sponsio — World Cup 2026 belief-coin market (Next.js 16),
dev2 worktree, dev server on localhost:3001. Restoring the signature
viewport-locked layout across all pages before the next prod merge...

Task:
1. Every (site)/ page uses the viewport-locked layout (body never
   scrolls; inner panels scroll) as on main at commit <hash>...
2. /home next-launch card: "Bosnia & Herzegovina" renders as exactly
   two lines at 390, 768, 1280 and 1440 px widths.
...

Done means: body scrollHeight equals clientHeight on every page at
1440×900 and 390×844 while the board panel scrolls internally;
screenshots at four named viewports; tsc and production build green.

Boundaries: don't touch main/prod, tokens.json, /mini. Don't add
features or abstractions beyond what the task requires.

Process: when you have enough information to act, act. Verify against
the checks above before claiming done; report outcome first.

[operator: run at effort xhigh]
```

## Install

The repo is the skill directory (standard Claude Code convention). Clone
and symlink into your personal skills folder:

```bash
git clone https://github.com/vasilichnick/fable5.git ~/dev/fable5
ln -sfn ~/dev/fable5 ~/.claude/skills/fable5
```

Or clone straight in:

```bash
git clone https://github.com/vasilichnick/fable5.git ~/.claude/skills/fable5
```

## Use

Explicitly:

```
/fable5 i want users to see their reward share somehow, and the table
should not jump on mobile
```

Or just send a substantial-but-vague build request — the skill
auto-fires when goal, done-criteria, or scope are genuinely open,
compiles the prompt, and asks before running anything.

## Tools it drives

The compile stage is read-only by design, so v2 ships an
`allowed-tools` grant for exactly what it touches:

| Phase | Tools |
| --- | --- |
| 1 — Ingest & recon | `Read`, `Glob`, `Grep`, `Bash` (read-only state: branch, git log, exact line text) |
| 2 — Gap interview | `AskUserQuestion` (one card, ≤4 questions) |
| 3 — Compile | none — pure judgment against the guide's patterns |
| 4 — Handoff | `AskUserQuestion` (run now / keep); on "run now" the produced prompt executes with your session's normal toolset |

No MCP servers, no network calls of the skill's own, nothing written to
disk — the deliverable is text in your chat.

## Why "fable5"

Named after the model whose prompting guide it encodes. The guide tells
you how to brief the model well; this skill does the briefing.

v1 of this skill (interview → spec → execute at auto-selected capacity)
lives one commit back in history if you want it.

## License

MIT. See [`LICENSE`](LICENSE).
