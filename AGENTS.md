# <project> contract (DOX root)

Chain: none - every other contract in this tree points here.
Scope: only what is needed before the first tool call, plus the routing index. Detail lives in opt-in children (`context/*.md`), never here.

Grammar: -> leads-to/imply <- reads-from => must ~ approx + and | or ! hard rule/pitfall STALE=unverified OPEN=owner decision pending ? unknown [date]=verified YYYY-MM-DD. Style: telegraphic, no tables, no em dash; commands, paths, SHAs, ports, test names, dates byte-exact; credential LOCATION only, never the value.

## Protocol
- Chain: nearest `AGENTS.md` above cwd wins conflicts. A child opens `# <topic> contract`, then `Chain:` to its parent, then `Load:`.
- Load: read a child only when its `Load:` line matches the task shape; never the whole sibling set. Split a large topic into topic children behind an index child that keeps the old filename, so inbound refs still resolve.
- Deepest need: a fact lives where an agent uses it; the parent keeps one pointer line. Overflow => offload, never truncate.
- Budgets (bytes): root 17920 | hub 14336 | contract 8192 | index child 3300 | topic child 4096 target, 8192 hard.
- Live state > snapshot: query (`git ls-remote`, fetch, running listener) before claiming absent/stale.
- ! Never delete a load-bearing fact: move it and leave the pointer that reaches it.

## Verification
- Behavior change => failing test FIRST: paste the failing output (test name + command + failure line) before implementing, then the passing output of the same command. Same transcript, same commit.
- No meaningful failing test (docs, config, infra posture) => `EVIDENCE-BEFORE:` and `EVIDENCE-AFTER:` (same commands, measured output) + what changed, or `NO AUTOMATED COVERAGE POSSIBLE: <why>`. The exception never skips verification.

## Editing a contract
`cp` every file you touch before editing it, in the same task. After: the dropped-identifier set vs that snapshot (`comm -23`) must be EMPTY. Refs absolute and literal, no brace shorthand. Classify an offload failure as MOVED-TO-CHILD (provable: parent pointer + child `Load:`) or LOST (zero tolerance).

## Cost model
Report `E% @ injected-bytes`. A dir loads its ancestor chain automatically, so hub bytes are paid once per dir; children are paid only when routed to. Judge an offload on tools-on effectiveness + bytes actually read, never on recall mode. Accept a shrink iff `dE >= -2 pts` AND `new fatal == 0` AND bytes down.

## Map - create these, each opening with its own `Load:` trigger
- `context/build.md` - build/test commands, artifacts, flake handling.
- `context/release.md` - commit/CI/push gates, receipts, failure ladder.
- `context/domain.md` - invariants, canonical tests, files not to edit.

Upstream format this extends: https://github.com/agentsmd/agents.md (rationale + measured evidence in README.md).
