# Agentic coding with executable specifications

A summary of the process, a pitch for the executive team, and a guide to the accompanying skill pack.

## 1. Summary

AI coding agents write code quickly, but they can drift from what the business actually asked for. A markdown specification is like the Highway Code: it describes good driving, but nobody fails for ignoring it. An **executable specification** is the driving test: an automated examiner that the work must pass before it ships.

The approach, following Dave Farley's argument for BDD/ATDD:

- Write behaviour as plain-English scenarios (Given / When / Then, in Gherkin).
- Turn those scenarios into automated acceptance tests. The build goes red if the code drifts from the behaviour.
- Let the agent work until red becomes green, using ordinary unit-level TDD inside that loop.
- Humans own and approve the scenarios. Machines do the checking.
- The scenarios stay in place afterwards as the regression suite and the living documentation.

The same idea scales down to legacy systems: a thin set of characterization tests around the riskiest journeys makes a framework upgrade safe enough to attempt without a full test suite.

## 2. The process for new development

| # | What happens | Who |
| --- | --- | --- |
| 1 | Write or approve Gherkin scenarios in domain language, one per business rule | Human |
| 2 | `bddgen` (playwright-bdd) generates Playwright tests from the Gherkin. A scenario with no matching step definition fails here, before anything runs | Tooling |
| 3 | Run the suite: everything is red, and red for the right reason (missing field, not a typo in a selector) | Pipeline |
| 4 | Agent picks **one** scenario. Inner loop: write a failing unit test, write minimal code to pass it, refactor | Agent |
| 5 | The scenario goes green. Repeat steps 4 and 5 for the next scenario | Agent |
| 6 | Fast gates run in hooks: lint, type-check, unit tests, the feature's scenarios. A Stop hook blocks the agent from finishing while anything is red | Hooks |
| 7 | Slower quality gates run in the pipeline: coverage, mutation testing on changed files, complexity/CRAP thresholds, no skipped tests | Pipeline |
| 8 | CI re-runs everything as a required check on the pull request. CODEOWNERS protects the specs and step definitions | Pipeline |
| 9 | A human reviews the Gherkin diff and any step-definition changes. The question is "are these the right behaviours?", not "is this code right?" | Human |
| 10 | Merge. The scenarios stay as regression tests and documentation |  |

**Typical stack for a Next.js and TypeScript app:** playwright-bdd and Playwright (acceptance), Vitest (unit), Stryker (mutation), ESLint complexity rules plus a coverage report (CRAP).

### How Gherkin becomes a test

They are not unit tests. They are end-to-end acceptance tests that drive the running app. `bddgen` reads each `.feature` file and writes real Playwright test files into a gitignored folder, rebuilt on every run, so the `.feature` file is the only source of truth. Each scenario becomes one test; each line is matched by its text to a step definition.

| If the Gherkin changes... | Result |
| --- | --- |
| A value is edited | The new value flows through; the test goes red if the app disagrees |
| A scenario is added using existing wording | A new test appears automatically |
| A step is reworded | `bddgen` fails with a missing-step error, before the app is touched |
| A scenario is deleted | The test vanishes. This is the one silent change, so protect it with CODEOWNERS and review |

### Farley's layering

Think of ordering coffee: the scenario is the order, the domain DSL is the menu, and the protocol driver is the barista's workflow. A kitchen refactor changes the workflow, not the menu or the order.

| Layer | Contains | Who changes it |
| --- | --- | --- |
| Test cases | Gherkin scenarios | Humans |
| Domain DSL | Business-language helpers | Rarely |
| Protocol driver | Browser, HTTP or DB calls | Agent, with review, when the implementation changes |
| System under test | The application | Agent |

## 3. Keeping the suite lean and the steps tidy

**Lean suite**

- Acceptance scenarios cover business rules (few, slow). Edge cases belong in unit tests (many, fast). A thin set of browser journeys covers wiring.
- Write declarative scenarios ("When I plan to repay £1200 over 12 months") rather than click-by-click ones.
- Edit a scenario when behaviour changes. Never add a "v2" next to the old one.
- Test public behaviour, not internals. Mock-asserting tests break on every refactor and add cost without confidence.
- Use mutation testing as a hint for pruning: a test that never uniquely catches anything is a candidate for review.
- Run only what is relevant in the inner loop (tag by feature), the full suite in CI, and set a time budget for the suite.

