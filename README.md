# DOX - an AGENTS.md protocol for agent context that you can measure

DOX is a way of laying out `AGENTS.md` files as a **tree of contracts** instead of one big prompt file, plus the measurement discipline that tells you whether an edit made agent context better or merely shorter.

It builds on the open [AGENTS.md](https://agents.md) format. The original format and its specification live at **[agentsmd/agents.md](https://github.com/agentsmd/agents.md)** - that is the upstream this protocol extends, and it is the source of the `AGENTS.md` convention itself. DOX adds two things on top: the distributed parent/child routing layer, and a cost model so "we cut 40% of the context" is a measurement rather than a claim.

See [`AGENTS.md`](AGENTS.md) for the protocol written in its own style, including the grammar legend, budgets, verification rules and the retention proof.

## The problem it solves

An agent that starts in a directory loads that directory's `AGENTS.md`, and in most runtimes also every `AGENTS.md` above it. Everything you write in a shared root contract is therefore paid **once per working directory**. In a 12-repo workspace, a 17 KB root contract costs roughly 200 KB of context reads across a day of work, and most of it is irrelevant to the task at hand.

The instinct is to trim prose. That is not the lever. The lever is **routing**: keep in the always-loaded chain only what an agent needs before its first tool call, and move everything else into children that are loaded on demand because their `Load:` trigger names the task shape that justifies them. On-disk bytes frequently go *up* after this, while the bytes a task actually reads go down by half or more.

## What is in a DOX tree

- **Pointer chain.** Each contract opens with `# <dir> contract` and a `Chain:` line naming its parent. Any agent landing anywhere can walk up to the root and reach the whole system from that dir's scope.
- **`Load:` triggers.** Every opt-in child states the task shapes that justify reading it, so an agent reads one 2 KB file instead of a 12 KB topic folder.
- **Ownership = deepest need.** A fact lives in the deepest contract where an agent uses it; the parent keeps one one-line pointer. Budgets are per file type, and overflow is resolved by offloading, never by truncating a fact away.
- **A grammar**, so dense telegraphic lines stay unambiguous (`->`, `<-`, `=>`, `!`, `STALE`, `OPEN`, `[date]`).
- **Verification rules**, including a greppable exception (`EVIDENCE-BEFORE:` / `EVIDENCE-AFTER:` / `NO AUTOMATED COVERAGE POSSIBLE:`) for docs/config/infra changes that can't have a failing test, so "docs-only" is still evidence.

## Why the protocol is worth adopting: measured

All numbers below come from a live 60-contract workspace, measured with the harness described in `AGENTS.md`. Injection is measured from the transcript each runtime writes (bytes of contract text actually sent), and effectiveness is scored by probe banks whose answers are anchored to specific contract lines.

| change | context cost | effectiveness |
|---|---|---|
| one repo contract rewritten dense (34,086 B -> 7,665 B) | injected at that dir **52,883 B -> 26,402 B (-49.8%)** | probe bank majority **61.5% -> 100%**, tools-on **76.9% -> 100%**, zero facts lost |
| hub/root contract split into core + opt-in children (17.9 KB -> 10.0 KB) | injected at root **-43%**, at the ecosystem hub **-42%** | tools-on **100%** held, recall flat (as designed) |
| on-demand children split by task shape (index kept the same filename) | bytes a task reads **50-55% lower** per shape | recall bank **100%** held, 0 orphan children, all inbound refs still resolving |
| dense vs verbose head-to-head, same facts | 3,439 B vs 10,249 B | dense **83.3%** vs verbose **66.7%** - smaller *and* more effective |
| infra index split into 6 topic children | per-read **-50% to -65%** on seven task shapes | tools-on **66.7% -> 75.0%**, fatal misses 3 -> 2 |

Retention was proven per edit, not asserted: byte-exact snapshots taken before editing, identifier diff (`comm -23`) against the union of the new core plus its children, empty every time. Where a check found a genuine loss, the fact was restored - which is the whole argument for snapshot-before-editing.

## Four measurement lessons that changed the protocol

1. **Recall mode and tools mode disagree, and only one is real.** A tool-less run scores every offload as a loss, because it cannot open the child. Judging a shrink on recall alone punishes exactly the improvement you wanted. Gate on tools-on effectiveness plus measured per-read bytes.
2. **A "blind" control that has tools is not a control.** Run probes from a contractless cwd with tools disabled; anything that arm passes is testing the model's guessing, not your contract. A small always-injected pointer file can leak enough path names to pass routing probes, so anchor probes to *values*, not to file names.
3. **Provider token accounting is not evidence.** In these runs the arm with 6.8 KB *more* contract text reported *fewer* tokens. Measure the injected bytes from the transcript the runtime writes.
4. **A retention proof must diff a snapshot, not a memory.** One split passed a self-reported losslessness check and still silently dropped a hard rule, because the "before" it diffed was a reconstruction. Snapshot first, in the same task; and classify each offload failure as MOVED-TO-CHILD or LOST, with zero tolerance for LOST.

## Adopting it

1. Write the root contract as a routing index: identity, hard rules, budgets, and one pointer line per child. Keep the grammar legend at the top so density stays unambiguous.
2. Move detail into `context/<topic>.md` children, each opening with a `Load:` line that names the task shapes that need it, and let the index keep the old filename when you split something existing - every inbound reference keeps resolving.
3. Before your next compression pass, snapshot the files, then prove emptiness of the dropped-identifier set. Never accept "I rewrote it and it reads fine".
