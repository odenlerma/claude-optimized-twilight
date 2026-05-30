# Clean code structure rules

Reusable across any software project. Applies when writing or editing code.
Load via `/vibe-clean-code-rules`.

## 1. One function, one job
Each function does single thing at single level of abstraction. Mixed concerns (fetch + validate + render + log in one function) are hard to read, test, reuse. Name reveals the one job. Can't name it without "and" → split it.

## 2. Keep functions short
Target small functions — roughly a screen or less. Long function hides multiple responsibilities and branches. Extract named helpers. Short functions read like prose and isolate failures.

## 3. Name things for intent, not implementation
Names say what/why, not how or type. `activeUsers` not `arr2`. `isEligible` not `flag`. Booleans read as predicates (`hasAccess`, `isValid`). Good name removes need for a comment. Searchable, pronounceable, no cryptic abbreviations.

## 4. Delete dead code
Remove unused functions, variables, imports, commented-out blocks. Dead code lies about what runs and rots over time. Version control remembers — no need to keep corpses around "just in case".

## 5. DRY — but do not abstract prematurely
Duplicated knowledge (a business rule in three places) gets one source of truth. But two lines that merely look alike today are not duplication — wait for the third real occurrence (rule of three) before abstracting. Wrong abstraction costs more than duplication.

## 6. Handle errors explicitly — no silent swallow
Never catch-and-ignore. Handle, or propagate with context. Empty `catch {}` hides bugs until production. Fail fast and loud near the cause. Validate inputs at the boundary, not deep in the call stack.

## 7. Comments explain why, not what
Code says what; comment says why — the tradeoff, the workaround, the non-obvious constraint, the link to the bug. Comment restating the code rots and adds noise. Self-explanatory code needs no narration. Document the surprising.

## 8. No magic numbers or strings
Replace literal `86400`, `0.15`, `"ADMIN"` with named constants (`SECONDS_PER_DAY`, `TAX_RATE`, `Role.ADMIN`). Name explains meaning and centralizes change. Bare literal scattered across files is a maintenance trap.

## 9. Keep structure and style consistent
Match project's existing patterns — naming, file layout, formatting, error style. Consistency lowers reading cost more than any single "better" style. Use a formatter/linter so style is automatic, not argued. New code looks like it belongs.

## 10. Dependencies point inward (toward stable code)
High-level policy must not depend on low-level detail — depend on abstractions, inject the concrete. Business logic should not import the web framework, DB driver, or file system directly. Wrong dependency direction makes code untestable and rigid.

## 11. Write code you can test without the world
Isolate side effects (I/O, network, time, randomness) from logic so logic is testable with plain inputs/outputs. Pass dependencies in (injection) instead of reaching for globals/singletons. Hard-to-test code is usually badly structured code.

## 12. Prefer pure functions and immutability
Functions that return outputs from inputs without mutating shared state are easier to reason about, test, parallelize. Avoid hidden mutation of arguments and shared globals. Mutable shared state is source of the worst bugs.

## 13. Minimize function arguments
Few parameters (target three or fewer). Many params signal function does too much or needs a grouping object. Avoid boolean flag params that fork behavior — split into two named functions instead.

## 14. Keep nesting shallow with guard clauses
Return/throw early on invalid cases instead of wrapping happy path in deep `if` pyramids. Flat code with guard clauses reads top-to-bottom. Each nesting level adds branches the reader must hold in their head.

## 15. Leave clear seams at module boundaries
A module exposes small, intentional public surface and hides internals. Callers depend on the interface, not the guts. Tight, documented boundaries let you change implementation without breaking everyone. Big balls of cross-imported state cannot evolve.
