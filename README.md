# deep-research

A pi skill that runs a multi-agent web research swarm and returns one cited report.

## Why

A single agent doing research grades its own work. It judges its findings with the same
context that produced them, and nothing checks either. This skill replaces self-grading
with three controls:

1. A swarm of reusable agents, resolved from the agent-directory extension. Each
   researcher owns one direction, so no agent scores its own coverage.
2. State on disk. Every finding lands in a scratchpad file before an agent compacts.
   The final report assembles from the scratchpad, never from agent summaries.
3. A skeptic pass. One skeptic subagent attacks each finding against its source link
   and writes a ternary verdict: `confirmed`, `needs-revisit`, or `rejected`. Rejected
   claims never reach the report.

The result is one report where every claim carries its source URL.

## How it works

The parent agent runs one phase at a time and gates each phase itself. Subagent reports
are data. The parent re-runs every check.

```
                 ┌──────────────┐
 question ─────▶ │ P0  Planner  │──▶ plan.md (3–6 directions, seed queries)
                 └──────────────┘
                        │
        ┌───────────────┼──────────────────────────┐
        ▼               ▼                          ▼
 ┌────────────┐  ┌────────────┐            ┌────────────┐
 │ Researcher │  │ Researcher │  ...       │ Researcher │  one per direction
 └─────┬──────┘  └─────┬──────┘            └─────┬──────┘
       └───────────────┴──────────────────────────┘
                       ▼
              scratchpad/findings.md   claim | source | date
              scratchpad/visited.md    shared URL blacklist
                       │
                       ▼
               ┌──────────────┐
               │ P2  Skeptic  │──▶ skeptic/verdicts.md
               └──────┬───────┘    confirmed / needs-revisit / rejected
                      ▼
               ┌──────────────┐
               │ P3  Revisit  │──▶ targeted mini-rounds, findings get
               └──────┬───────┘    revisit: prefix
                      ▼
               ┌──────────────┐
               │ P4  Merger   │──▶ report.md (from confirmed findings only)
               └──────┬───────┘
                      ▼
               ┌──────────────┐
               │ P5  Gate     │──▶ parent runs the checks itself:
               └──────────────┘    coverage, citations, verdict closure
```

State lives on disk, so context stays expendable. A researcher that compacts writes its
notes first and resumes from the notes file. With the optional
deep-research-compact extension, the parent injects `remember` and `skip` directives
into each compaction summary.

Full state contract, file schemas, brief templates, and gate commands:
[deep-research/SKILL.md](deep-research/SKILL.md) and
[deep-research/REFERENCE.md](deep-research/REFERENCE.md).

## Vendored dependencies

Two runtime pieces ship under `deep-research/vendor/`:

1. `vendor/agent-directory/` — loader extension that resolves reusable agents from
   `.pi/agents/*.md` (project) and `.agents/agents/*.md` (user). Copy it into
   `<project>/extensions/` or load it with `pi --extension`.
2. `vendor/custom-compaction.reference.ts` — the pi `session_before_compact` example
   that the planned deep-research-compact extension adapts. Reference material, not
   loaded.

## Install

Prerequisites: pi with subagent and web tools, plus a `deepseek/deepseek-flash:low`
model id for researchers and a `kimi-coding/kimi-for-coding:low` id for skeptic and
merger. Resolve every model id with `subagent({action:"models"})` before a run.

Copy the skill folder into your user skills directory:

```bash
cp -r deep-research ~/.agents/skills/deep-research
```

## Usage

One path to one result. From the pi prompt:

```text
/deep-research
```

Or say: "run deep research on X", "research X in depth".

The skill asks for five policy answers once, then runs phases P0–P5:

1. Research question and scope.
2. Output path for the run directory.
3. Model split for researchers, skeptic, and merger.
4. Researcher count and directions.
5. Compaction directives.

The default run directory is `AGENTIC_PAPERS/deep-research/<slug>-<date>/`. The final
output is `report.md` there, with a `source:` line under every claim.

Not for single questions a web_search answers.

## Contributing

File issues against the protokol11 pack repository. Propose changes to the skill files
first: `deep-research/SKILL.md` (procedure) and `deep-research/REFERENCE.md` (state
contract). The vendored loader tests in
`deep-research/vendor/agent-directory/tests/loader.test.ts` run with the pi extension
test runner.

## License

[MIT](LICENSE) — Copyright (c) 2026 protokol11
