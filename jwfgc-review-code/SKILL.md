---
name: "jwfgc-review-code"
description: "Use when reviewing or refactoring existing code."
---

# Reviewing code against JWFGC

JWFGC (Just Write Freaking Good Code), by Richard Haynes, aims for code that is easy to read and write without degrading performance. Use this skill to audit a file, diff, or PR, and optionally refactor it.

## What to look for

| # | Rule | Flag when you see |
|---|------|-------------------|
| 1 | Types are nouns, functions are verbs | Types named like actions (`ProcessOrder` class), functions named like things (`userData()` that fetches), vague names (`handle`, `doStuff`, `Manager`) |
| 2 | Branch on one value → `switch` / `match` | `if / else if / else if` chains testing the same variable or type |
| 3 | Avoid `else` | `else` after a branch that could return/exit early; deeply nested conditionals |
| 4 | Avoid returning null | Functions returning `null` / `nil` / `undefined` / `None`; callers peppered with null checks |
| 5 | Abstract only to consolidate and reuse | Interfaces with one implementation, factories/strategies/patterns used once, layers with no reuse |
| 6 | Avoid private functions | Private helpers that read/mutate lots of instance state, grab-bag helpers, logic that can't be tested |
| 7 | Boolean encoding a business mode → enum | Boolean fields/params encoding a business mode (`isExpress`, `render(true)`), multiple related booleans |
| 8 | Numeric literal with meaning → enum | Magic numbers or numeric constants representing a fixed set of states/codes |
| 9 | Prevent inheritance when not needed | Non-sealed/non-final classes with no subclasses |
| 10 | Don't do anything you don't agree with | (Applies to the review itself — see below) |

## Applying rule 10 to reviews
- Only flag something if the fix genuinely improves the code. Skip findings that would add churn or fight a deliberate project convention.
- Respect existing style guides, framework requirements (e.g. ORMs or mocking libraries that need non-sealed classes, APIs that must return null for interop), and the user's stated preferences — name what you're deferring to in the finding.
- Explain the *why* for each finding so the reader can agree or disagree — never cite a rule as the only justification.

> **jwfgc:** "I don't personally agree with this rule here" is not a reason to skip a finding — it is an escape hatch for every rule above. A concrete convention, framework constraint, or user preference is. Name it in the skipped-findings note, or raise the finding.

## Language notes for fixes
- **Rule 2:** prefer exhaustive matching (Swift `switch`, Kotlin `when`, Rust `match`, C# switch expressions, TS `switch` + `never` check).
- **Rule 4:** empty collection, `false`, `0` (only when truthful), or a result/enum type (`Result<T,E>`, discriminated union, `(value, err)` in Go). Never replace null with a value that lies.
- **Rule 6:** move logic to a pure standalone function, extension method, another constructor, or a small dedicated type that tests can reach.
- **Rule 9:** `sealed` (C#), `final` (Swift/Java/PHP/Dart), non-`open` (Kotlin), `@final` (Python).

## Workflow
1. Identify the scope (file, diff, PR) and language; read project conventions first.
2. Go through the code rule by rule using the table above.
3. Report findings grouped by rule, most impactful first. For each: location, what's wrong, why it matters, and a short suggested fix (before/after snippet when helpful).
4. Note any rule you intentionally did not apply and why (rule 10).
5. If asked to refactor: make the changes, preserve behavior, keep each change minimal, and run existing tests if available. Don't mix unrelated cleanups into the refactor.
6. End with a one-line summary: number of findings per rule and the top fix to do first.