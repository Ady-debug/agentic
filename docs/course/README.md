# Agentic Workflow Course: Breakable Toy

**Goal:** by the end you can explain, implement and defend an agentic delivery workflow in your team, using GitHub Copilot, GitHub and the executable-specification approach in [the brief](../Agentic%20coding%20with%20executable%20specifications.md).

**Format:** short sessions (30-45 min). Each module has *Why*, *Concepts*, *You do* (you type it, not me), *Check* (how you know it worked) and *Reflect* (what goes in your learning log). Ask the assistant to review your work or unblock you, but do the implementation yourself.

**Decisions so far**

| Topic | Choice |
| --- | --- |
| Toy app | Next.js + TypeScript (process kept language-agnostic; Angular and .NET translations come later) |
| Copilot surfaces | VS Code agent mode, Copilot CLI, Copilot coding agent (assign GitHub issues) |
| Work intake | GitHub Issues now; each module notes the Jira equivalent |
| Legacy upgrade strand | Postponed until the new-feature strand works |
| Repo | This repo (`/Users/Ady/Documents/agentic`), pushed to a personal GitHub repo with Actions. Everything is built here, including course notes, toy app and harness files |
| Reuse | Build everything in this repo first; at the end, extract the settled parts into a reusable pack (see [Building here, exporting later](#building-here-exporting-later)) |

## The three tracks

Every module advances one or more of these. The tracks stay separate so you can later adopt them independently.

| Track | Question it answers | Brief sections |
| --- | --- | --- |
| **A. Prepare** | What must exist before an agent starts? (scenarios, issue shape, acceptance criteria, protected paths) | 2 (steps 1-3), 4 |
| **B. Harness** | What do we give the agent? (`AGENTS.md`, skills, custom agents, hooks) | 6, 8 |
| **C. Validate** | How do we know it did the right thing? (acceptance tests, CI, coverage, mutation, complexity, metrics) | 2 (steps 6-9), 3, 7 |

## Roadmap

### Phase 1: Proof of concept (get a loop working end to end)

| # | Module | Tracks | Outcome |
| --- | --- | --- | --- |
| 1 | [Foundations and repo setup](./01-foundations.md) | all | Repo, Next.js toy, baseline docs, learning log |
| 2 | First executable specification | A, C | One Gherkin scenario, `bddgen`, suite red for the right reason |
| 3 | Agent-ready issue and minimal `AGENTS.md` | A, B | Issue template + a short rules file |
| 4 | First agent run (VS Code agent mode) | B | Scenario goes green via agent; you review how it behaved |
| 5 | CI as the referee | C | Required GitHub Actions check on PRs |
| 6 | Hand the issue to Copilot coding agent | A, B, C | Issue to PR to green check with no local involvement |

*Checkpoint after Module 6:* you can demo the whole loop. Write a one-page "what surprised me" note.

### Phase 2: Harden (make it trustworthy)

| # | Module | Tracks | Outcome |
| --- | --- | --- | --- |
| 7 | Split rules into skills (`write-scenarios`, `implement-scenario`) | B | `AGENTS.md` stays short; procedures move to skills |
| 8 | Guardrails: CODEOWNERS, spec guard, branch protection | C | Agent cannot quietly edit specs |
| 9 | Hooks: Stop hook and fast gates | B, C | Agent cannot finish while red (check hook support per surface) |
| 10 | Quality gates: coverage, complexity, CRAP | C | Thresholds enforced in CI |
| 11 | Mutation testing (Stryker) on changed files | C | Evidence on test quality |
| 12 | Separate author and implementer agents | B | Custom agents with different tool permissions |

### Phase 3: Extend (cover real-world work)

| # | Module | Tracks | Outcome |
| --- | --- | --- | --- |
| 13 | Bug-fix flow (lowest-layer test first) | A, B | `bug-fix` skill + bug issue template |
| 14 | Dependency and security updates | A, B, C | Dependabot PR handled with "change no scenarios" rule |
| 15 | Metrics baseline and dashboard | C | Lead time, change failure rate, rework rate captured |
| 16 | Legacy upgrade: characterization net | A, B, C | Old-version toy app, thin net, hop-by-hop upgrade |
| 17 | Translate: Jira intake, Angular, .NET | A, B | Mapping document for the team |
| 18 | Extract and export: team rollout pack | all | Reusable template/pack extracted from this repo, adoption guide, pilot plan, guardrails agreement |

Later modules are written just in time, so the brief can change without rework. Ask for the next module when you finish one.

## Building here, exporting later

Everything is built in this one repo so you can experiment freely and see the whole system working together. Nothing is built "for export" up front.

**Where things live** (this also makes the later extraction easy):

| Kind | Location | Exported at the end? |
| --- | --- | --- |
| Course material, learning log | `docs/course/` | Learning log distilled into the team guide; modules are reference only |
| Toy app (Next.js, unit tests) | `src/`, `package.json` etc. | No, it only exists to exercise the process |
| Gherkin, step definitions | `features/`, `steps/` | Examples only; the *pattern* is exported |
| Agent harness | `AGENTS.md`, `.github/skills/`, `.github/agents/`, `.github/copilot-instructions.md` | **Yes**, after generalising |
| Hooks and guard scripts | `.github/hooks/` or equivalent, `scripts/` | **Yes** |
| CI and ownership | `.github/workflows/`, `.github/CODEOWNERS` | **Yes**, parameterised per repo |
| Intake | `.github/ISSUE_TEMPLATE/` | **Yes**, with a Jira mapping |

**Export candidates.** Whenever you create or settle a reusable file, add a line to the table at the bottom of the [learning log](./learning-log.md): file, what is app-specific in it, and a status (*experimental*, *settled*, *dropped*). Module 18 uses this table to decide what goes into the pack, and "settled" means you've used it across several modules without changing it.

**Keep it generalisable as you go.** Prefer commands via `npm run` scripts over hard-coded tool invocations in skills and `AGENTS.md`, and keep app-specific wording (e.g. "repayment") in examples, not in rules. This is what makes the Angular and .NET translation in Module 17 cheap.

## Working agreements

- **Commit per module** with a message that names it (e.g. `module-02: first scenario`).
- **Keep the [learning log](./learning-log.md)** updated after each module; it becomes your team guide.
- **Break things deliberately.** Each module should include one experiment where you try to make the process fail.
- **Verify tool syntax.** Copilot features change quickly; check the current GitHub docs before relying on any file format or hook name.
