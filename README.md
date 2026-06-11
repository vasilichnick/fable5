# fable5

A Claude Code skill that makes Claude interview **you** before it builds.

Vague request in → Claude recons your repo first, asks only the questions
it genuinely can't answer itself (as clickable option cards, not essays),
plays back a one-block spec, then executes at an effort level it picks on
its own. You never touch a setting.

Built from the patterns in Anthropic's official
[Claude Fable 5 prompting guide](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/prompting-claude-fable-5),
whose core thesis is: this model converts *well-specified tasks given up
front* into first-shot correct results. Most people don't write
well-specified tasks. This skill inverts the burden — the model extracts
the spec from you.

## What it does

1. **Recon before questions.** Reads the repo, memory, and conversation
   first. It is forbidden from asking anything a `grep` could answer.
2. **One batched interview.** Up to 4 questions in a single interactive
   card: goal & why, what "done" means, boundaries, and taste forks (with
   side-by-side previews for visual/copy choices).
3. **Spec playback.** One compact block — Goal / Done means / Boundaries /
   Capacity — so you catch misunderstandings before any code exists.
4. **Auto-capacity.** The skill grades the task's stakes and blast radius
   and picks its own firepower:

   | Capacity | When | What you get |
   | --- | --- | --- |
   | quick | cosmetic, reversible | single pass + lint |
   | high | normal feature work | build + lint/types/build, screenshots for UI |
   | xhigh | multi-file / user-critical | + fresh-context verifier subagent, edge sweep, audits |
   | max | production, money, irreversible | + adversarial multi-agent verification, evidence-cited report |

   You never choose a level (most users shouldn't have to know they
   exist) — but if you name one in your message, your word wins.
5. **Evidence-grounded report.** Every claim in the final summary must
   point at a tool result: test output, screenshot, diff. Unverified work
   is reported as unverified.

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
/fable5 i want users to see their reward share somehow
```

Or just ask for something substantial and vague — the skill auto-fires
when a build request arrives underspecified, recons, asks the gaps, and
goes.

Works in any Claude Code session; the capacity ladder is tuned for the
Fable 5 / Opus model family but degrades gracefully on anything.

## Tools it drives

The frontmatter deliberately ships **without** an `allowed-tools`
allowlist — in Claude Code that field *restricts* rather than documents,
and Phase 4 executes whatever your build legitimately needs. What each
phase actually reaches for:

| Phase | Tools |
| --- | --- |
| 1 — Recon | `Read`, `Glob`, `Grep`, `Bash` (read-only state checks) |
| 2 — Interview | `AskUserQuestion` (the interactive option cards) |
| 3 — Spec + capacity | none — pure judgment |
| 4 — Execute | your session's normal toolset (`Edit`/`Write`/`Bash`/…); plus `Agent` (fresh-context verifier subagent) at xhigh, and `Workflow` (multi-agent adversarial verification) at max |
| 5 — Report | cites the tool results produced above |

Everything named is a Claude Code built-in: nothing to install, no MCP
servers, no network calls of the skill's own. The Workflow tier only ever
activates at `max` capacity (production/money/irreversible stakes) — and
per Claude Code's rules, a skill instructing it counts as your explicit
opt-in to multi-agent token spend, so it's also documented here.

## Why "fable5"

Named after the model whose prompting guide it encodes. The guide tells
*you* how to prompt the model well; this skill makes the model hold up
its end of that contract instead.

## License

MIT. See [`LICENSE`](LICENSE).
