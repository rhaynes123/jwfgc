---
name: "jwfgc-write-code"
description: "Use when writing or generating new code in any language."
---

# Writing code the JWFGC way

JWFGC (Just Write Freaking Good Code), by Richard Haynes, is a set of guidelines for code that is easy to read and easy to write **without quietly degrading performance**. Apply the ten rules below whenever you write new code. Readability and performance are both goals; never trade one away silently.

Before writing, check the project's existing conventions (linters, style guides, CLAUDE.md, surrounding code). Rule 10 outranks everything: where the codebase or the user has deliberately chosen otherwise, follow them and do not fight it.

## The rules

### 1. Name data types as nouns, functions as verbs
- Types, classes, structs, enums, records: nouns or noun phrases (`Invoice`, `PaymentStatus`, `UserRepository`).
- Functions and methods: verbs or verb phrases describing the action (`calculateTotal`, `sendReceipt`, `loadUser`).
- Booleans and predicates read as assertions (`isExpired`, `hasAccess`).
- For finer naming detail, follow the spirit of the Swift API Design Guidelines: clarity at the point of use beats brevity.

### 2. Branch on one value's values → `switch` / `match`
If you are testing the same discriminant (variable, type, enum tag) across multiple branches, use `switch` / `match` / pattern matching. A chained `if / else if` on one discriminant is a bug smell — the compiler can't check exhaustiveness and a new case silently falls through. Prefer forms the compiler enforces: Swift `switch`, Kotlin `when`, Rust `match`, C# switch expressions, TypeScript `switch` with a `never` default.

### 3. Avoid the `else` keyword
Use guard clauses and early returns/exits so the happy path stays flat and unindented.
```
// instead of
if (user != null) { if (user.isActive) { process(user) } else { return error } } else { return error }

// write
if (user == null) return error
if (!user.isActive) return error
process(user)
```
(Adapted from Jeff Bay's Object Calisthenics.)

### 4. Avoid returning null
Do not return `null` / `nil` / `undefined` / `None` as an ordinary outcome — "if it can be null it will be null." Design it out up front:
- Collections → return an empty array/list/map.
- Counts/amounts → `0` **only when zero is literally the right answer**. `countFailures()` returning `0` for "no failures" is truthful. `getPrice()` returning `0` for "price unknown" is a lie — use `Result<Price, PriceUnavailable>` or an `Option<Price>` instead.
- Yes/no lookups → `false` where false is the honest answer (not where "we don't know" is).
- Richer outcomes → a result type, tuple, or enum that says what happened (`Result<T, E>`, `Found(value) / NotFound`, `(value, error)` in Go, a discriminated union in TypeScript).

### 5. Abstract only to consolidate and reuse
Create an interface, base class, generic, or design pattern only when it consolidates logic used in more than one place, or there is a concrete, present need. If logic is used once, keep it inline. Don't add architecture patterns "just in case."

### 6. Avoid private functions as dumping grounds
Don't hide unrelated logic in private helpers. If a helper is genuinely needed:
- Make it a pure function that works only from its input parameters.
- Consider whether it belongs somewhere testable instead: an extension method, a standalone/module-level function, a different constructor, or a small type of its own.
Private functions are hard to test directly — prefer placing logic where tests can reach it.

### 7. A boolean that encodes a business mode is an enum waiting to grow
If a boolean names a business decision (`isExpress`, `isPremium`, `useLegacyFlow`), model it as an enum from the start (`DeliveryMethod.standard / .express / .pickup`). Enums store as numbers like booleans do, cost nothing extra, and leave room for the third state that always eventually arrives. Boolean parameters that switch behavior at the call site (`render(true)`) are the strongest signal — pass an enum.

### 8. A numeric literal with meaning is an enum member
If a number carries a fixed meaning (`3` = high priority, `200` = HTTP OK, `-1` = sentinel), replace it with an enum (`Priority.high`, `HttpStatus.ok`). Enums are compile-time known: you get the name, the reuse, and the exhaustiveness checks at zero runtime cost.

### 9. Prevent inheritance when it isn't needed
If a type isn't designed to be subclassed, seal it: `sealed` (C#), `final` (Swift, Java, PHP, Dart), leave Kotlin classes non-`open`, `@final` (Python typing). This states intent and lets the compiler devirtualize calls. Only open a type for extension when a subclass actually exists or is planned.

### 10. Defer to the project and the user
Established project conventions win. Explicit user instructions win. If a rule above would clash with a documented style guide, framework requirement, or something the user asked for, follow the project or the user — and name what you're deferring to in one line (`// jwfgc: using isActive — matches existing User fields`).

> **jwfgc:** "I don't personally agree with this rule here" is too easy to invoke under pressure and becomes an escape hatch for every rule above. Deferring only counts when there is a concrete thing on the other side of the trade — a convention, a framework constraint, a user preference. Name it or apply the rule.

## Workflow
1. Read the surrounding code and project conventions; note any rules the codebase already contradicts on purpose.
2. Write the code applying rules 1–9.
3. Self-check before finishing:
   - Any `else` that could be an early return?
   - Any if-chain on one value that should be a switch?
   - Any function that can return null/nil/undefined?
   - Any boolean flag or magic number that should be an enum?
   - Any abstraction used in only one place?
   - Any private helper that isn't pure or should live somewhere testable?
   - Any non-sealed type with no subclasses?
   - Are types nouns and functions verbs?
4. If you deliberately broke a rule, mention it in one line with the reason (rule 10).