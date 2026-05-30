# Testing discipline rules

Reusable across any software project. Applies when writing or editing tests.
Load via `/vibe-testing-rules`.

## 1. Test behavior, not implementation
Assert on observable outputs and effects, not internal calls or private state. Tests coupled to implementation break on every refactor and test nothing real. Test the contract.

## 2. Arrange-Act-Assert structure
Each test: set up inputs, perform the one action, assert the outcome. Clear three-part shape makes tests readable and failures obvious. No logic branches inside a test.

## 3. One reason to fail per test
A test checks one behavior. Many unrelated assertions in one test obscure what broke. Split into focused tests named for what each verifies.

## 4. Keep tests deterministic
No dependence on real time, network, randomness, or execution order. Inject or mock the clock, RNG, and external calls. Flaky tests erode trust until everyone ignores them.

## 5. Cover edge cases and error paths
Empty, null, boundary, max, duplicate, malformed inputs — not just the happy path. Assert that errors are raised when they should be. Bugs live in the cases you skipped.

## 6. Fast unit tests, isolated slow tests
Unit tests run in milliseconds and gate every commit. Keep slow integration/e2e tests in a separate suite so the fast feedback loop stays fast. Slow default suite = skipped suite.

## 7. Tests are first-class code
Readable, maintained, DRY via helpers/fixtures. Delete dead and duplicate tests. A test you can't understand is a liability, not a safety net.

## 8. Name tests for scenario and expected outcome
`returns_401_when_token_expired`, not `test3`. Name should read as a sentence describing the case. Failure output then tells you exactly what broke without opening the test.

## 9. Don't test the framework or third-party libs
Test your logic, not that the ORM saves or the framework routes. Assume dependencies work; test your usage of them at integration boundaries only.

## 10. Run tests before commit; failing test stops the line
Green before you commit. A failing test is a stop signal, never something to skip, comment out, or `xfail` away. If a test is wrong, fix or delete it deliberately.

## 11. New bug → failing test first, then fix
Reproduce every bug with a failing test before fixing it. The test proves the fix works and guards against regression forever. No regression test = the bug comes back.

## 12. Cover risk, don't chase 100%
Prioritize coverage of complex logic, money/security paths, and bug-prone areas. Chasing a coverage number rewards trivial getter tests and punishes nothing. Coverage is a hint, not a goal.