**Tidy steps.** Nothing detects that "I press Calculate" and "I click Calculate" mean the same thing, because the tooling matches text, not meaning. Prevent duplicates by:

- listing existing steps before writing a new one (IDE Cucumber plugins, or the `bddgen export` command, verified against your playwright-bdd version);
- writing fewer, parameterised steps (`I press {string}`);
- picking one verb and enforcing it;
- reviewing step-file diffs, with CODEOWNERS on the folder.

## 4. What to do for each kind of change

| Situation | What you do |
| --- | --- |
| New feature | Add scenarios (red). Agent builds with unit tests. Scenarios stay |
| Behaviour change | **Edit** the existing scenario first (red), then implement |
| Bug fix | Write a failing test at the *lowest* layer that reproduces it. Add a scenario only if it exposes a missing business rule |
| Feature removed | Delete its scenarios and the code |
| Pure refactor | Change no tests. If tests break, they were coupled to implementation |
| Package, framework or security update | Change no scenarios. Update the dependency, run the full suite, let red/green decide. Fix breakages in code or the driver layer, not by editing specs |

When a scenario goes red after a dependency update there are three causes: a real regression (fix the code or pin the version), a stale driver layer (update with human review), or an intentional behaviour change in the library (a human decides).

## 5. Upgrading a legacy system with no safety net

An upgrade is a refactor, so behaviour must not change and something must prove it. With a short remaining lifespan you only need a **thin** net. Think of moving a house onto new foundations: photograph each room first, move the house, compare.

**Step 0: does it need upgrading at all?**

| Option | When it fits |
| --- | --- |
| Retire sooner | The replacement is nearly ready, or usage is low |
| Isolate and mitigate | The risk is "out of support", not "broken": network restrictions, WAF, tighter access |
| Paid extended support | Available, and cheaper than the upgrade |
| Upgrade | Support is mandatory (compliance, audit, supplier) and the retirement date is far enough away |

Compare the support end date with the retirement date, and set exit criteria and a time-box up front.

**Steps 1 to 4**

1. **Build a thin characterization layer.** Roughly 10 to 20 scenarios chosen by risk (usage, financial or regulatory impact, past incidents), sourced from logs, tickets and the people who run the system. Snapshot current behaviour rather than hand-writing expectations. Use Gherkin only if the replacement system will reuse the scenarios.
2. **Prove the net on the old system first.** Run it against the unchanged legacy system: it should be green, repeatable, and go red when you deliberately break something small.
3. **Upgrade in small hops.** One major version per pull request, official migration tooling first, full suite after every hop, agents never edit tests or snapshots.
4. **Cut over safely.** Pre-production with realistic data, a side-by-side comparison of old and new outputs, a tested rollback, a staged rollout, and monitoring in the first days.

**How the full process shrinks for legacy work**

| Stage | Legacy upgrade version |
| --- | --- |
| Gherkin scenarios | Shrink to 10 to 20 risk-based journeys |
| Generate tests | Keep if using Gherkin, otherwise plain Playwright or snapshot tests |
| Run red first | Invert: green on the old system, then red means the upgrade broke something |
| Unit-level TDD | Mostly skip; add unit tests only where real logic changes |
| Mutation and CRAP | Skip; their payoff is long-term code health |
| CI required check, Stop hook, CODEOWNERS | Keep; cheap and they protect the net |
| Human review of test changes | Keep |

The residual risk is a decision, not an accident: list what the tests cover and what they do not, and have the system owner sign off on the gap.

## 6. AGENTS.md or skills?

`AGENTS.md` is the house-rules poster: always visible, so short and always true. A skill is a manual on the shelf: only its name and description load up front, and the full instructions load when the task matches. Skills can also bundle scripts and checklists.

- **Keep in `AGENTS.md`:** definition of done, protected paths, "if a scenario looks wrong, stop and ask", basic test hygiene.
- **Move to skills:** the task-specific procedures (writing scenarios, implementing a scenario, bug fixes, dependency updates, characterization tests, framework upgrades, quality gates).
- **Skills teach the procedure; hooks, CI and CODEOWNERS enforce the outcome.** Instructions are advisory; enforcement is mechanical.
- Each rule lives in one place only, and skill support varies by tool, so keep the non-negotiable guardrails in `AGENTS.md`.

## 7. Pitch for the executive team

### The headline

**Let AI write the code, but make the business rules decide whether it ships.**

### The problem

