---
name: qa-execute
description: >
  Run a MANUAL QA test plan by hand through a browser-automation MCP, capture a
  screenshot per scenario as evidence, and write a plain-language pass/fail
  report (Scenario | Expected result | Screenshot | Pass/Fail table + issues log
  + verdict). Use when user says "run qa", "execute test plan", "manual qa",
  "qa with screenshots", "do the qa", after a test plan exists, or invokes
  `/feature-qa`. Runs after `qa-test-plan`, before `qa-jira-report`.
---

# QA execute

Run manual QA. Capture screenshot evidence per scenario. Write plain pass/fail report. No jargon.

## Hard rules

- **Manual QA only.** Drive a browser-automation MCP step by step like a human tester. No automated test code, no test runners.
- Identify browser tool by its **description**, not its name. Names vary across vendors.
- No browser tool detected → cannot capture evidence. Stop, report what is missing.
- One screenshot per scenario. No screenshot → scenario cannot pass.
- Any scenario fails OR any issue logged → overall verdict FAIL.

## Flow

### 1. Load plan

Read `qa-reports/<TICKET>/test-plan.md`. Missing → run `qa-test-plan` for `<TICKET>` first, then return here.

### 2. Detect browser tool

Scan currently loaded tool descriptions/schemas. Match by **description content**, not name.

Tool qualifies if its description:

- Says it navigates web pages, clicks, types, or fills forms in a browser, and
- Can take a screenshot of the page.

Names vary (Playwright MCP, browser MCP, vendor-specific). Read description text.

Zero qualify → print:

```
No browser-automation MCP detected. Manual QA needs one to drive the app and capture evidence. Wire a browser MCP (e.g. Playwright) via your config, then re-run.
```

Stop. Do not fake results.

### 3. Run each scenario

For each scenario in the plan, in order:

1. Do the steps by hand through the browser tool.
2. Observe actual result vs expected.
3. Screenshot the result → `qa-reports/<TICKET>/screenshots/<n>-<slug>.png` (`<n>` scenario number, `<slug>` short kebab name).
4. Mark **Pass** (actual matches expected) or **Fail** (does not).

Regression scenario differs from expected → it is a Fail (something broke that should not).

### 4. Log issues

Any bug, error, glitch, or odd behavior seen during a run → log it in the Issues section, plain words. Includes problems outside the scenario being tested.

### 5. Verdict

- **PASS** — every scenario Pass AND zero issues logged.
- **FAIL** — any scenario Fail OR any issue logged.

### 6. Write report

Write `qa-reports/<TICKET>/report.md`:

```markdown
# QA report — <TICKET>

Ticket: <TICKET>
Verdict: PASS | FAIL

| Scenario | Expected result | Screenshot | Pass/Fail |
|---|---|---|---|
| <plain what tester did> | <plain what should happen> | ![](screenshots/1-....png) | Pass |

## Issues
- (none) | - <plain description of problem seen>
```

Table cells: plain words, no technical terms. Screenshot cell links the relative path.

Tell user report path + verdict. PASS → next step `qa-jira-report` for `<TICKET>`. FAIL → fix issues, do not report to Jira.
