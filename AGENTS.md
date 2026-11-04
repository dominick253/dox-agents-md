# <repo> contract (DOX root of this tree)

Chain: none - this IS the root of the chain; every other contract in the tree points here.

Scope: what an agent must know before its first tool call, plus the routing index to everything else. Detail lives in opt-in children, never here.

## Grammar (host-wide, every contract in the tree uses it)

-> leads-to/imply <- reads-from => must ~ approx + and | or ! hard rule/pitfall STALE=unverified OPEN=owner decision pending ? unknown [date]=verified YYYY-MM-DD

Style: telegraphic, articles/copulas/filler dropped, no tables, no em dash, no human-facing prose. Commands, paths, SHAs, ports, test names, env names, dates byte-exact. Credential rule: reference the LOCATION, never the value.

## Protocol

- Hierarchy: nearest AGENTS.md above cwd wins on conflict. A contract opens `# <dir> contract`, then a `Chain:` line naming its parent, then `Load:` if it is a child.
- Ownership = deepest need: a fact lives in the deepest contract where an agent actually uses it. A parent keeps one pointer line per offload (essence + child path); section too big => move it to the owning child in the same edit; a child must never contradict its parent line (fix both, one pass).
- Children are opt-in: every offload child opens with a `Load:` trigger naming the task shapes that justify reading it => read only the child whose trigger matches, never the whole sibling set. A large topic splits into topic children behind a small index child that keeps the parent's filename. On-disk total may grow; per-read tokens must fall.
- Budgets (bytes, hard): root 17920 | hub 14336 | contract 8192 | index child 3300 | topic child 4096 target, 8192 hard. Overflow => offload to a child, never truncate, never drop a load-bearing fact.
- Maintenance: meaningful change => update the owning contract in place; essence changed => update the parent pointer line; new contract => register it in the map the same day.
- Live state > snapshot: report absent/stale only after querying the thing itself (`git ls-remote`, fetch the URL, check the running listener). A local tracking ref being absent does not mean the data is absent.
- ! Never delete a load-bearing fact: move it, and leave the pointer that reaches it.

## Verification (Red/Green, and its only exception)

- Behavior change => failing test FIRST: run it, paste the actual failing output (test name + command + failure line) into working evidence BEFORE implementing; implement; re-run the same command; paste the passing output. RED->GREEN in the same transcript and the same commit message.
- A non-vacuity mutation must be verified present (grep -c, byte count) before citing a run. A "paraphrase" typo that leaves a file byte-identical makes the passing test prove nothing.
- EXCEPTION, fixed greppable labels, when no automated failing test is meaningful (docs-only, config/prompt text, infra/service posture, no-behavior refactor, record-only): record `EVIDENCE-BEFORE:` (commands run before + measured output), `EVIDENCE-AFTER:` (same commands after), one line on what changed and why no test applies, and name the closest covering test, or `NO AUTOMATED COVERAGE POSSIBLE: <reason>`. The exception never skips verification.

## Retention proof (any edit that moves or compresses contract text)

- `cp` every file you will touch into a working dir at the START of the same task, byte-exact. The proof may only diff against that snapshot: never against a reconstruction, a memory, or a token list. A reconstruction can only prove the facts its author remembered.
- Extract identifiers from before and after (SHAs, IPs, ports, dates, versions, filenames, commands), then `comm -23 before after`. Dropped set must be EMPTY. Content compression that changes wording needs a second screen (content-word parity, modal-verb parity for rules).
- A ref that moves to a child must stay reachable: the parent keeps one pointer line, the child keeps a `Load:` trigger that covers the question. Absolute literal paths; brace shorthand (`context/{a,b}.md`) is banned - it is non-greppable and invisible to a resolver.
- Offload failures are classified, never averaged: MOVED-TO-CHILD (provable by quoting the parent pointer line + the child `Load:` line) vs LOST (zero tolerance).

## Cost model (what "better" means)

Report `E% @ C bytes / $` per variant: effectiveness of the contract against the agent's real context cost.
- An agent loading a directory gets its ancestor `AGENTS.md` chain automatically; `context/*.md` children are read only when a `Load:` trigger matches. So hub bytes are paid once per working directory: one byte in a root contract is N bytes of aggregate cost.
- Judge an offload on tools-on effectiveness + measured per-read bytes. NEVER on recall-mode effectiveness: a tool-less run cannot open a child, so recall falls for every offload no matter how good the routing is.
- Measure injected bytes from the transcript the runtime writes, not from provider token accounting (which under-reports and moves in the wrong direction here).
- Accept a shrink iff `dE >= -2 pts` AND `new fatal == 0` AND bytes down. Ratchet: never merge a change that creates a new fatal failure, even if the aggregate score rises.
- Probes: `recall` (non-guessable fact: port, SHA, test name, rule), `trap` (the expensive mistake, correct answer is a refusal), `route` (fact lives in a child: correct = UNKNOWN + the right path), `plan` (named commands/paths). Anchor every probe to the contract line that supplies the answer and grep-verify it; a probe whose prompt contains its own answer token tests recognition, not knowledge. Grade on fact tokens, never on citation format - a model with nothing loaded will fabricate a confident `file:line` citation.
- Controls: a blind arm from a contractless cwd (any probe the blind arm passes is non-discriminating - rewrite or quarantine it), repeated interleaved runs with paired per-probe comparison, and the same bank run against BOTH states in one session.

## Map

- `context/build.md` - build/test commands, artifacts, flake handling. Load: before running anything.
- `context/release.md` - commit/CI/push gates, receipts, failure ladder. Load: before the first push of a session.
- `context/domain.md` - invariants, canonical tests, what not to edit. Load: before editing domain logic.

This repo ships only `AGENTS.md` + `README.md`; the children above are illustrative of the shape.

Pointers: upstream format this builds on = https://github.com/agentsmd/agents.md (see README.md).
