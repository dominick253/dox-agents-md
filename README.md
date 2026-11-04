# DOX - an AGENTS.md protocol for agent context that you can measure

DOX is a way of laying out `AGENTS.md` files as a **tree of contracts** instead of one big prompt file, plus the measurement discipline that tells you whether an edit made agent context better or merely shorter.

It builds on the open [AGENTS.md](https://agents.md) format. The original format and its specification live at **[agentsmd/agents.md](https://github.com/agentsmd/agents.md)** - that is the upstream this protocol extends, and it is the source of the `AGENTS.md` convention itself. DOX adds two things on top: the distributed parent/child routing layer, and a cost model so "we cut 40% of the context" is a measurement rather than a claim.

**[`AGENTS.md`](AGENTS.md) is the template: copy it, fill the `<placeholders>`, create the three children it routes to.** It is 2.9 KB on purpose - a root contract is loaded by every agent that starts anywhere in the tree, so a template that teaches the discipline by staying small is the discipline. This README is the reasoning; the contract is what you ship.

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
| **neutral test, one synthetic repo, 16 identical facts** (two attempts, both losing) | fat template **5,560 B vs 2,474 B** (2.25x); slimmed template **2,886 B vs 2,474 B** (+17%) | fat: tools-on **100% vs 95.8%** (wins on quality, loses on cost). slim: tools-on **91.7% vs 100%** (loses on both). Neither variant passes cost **and** quality at this scale |
| infra index split into 6 topic children | per-read **-50% to -65%** on seven task shapes | tools-on **66.7% -> 75.0%**, fatal misses 3 -> 2 |

Retention was proven per edit, not asserted: byte-exact snapshots taken before editing, identifier diff (`comm -23`) against the union of the new core plus its children, empty every time. Where a check found a genuine loss, the fact was restored - which is the whole argument for snapshot-before-editing.

## When DOX pays, and when it does not

Measured, not asserted - the first neutral test we ran came back **against** us and changed this section.

- **One small repo: do not adopt DOX.** Measured twice, and it lost twice. The plain upstream file injected 2,474 B and scored tools-on 100%. The DOX arm at 5,560 B scored 100% (better than nothing, 2.25x the cost); trimmed to 2,886 B it scored 91.7% (cheap, but now worse on both axes, the whole gap being one probe). A single flat directory should use upstream `AGENTS.md` and stop here.
- **The mechanism, which is the useful part.** The two DOX arms had the *same children* and different cores, and their routing differed sharply: the fat core sent the agent to exactly one correct child (6/6 detail probes, 7 tool calls), the slim core made it open 1.8-2.1 files per probe, minimally just 1/22, with 42 accesses outside the repo, because a core stripped to pure protocol has no anchor token telling the agent which child holds the port, the unit name or the test name. Routing quality tracks **anchor tokens in the index line**, not the existence of children. Over-shrinking a contract is a real failure mode, not a safer direction.
- **Where the fixed overhead returns profit: shared contracts.** A root contract is loaded by every directory beneath it, so hub bytes are multiplied by the number of working dirs while the saving is paid per directory. On a live 60-contract workspace: root contract **-43% injected**, a large repo contract **-49.8%**, on-demand children **-50% to -65% per task shape**, probe effectiveness flat or up. A 2.25x penalty in one repo becomes a 40-50% saving across a workspace.
- **Recall scores will look worse and mean nothing.** A tool-less agent cannot open a child, so every offload registers as a loss (DOX 0-66.7% vs single-file 100%, with zero facts actually lost). Judge an offload on tools-on effectiveness plus bytes actually read.

An ablation is running to find the cheapest fix (anchor tokens in the index lines vs keeping hot facts in the core) and this section will carry its number when it lands, whichever way it comes out.

## Four measurement lessons that changed the protocol

1. **Recall mode and tools mode disagree, and only one is real.** A tool-less run scores every offload as a loss, because it cannot open the child. Judging a shrink on recall alone punishes exactly the improvement you wanted. Gate on tools-on effectiveness plus measured per-read bytes.
2. **A "blind" control that has tools is not a control.** Run probes from a contractless cwd with tools disabled; anything that arm passes is testing the model's guessing, not your contract. A small always-injected pointer file can leak enough path names to pass routing probes, so anchor probes to *values*, not to file names.
3. **Provider token accounting is not evidence.** In these runs the arm with 6.8 KB *more* contract text reported *fewer* tokens. Measure the injected bytes from the transcript the runtime writes.
4. **A retention proof must diff a snapshot, not a memory.** One split passed a self-reported losslessness check and still silently dropped a hard rule, because the "before" it diffed was a reconstruction. Snapshot first, in the same task; and classify each offload failure as MOVED-TO-CHILD or LOST, with zero tolerance for LOST.

## Adopting it

1. Copy `AGENTS.md`, rename the title to your project, keep the grammar legend at the top so dense lines stay unambiguous, and cut any line that no agent in your tree actually needs before its first tool call.
2. Move detail into `context/<topic>.md` children, each opening with a `Load:` line that names the task shapes that need it, and let the index keep the old filename when you split something existing - every inbound reference keeps resolving.
3. Before your next compression pass, snapshot the files, then prove emptiness of the dropped-identifier set. Never accept "I rewrote it and it reads fine".