- AI coding agents are fast, but they can quietly drift from what was asked. Reviewing every line of generated code does not scale.
- Written specifications are not enforced. Nothing fails when they are ignored.
- Our legacy systems need to stay supported until they are retired, and upgrading without tests is risky.

### The proposal

Adopt **executable specifications** as the control point for agent-written code. Business behaviour is written as plain-English scenarios that double as automated tests. The pipeline will not accept a change unless every scenario passes. People approve *what the system should do*; the pipeline proves *that it does it*.

### What changes for people

- Product and business owners can read and approve scenarios without reading code.
- Engineers spend less time line-reviewing generated code and more time deciding whether the behaviour is right.
- Agents get an objective finish line instead of a description to interpret.

### What we gain

| Benefit | How |
| --- | --- |
| Speed with control | Agents work quickly inside a loop that cannot be skipped: red, then green |
| Audit trail | Scenarios are living documentation; history shows the failing test first and the passing implementation after |
| Lower regression risk | Every agreed behaviour is checked on every change |
| Faster patching | Security and framework updates can merge with confidence when the suite is green |
| Reuse | Scenarios outlive the code and become the acceptance spec for the replacement systems |
| Safer legacy work | A thin safety net makes supported-version upgrades defensible |

### What it is not

- Not removing human oversight. Humans own the specifications and review any change to them.
- Not a big-bang rewrite or a large tool purchase. The core tooling (Playwright, playwright-bdd, Vitest, Stryker) is open source; the main costs are people's time and agent licences.
- Not a promise of perfect tests. Mutation testing and review exist to catch weak ones.

### Risks and mitigations

| Risk | Mitigation |
| --- | --- |
| Agents edit tests to get green | Protected paths, CODEOWNERS, a CI guard on agent branches, separate author and implementer roles |
| Slow, bloated test suites | Keep acceptance scenarios few, push detail to unit tests, set a time budget |
| Weak tests give false confidence | Mutation testing, human review of the Gherkin diff, no skipped tests |
| Upfront effort writing scenarios | Start with one feature and one legacy system; the effort is written down once and reused |
| Tool support differs between agents | Keep non-negotiable guardrails in `AGENTS.md`, hooks and CI |

### Proposed pilot

A time-boxed pilot (suggested six to eight weeks, to be adjusted) with two strands:

1. **New build:** one small feature delivered end to end through the process.
2. **Legacy upgrade:** one system moved to a supported framework using the thin characterization net.

**Measure:** lead time per change, change failure rate, defects escaping to production, human review time per change, agent rework rate, and share of agreed journeys covered by scenarios. Capture a baseline first so the comparison is fair.

**Decision gate:** at the end, continue, adjust or stop, based on those measures.

### The ask

- An executive sponsor.
- Two engineers and a product owner part-time for the pilot.
- Access to agent tooling and CI changes for the pilot repositories.
- Agreement to the guardrails: humans approve specifications, and the pipeline decides what ships.

## 8. The skill pack

The accompanying download contains a ready-to-adapt set of files:

| File | Purpose |
| --- | --- |
| `AGENTS.md` | Short baseline rules that always apply |
| `.claude/skills/write-scenarios` | Writing Gherkin and step definitions without duplicates |
| `.claude/skills/implement-scenario` | The outer acceptance loop and inner unit-test loop |
| `.claude/skills/bug-fix` | Reproduce at the lowest layer first |
| `.claude/skills/dependency-update` | Package, framework and security updates |
| `.claude/skills/characterization-tests` | Building the thin safety net for legacy code |
| `.claude/skills/framework-upgrade` | Hop-by-hop upgrade playbook, with checklist and script |
| `.claude/skills/quality-gates` | Coverage, mutation testing and complexity/CRAP |
| `.claude/settings.json` and `scripts/` | The Stop hook and the CI spec guard (enforcement) |
| `.github/` | CODEOWNERS and the acceptance workflow |

These are starting points. Adjust paths, commands and thresholds to your repositories, and confirm each tool's current syntax (especially playwright-bdd's step-listing command and your agent's hook and skill formats) before relying on it.

## 9. Honest limits

- The process is only as good as the scenarios. Weak or missing scenarios mean weak protection, which is why humans review them.
- Skills and `AGENTS.md` are advisory. Only hooks, CI and CODEOWNERS enforce.
- Thin characterization coverage on legacy systems leaves residual risk. Make it visible and get it signed off.
- Thresholds (mutation score, CRAP, suite time) should be agreed with the team and revisited after the pilot, not copied blindly.
