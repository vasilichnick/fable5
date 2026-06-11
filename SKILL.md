---
name: fable5
description: |
  Interview-to-spec protocol for non-trivial build requests: recon the
  repo/memory first, ask the user only what recon cannot answer (batched
  AskUserQuestion with previews), play back one compact spec, then execute
  at a capacity (quick/high/xhigh/max) the skill selects and announces
  itself — the user never has to know the levels exist. Fires on /fable5
  or when a substantial build/feature/change request arrives underspecified.
when_to_use: |
  The user asks for something non-trivial to be built or changed and the
  request leaves goal, done-criteria, or scope genuinely open. Also on
  explicit /fable5. NOT for trivial edits, fully-specified asks, or
  questions/assessments (answer those directly).
argument-hint: <your rough request>
---

# fable5 — interview to spec, execute at auto-capacity

## Contract

Calling this skill (or having it auto-fire on an underspecified build
request) guarantees:

- Recon happens before any question: repo, memory, and conversation are
  checked first, and nothing answerable from them is ever asked.
- At most two AskUserQuestion rounds, covering only what the user alone
  knows: goal and why, what "done" looks like, boundaries, and taste forks
  (with previews when the choice is visual or copy).
- One compact spec playback (Goal / Why / Done means / Boundaries /
  Capacity) before execution — corrected at most once, never looped.
- Capacity (quick / high / xhigh / max) is selected by the skill from the
  task's stakes and blast radius, announced with a one-line rationale, and
  never asked of the user. An unprompted user override always wins.
- Execution then runs against the spec, and the final report ties every
  claim to tool evidence.

The capacity levels mirror the model's effort semantics (the Claude
Fable 5 prompting guide's low→max scale) but are implemented behaviorally:
verification depth, subagent firepower, and workflow use — not the API
parameter, which the harness owns.

This skill is fat by THFS contract (Thin Harness, Fat Skills: judgment
lives in markdown, deterministic checks live in code, the harness stays
thin). It owns the protocol — interview → spec → capacity → evidence;
the craft of building stays latent. If you also run an always-on
guardrail skill (e.g. karpathy-guidelines: surgical diffs, no speculative
abstractions), it applies underneath; the anti-patterns below carry the
essentials either way.

## Trigger

- "/fable5 <request>"
- "interview me about this", "help me spec this"
- A non-trivial build/feature/change request arrives with goal,
  done-criteria, or scope genuinely open — and recon can't close the gap.
- Does NOT fire: trivial edits, fully-specified requests (skip straight to
  work), or the user thinking out loud / asking a question (the
  deliverable there is an assessment, not an interview).

## Phases

### Phase 1: Recon before questions

Read the request, then close every closable unknown yourself: `Grep`/
`Glob`/`Read` the repo, check memory and the conversation, `Bash` for
state (branch, running services) when relevant. Output: a short private
list of unknowns that are genuinely user-only — intent, audience,
done-criteria, boundaries, taste. If that list is empty and the task is
small, say so in one line and jump to Phase 4 (capacity quick or high).

### Phase 2: Interview — only the user-only unknowns

One `AskUserQuestion` call with 2–4 batched questions (a second call only
if the first answers genuinely fork the design). Question kit, derived
from the Claude Fable 5 prompting guide:

1. **Goal & why** — "give the reason, not only the request": who is it
   for, what does it enable.
2. **Done means** — a verifiable bar, not "make it good".
3. **Boundaries** — what must NOT change; what's explicitly out of scope.
4. **Taste forks** — concrete options with `preview` blocks when the
   choice is visual/copy; first option marked "(Recommended)".

Never ask about capacity, tooling, or anything recon could have answered.

### Phase 3: Spec playback + capacity selection

Compose and show the spec block (format below). Select capacity by latent
judgment of stakes × blast radius:

| Capacity | When | What execution gets |
| --- | --- | --- |
| quick | cosmetic / single-file / easily reversible | single pass + lint |
| high (default) | normal feature work in known territory | build + deterministic verify (lint, types, build; screenshot if UI) |
| xhigh | multi-file, architectural, or user-facing-critical | + fresh-context verifier subagent (`Agent`), edge-case sweep, relevant audits |
| max | irreversible, production-critical, money/keys/data paths, or the user signals high stakes | + adversarial multi-agent verification via `Workflow` (find → refute panel), evidence-cited report |

Escalate one level when production, money, credentials, or data loss are
in play; de-escalate when the change is dev-only and reversible. If the
user named a capacity anywhere in their message, that wins — but the user
is never asked to choose.

Gate rule: at quick/high, play the spec back and proceed in the same
turn. At xhigh/max, end the turn after the playback for one confirmation
— big work earns one checkpoint, small work earns none.

### Phase 4: Execute at capacity

Run the work per the capacity row. Deterministic checks (lint, build,
tests, screenshots) go through `Bash`; latent review at xhigh+ goes to a
fresh-context subagent with the spec as its brief, not the build
transcript. Keep diffs surgical: change only what the spec asks; no
speculative abstractions, no drive-by refactors.

### Phase 5: Evidence-grounded report

Lead with the outcome. Audit each claim against a tool result from this
session before stating it — unverified work is reported as unverified.
Include: what was deliberately not done, and one line on the capacity
used ("ran at xhigh — verifier caught N issues" / "quick was enough").

## Quality gates

- Zero questions asked that recon could have answered (every question maps
  to a named user-only unknown from Phase 1).
- ≤2 AskUserQuestion calls per invocation, ≤4 questions each.
- A spec block was shown before execution (or an explicit one-line skip
  for trivial cases).
- Capacity appears in the spec with a one-line rationale — and no question
  to the user about it.
- Every claim in the final report points at tool evidence (test output,
  screenshot, diff, curl).

## Anti-Patterns

- ❌ Asking the user to pick quick/high/xhigh/max — capacity selection is
  this skill's job; only an unprompted override is honored.
- ❌ Interviewing a fully-specified request — that's stalling dressed as
  rigor; go build.
- ❌ Questions a `Grep` would answer ("which file holds the header?").
- ❌ More than one playback/confirm loop — one spec, one correction max,
  then act.
- ❌ Reaching for `Workflow` on a quick/high task because it feels
  thorough — max is for stakes, not vibes.
- ❌ Encoding build craft (how to write the code) into this file — the
  skill owns the protocol; over-prescription degrades Fable 5 output.

## Output format

Interview turns: AskUserQuestion modal(s) only — no essay around them.

Spec playback:

```
**Spec — <short title>**
Goal: <what & why, one line>
Done means: <verifiable bar>
Boundaries: <untouchables / out of scope>
Capacity: <level> — <one-line rationale>
```

Final report: outcome first (one sentence a TLDR-reader needs), then
evidence-linked detail, then "deliberately not done", then the capacity
line. No arrow-chain shorthand; complete sentences.
