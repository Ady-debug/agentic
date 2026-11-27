# Learning log

One entry per module. Keep it honest: what worked, what failed, what you'd tell a colleague.

## Template

### Module N: title (date)

- **What I did:**
- **What I expected vs what happened:**
- **Where the agent went wrong or drifted:**
- **What I'd change in the process:**
- **How this maps to Jira / Angular / .NET:**
- **Open questions:**

---

## Export candidates

Reusable files to extract in Module 18. Status: _experimental_, _settled_ (used across several modules unchanged) or _dropped_.

| File | App-specific parts to generalise | Status | Module |
| ---- | -------------------------------- | ------ | ------ |
|      |                                  |        |        |

### Module 1: Foundations (27/11/27)

Executable specifications are specifications that are tied to tests, allowing a machine to verify whether the system meets the specification. If a specification has to be read to be verified there is a risk this will drift over time, especially when involving agents.

If system changes violate the specification you know immediately - if the specification change, the tests change with it.

It's agnostic to the way the system is built, hand written code or AI driven, all that matters is the tests and spec agree with the system passing.

Executable tests build a continuous verification mechanism. In the AI era this matters more than ever, AI generates code extremely fast and can changes hundreds of lines in a single prompt. Without executable specs it isn't possible to verify that new changes don't violate previous design decisions.

Consider using **given/when/then** from domain behaviour (maps to BDD frameworks) and **describe/it/should** for components (maps to test frameworks).

E.g.

```md
Feature: Repayment planner
As a user
I want to see my monthly payment for an amount and a term
So that I know whether I can afford to borrow

Scenario: Monthly payment for an even split
Given I am on the repayment planner
When I plan to repay £1200 over 12 months
Then my monthly payment is £100.00
```

Translates to:

```ts
// features/steps/repayment-planner.steps.ts
import { createBdd } from "playwright-bdd";
const { Given, When, Then } = createBdd();

Given("I am on the repayment planner", async ({ page }) => {
  await page.goto("/");
});

When(
  "I plan to repay £{int} over {int} months",
  async ({ page }, amount: number, months: number) => {
    await page.getByLabel("Amount").fill(String(amount));
    await page.getByLabel("Months").fill(String(months));
    await page.getByRole("button", { name: "Calculate" }).click();
  },
);
```

and...

```md
### `calculateMonthlyPayment(amount, months)`

**describe** with valid input

- **it** should return amount divided by months
- **it** should round to 2 decimal places
- **it** should return a number, not a formatted string
```

Translates to:

```ts
// src/lib/calculate-monthly-payment.test.ts
import { describe, it, expect } from 'vitest';
import { calculateMonthlyPayment } from './calculate-monthly-payment';

describe('calculateMonthlyPayment', () => {
  describe('with valid input', () => {
    it('should return amount divided by months', () => {
      expect(calculateMonthlyPayment(1200, 12)).toBe(100);
    });

    it('should round to 2 decimal places', () => {
      expect(calculateMonthlyPayment(1000, 3)).toBe(333.33);
    });
  });
```

These for part of the **_PREPARE_** step of the cycle.
