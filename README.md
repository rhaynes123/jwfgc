# jwfgc

**Just Write Freaking Good Code** — a set of opinionated guidelines by [Richard Haynes](https://medium.com/@rahayn59/the-jwfgc-paradigm-38a2cd8d92d8) for code that is easy to read, easy to write, and not slow.

This repo packages those guidelines as Claude Code skills so an agent will apply them when writing or reviewing code.

## Skills

- **`jwfgc-write-code/`** — invoked when generating new code; applies the ten rules as it writes.
- **`jwfgc-review-code/`** — invoked when reviewing or refactoring existing code; reports findings rule-by-rule and offers fixes.

Each folder contains the `SKILL.md` the agent loads.

## The ten rules

1. Name data types as nouns, functions as verbs.
2. Favor `switch` / `match` over chains of `if … else if`.
3. Avoid `else` — use guard clauses and early returns.
4. Avoid returning null/nil/undefined — use empty collections, `0`, `false`, or a result type.
5. Abstract only to consolidate real reuse, not "just in case."
6. Avoid private helpers as dumping grounds — prefer pure, testable functions.
7. Favor enums over booleans when a decision may ever grow a third state.
8. Favor enums over magic numbers.
9. Seal / `final` types that aren't designed to be subclassed.
10. Don't do anything you don't agree with — these are guidelines, not laws.

Full rationale and language-specific notes live inside each skill file.

## Install

Copy (or symlink) each skill folder into your Claude Code skills directory:

```bash
ln -s "$PWD/jwfgc-write-code"  ~/.claude/skills/jwfgc-write-code
ln -s "$PWD/jwfgc-review-code" ~/.claude/skills/jwfgc-review-code
```

Claude Code picks them up by `description` on the next session.

## Model invocation

Both skills ship with model invocation enabled — Claude will auto-load them when the `description` matches (writing new code, reviewing existing code). This matches the intent: JWFGC applied by default, with rule 10 as the escape valve when a project's conventions say otherwise.

If you'd rather invoke them explicitly (e.g. you want `jwfgc-review-code` to fire only when you ask for it specifically, not on every "review this" request), add this to the skill's frontmatter:

```yaml
disable-model-invocation: true
```

The skill is then reachable only by explicit reference or slash command.

## Build (optional)

To produce distributable `.skill` archives (zip files containing the `SKILL.md`):

```bash
zip -r jwfgc-write-code.skill  jwfgc-write-code/SKILL.md
zip -r jwfgc-review-code.skill jwfgc-review-code/SKILL.md
```

`*.skill` is gitignored — treat the archives as release artifacts, not source.

## License

See [LICENSE](LICENSE).
