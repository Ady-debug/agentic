# Module 1: Foundations and repo setup

**Time:** 30-45 min | **Tracks:** A, B, C (groundwork) | **Brief:** sections 1, 2

## Why

Before any agent runs, you need a place for it to work and a shared picture of what "done" means. This module builds the repo and the mental model. No agent yet.

## Concepts (read, ~10 min)

1. **Executable specification.** A plain-English scenario that is also an automated test. If code drifts from the behaviour, the build goes red (brief section 1).
2. **The loop.** Human writes scenarios, tooling turns them into tests (red), agent makes them green, pipeline re-checks, human reviews the _behaviour_ diff (brief section 2).
3. **Three tracks.** Prepare (before the agent), Harness (what the agent is given), Validate (how we know it worked). See the [roadmap](./README.md).
4. **Advisory vs enforced.** Instructions and skills ask the agent nicely. Hooks, CI and CODEOWNERS make it mechanical. We build advisory first, then enforcement (Phase 2).

Read brief sections 1 and 2 again with this question: _which steps are Prepare, Harness, Validate?_ Write your answer in the learning log.

## The toy domain

A small **repayment planner**: a user enters an amount and a number of months and sees the monthly payment. It is simple enough to finish quickly yet has real business rules (validation, rounding, limits), which suit scenarios such as "When I plan to repay £1200 over 12 months, Then my monthly payment is £100.00". We build it feature by feature in later modules.

## You do

1. **Use this repo.** Everything in the course is built in this existing repo, so it is both the toy app and the home of the harness (we extract the reusable parts at the end, see the [README](./README.md#building-here-exporting-later)). No new repo is needed. Check it is linked to GitHub:
   ```bash
   cd /Users/Ady/Documents/agentic
   git remote -v
   gh repo view --web
   ```
   Then create a working branch if you prefer not to commit to `main` directly (optional): `git switch -c module-01`.
2. **Scaffold the app in the repo root.** Choose TypeScript yes, ESLint yes, App Router yes, `src/` directory yes, Tailwind your choice.

   ```bash
   npx create-next-app@latest .
   ```

   The folder is not empty (`docs/`, `README.md`, `.gitignore`). Current `create-next-app` scaffolds in place without asking, and **overwrites `README.md` and `.gitignore`**. Expect these things:
   - Restore the two overwritten files afterwards: `git checkout -- README.md .gitignore`. (The original `.gitignore` already covers `node_modules`, `.next` and `next-env.d.ts`.) Check `git status` to confirm `docs/` is untouched.
   - The scaffolder's own install step may fail (for example `EALLOWSCRIPTS` if your `~/.npmrc` sets `allow-scripts`). The files are still created, so just run `npm install` yourself in the root.
   - If you would rather avoid the overwrite, scaffold into a temporary folder and copy across without clobbering: `npx create-next-app@latest tmp-app`, then `rsync -a --ignore-existing --exclude node_modules --exclude .git tmp-app/ ./`, then `rm -rf tmp-app` and `npm install`.

   Understanding why this conflict happens is part of the exercise.

3. **Run it** with `npm run dev` and open the page. Stop it.
4. **Add Vitest** (unit layer) now, because it is needed from Module 4. Follow the current [Vitest docs](https://vitest.dev/guide/) and add an `npm test` script (`"test": "vitest run"`). Write one trivial passing test to prove it runs. If `npm install -D vitest` fails with `ERESOLVE` on `@types/node`, the scaffold pinned `@types/node@^20` but Vitest needs `^22` or newer. Match your Node version instead of forcing it: `npm install -D @types/node@^24 vitest` (for Node 24).
5. **Create a `features/` folder** (add a `.gitkeep`) for the Gherkin files we write next module.
6. **Commit and push** with the message `module-01: scaffold toy app`.
7. **Start the [learning log](./learning-log.md)** with an entry for this module.

## Check

- [x] `npm run dev` serves the default page
- [x] `npm test` passes
- [x] `npm run lint` passes
- [x] The repo is on GitHub and **Actions is enabled** (Settings, Actions)
- [x] `docs/` (including the brief and this course) is still in the repo after scaffolding
- [x] Learning log has a Module 1 entry, including your Prepare/Harness/Validate classification

## Reflect

- Which brief steps did you class as Prepare, Harness, Validate? Which were hard to place?
- Where in your team's current process does each track already happen informally?

## Experiment

Ask Copilot (VS Code agent mode) _"Add a repayment calculator to this app"_ **without** any scenarios or rules, on a throwaway branch. Note what it produces and how you'd know whether it's right. Delete the branch afterwards. This is your "before" picture; we compare against it after Module 4.

## Next

Module 2: write the first Gherkin scenario, install playwright-bdd, and get the suite red for the right reason.
