# How I build with AI

The method behind the tools at [github.com/anader298](https://github.com/anader298). Most of what I build produces numbers someone will act on. Section 13 lists where my repos fall short of it today.

## 1. The through-line

AI writes most of the code. The code does the work.

Anything that has to hold up (a figure, a record match, a decision to send) comes from code a test can pin and a rerun reproduces. Models write and review that code at build time. At run time a model reads or drafts language, and its output passes code checks and a person before it counts. Start with the simplest thing that works and add agentic steps only for a measured gain ([Anthropic, Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)).

## 2. Start from the lever, not the model

1. **Pick the lever.** Revenue, margin, cost, working capital, time or risk. For spend, say which: commercial (how you buy: bundling, tenders, terms), technical (the spec), demand (how much and how often) or supplier (who from, and a second source).
2. **Size it.** An agreed baseline and a range for the prize. If the prize is small, stop.
3. **Name the job.** Screen, analyze or automate (section 3).
4. **Then place AI.** Usually at build time, writing the code; at run time only where language is the work.

## 3. Three uses: screen, analyze, automate

| Use | What a leader gets | Usual lever | Where a model may sit | Example |
|---|---|---|---|---|
| Screen | A shortlist, with the reason each item dropped | Revenue, time | Build time only; the criteria are code | farmland-acquisition-map: eight chained tests as a funnel |
| Analyze | An answer with figures and evidence | Margin, cost, risk | Figures from code; a model may draft commentary that code ties back to source | pit-wall: deal sizing and NPV in code; drafts citing unverified figures blocked |
| Automate | Recurring briefs and dashboards that show a leader value | Time, risk | A model may read and draft; anything irreversible waits for a person | chief-of-staff-agent: scheduled briefs; sending gated in code |

## 4. Vocabulary

| Term | What it is | Example |
|---|---|---|
| Playbook / workflow | Fixed steps in a fixed order; my default | farmland-acquisition-map `pipeline/build.py` |
| Agent | A model picking its next step and tool in a loop until a stop condition | chief-of-staff-agent `agents/cos.md` |
| Skill | A packaged procedure (instructions plus scripts) loaded only when needed | chief-of-staff-agent `skills/skill-feedback/SKILL.md` |
| Subagent | A worker with fresh context and a written brief, returning a short result | diligence-rooms `pipeline/PROMPTS.md`: one research agent per person |
| Tool / MCP | A typed function the model can call; MCP is the standard protocol for them | chief-of-staff-agent `tools/mail.py`, which refuses some sends itself |
| Hook | A command the harness runs at a fixed point that can block the action | chief-of-staff-agent `hooks/block_send.sh`, which fails closed |

## 5. Agents vs playbooks

- Workflow patterns (chaining with code gates, routing, parallel runs, evaluator loops) come before a free-running agent.
- An agent needs feedback at each step, a stop condition and a person checking the result.
- Multi-agent runs cost about 15 times a chat's tokens, so they are for broad, high-value work ([Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system)).
- Reads fan out to parallel agents; writes stay with one, because parallel writers make conflicting decisions ([Cognition](https://cognition.ai/blog/dont-build-multi-agents)).

## 6. Roles, briefs and review

I define a role by its inputs, outputs and permissions, not a job title.

| Role | Input | Output | Cannot |
|---|---|---|---|
| Researcher | A question and sources to try | One sourced record per finding | Write to the repo |
| Implementer | One small task and its spec | A commit with tests | Edit existing tests, push to main |
| Reviewer | The diff and the spec | Findings, most severe first | Edit anything (Read, Grep, Glob only) |

Every brief states the goal, what is ruled out, files to read, output format and how to check the result.

The reviewer is a separate session with fresh context and read-only tools enforced by the harness. It checks the diff against the spec. It is one input: tests and CI are the gate, and a person signs off the merge.

## 7. The build loop

1. **Spec**: problem, acceptance checks, edge cases, out of scope.
2. **Plan**, read before any code exists.
3. **Small tasks** that each fit one fresh context.
4. **Tests first**, with expected values from an independent source.
5. **Small commits**, one change each, with the reason.
6. **Fresh-context review** against the spec.
7. **CI** on every push.
8. **Retro**: a mistake made twice gets a hook, deny rule or test, not another line of instructions ([Claude Code best practices](https://code.claude.com/docs/en/best-practices)).

Example: on 26 October an empty recipient list passed chief-of-staff-agent's send gate and one message went out unapproved. The gate now refuses an empty list, pinned by `test_empty_list_blocked`.

## 8. Data rules

- **No arithmetic by a model.** Figures come from Python or SQL. When a model proposes a calculation, code redoes it (`recheckCalc` in diligence-rooms).
- **Re-runnable loads.** farmland-acquisition-map resumes fetches from `.done` markers and refetches by ID when paging comes up short.
- **Reconciliation.** Rows in equal rows out plus rows dropped with a reason; a county count mismatch stops the build.
- **Matching ladder.** Exact keys, rules, guarded fuzzy matching, then a person. Every owner merge and refusal is logged with a reason; an ambiguous LLC gets no people rather than a guess.
- **Lineage.** Every row carries its source.

## 9. Evals

Where code computes the answer, verification is tests and reconciliation. Where a model produces it:

- **Error analysis first**: read real outputs, group failures, fix the biggest group ([Hamel Husain](https://hamel.dev/blog/posts/evals-faq/)).
- **Pass or fail** with a reason, judged by the domain owner.
- **Code checks before model judges**, and a judge only once it agrees with human labels.
- **Regression cases in CI** for every fixed failure.
- **Repeat runs**: 7 against 8 passes out of ten is noise.

## 10. Tooling

- **Claude Code** is my main tool; its conventions exist in other agents too.
- **Plain SDK calls before frameworks**, as in diligence-rooms.
- **Structured outputs**: an enforced schema, validation in code, one retry, and a "not enough evidence" option.
- **A model per step**: small for extraction, frontier for judgement, with a fallback list.
- **Long context before retrieval** when the material fits; diligence-rooms caches the whole room.
- **Least access.** No session gets private data, untrusted content and a way out without a person in the loop ([Simon Willison](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)). chief-of-staff-agent enforces this with a hook and a check inside the mail tool.

## 11. Vibecoded signs and the guard for each

| Sign | Guard |
|---|---|
| No tests, or tests edited until they pass | Tests committed before the code |
| One giant commit | Small commits with reasons |
| Model output parsed with a regex | Enforced schema, validation, abstention |
| A model checking its own numbers | A code tie-out against source data |
| "The QA agent approved it" | Tests, CI and a person decide |
| "Production-ready" in the README | A "where it breaks" section |
| Secrets in the repo | `.env.example`, deny rules, a secret scan |

## 12. The honest productivity evidence

- **METR, July 2025**: experienced developers were 19% slower with AI tools while believing they were about 20% faster.
- **METR, February 2026**: an estimated 18% speedup with newer tools, which METR calls weak evidence.
- **DORA, 2025**: AI use is linked to higher throughput and higher delivery instability.

My own sense of speed is not evidence. Tests passing, defects caught and time to a reviewed change are.

## 13. What I don't do yet

As of November 2026, each is being fixed.

- **History**: one commit per repo, because the private history held secrets or client data.
- **Evals**: no public harness yet; farmland-acquisition-map gets a golden output first.
- **diligence-rooms**: parses model JSON with a regex, defaults its fast lane to a frontier model, has an agent recheck figures where pit-wall uses a code tie-out, and doesn't log tokens or cost.
- **farmland-acquisition-map**: the fetch tolerates a 2% shortfall; the hold-out isn't time-split; "three methods agree" fires on two.
- **pit-wall**: the number check misses small integers and numbers written as words.
- **Reproducibility**: two repos lack a lockfile; none has linter configuration. (CI now runs the tests in all five repos.)

## 14. Sources

- [Anthropic, Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- [Anthropic, Multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system)
- [Anthropic, Claude Code best practices](https://code.claude.com/docs/en/best-practices)
- [Cognition, Don't build multi-agents](https://cognition.ai/blog/dont-build-multi-agents)
- [Simon Willison, The lethal trifecta](https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
- [Hamel Husain, LLM evals FAQ](https://hamel.dev/blog/posts/evals-faq/)
- [METR, 2025 developer study](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/) and [2026 update](https://metr.org/blog/2026-02-24-uplift-update/)
- [DORA, 2025 State of AI-assisted Software Development](https://services.google.com/fh/files/misc/2025_state_of_ai_assisted_software_development.pdf)
