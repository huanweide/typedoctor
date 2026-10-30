**中文** · [README.md](./README.md)

<!-- badges -->
[![Node](https://img.shields.io/badge/Node.js-≥18-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Zero-Deps](https://img.shields.io/badge/dependencies-zero-0f9d58?logo=npm&logoColor=white)](https://www.npmjs.com/)
[![License](https://img.shields.io/github/license/huanweide/typedoctor)](LICENSE)
<!-- /badges -->

# typedoctor

A zero-dependency type-health CLI. It scans TypeScript / JavaScript projects for "type-discipline smells" — explicit `any` annotations, type casts, directive comments that bypass type-checking, and double casts — then reports a 0–100 health score that drops straight into CI as a type-discipline gate.

> In one line: code riddled with `any`, casual `as any`, and `@ts-ignore` blanket-suppressions quietly hollows out your type system. typedoctor finds all of it with a single command.

Zero-dependency single file (Node built-ins only). Works out of the box — no network, never executes your code.

## Who should use it / Who shouldn't

- **Use it if** your TS/JS team wants to quantify "any density", wants to stop "type rot" from spreading via pre-commit / CI, or runs a small project with no ESLint set up and wants zero-cost onboarding.
- **Skip it if** you already enforce `@typescript-eslint/no-explicit-any` under full strict mode (overlapping value), or your project is plain JavaScript with no type annotations at all (the `any` check has little to act on).

## Why it exists

- `tsc` is a compiler: slow, and it reports "does it compile", not "how dense is the type discipline". A 5000-line file stuffed with 200 `any`s compiles fine while the type system is effectively gone.
- `@typescript-eslint/no-explicit-any` needs ESLint installed and configured; without it the team is completely blind.
- typedoctor does exactly one thing: a static check-up of "type-discipline density" with a health score + CI gate. Wire it into pre-commit / CI to catch type rot before it spreads.

## Install / Use

```bash
# Run directly (no install, Node 18+)
node typedoctor.js

# Or link globally
npm link   # then just: typedoctor

# Target a directory
typedoctor --root ./packages/core
```

## Detection dimensions

| Rule id | Severity | Meaning |
| --- | --- | --- |
| `explicit-any` | Medium | Explicit `any` type annotations like `: any` / `<any>` (holes in the type system) |
| `any-cast` | High | Type casts like `as any` / `as readonly any` (erase type-checking outright) |
| `ts-ignore-comment` | High | `@ts-ignore` / `@ts-nocheck` / `@ts-expect-error` directives that bypass type-checking |
| `double-cast` | Medium | `as unknown as` double casts (bypass type protection) |

> Notes: `.vue` / `.svelte` files are scanned only inside `<script>` blocks (avoids mis-flagging `:prop` template bindings); in `.tsx` / `.jsx`, `<any>` is a JSX element, not a generic, so the `<any>` generic check is skipped (only `: any` is reported); `.d.ts` declaration files legitimately use `any` and are skipped entirely.

## Health score

Out of 100. Every 1000 lines tolerates 6 issue-equivalents (= 12 weighted points; High×3 / Medium×2). Beyond that, each extra weighted point costs −3, floored at 0. So a few `any`s in a small file cost nothing, while `any` flooding in a large file crashes the score.

## CI gate

```bash
# Any High issue (as any / ts-ignore) fails (exit 1)
typedoctor --fail-on-high

# Fine-grained control
typedoctor --max-high 0 --max-medium 3 --max-any 10 --max-issues 15 --min-score 85

# Pipeline parsing (JSON output)
typedoctor --json
```

Exit codes: `0` pass, `1` gate failed, `2` argument/path error.

## Related tools

typedoctor is the **9th source-layer axis — type health** of the "code-health family", completing the matrix with:

- **devdoctor** — dependency / license / secret / cycle check-up
- **testlite** — test hygiene
- **debtlens** — tech-debt density
- **a11ydoctor** — Web a11y static scan
- **awaitscan** — async performance anti-patterns
- **cycscan** — structural complexity
- **secscan** — security anti-patterns
- **dupscan** — duplicate code
- **typedoctor** — type-discipline health (this tool)

The git-layer set (repodoctor / docdoctor / commitdoctor / reldoctor) covers repo / docs / commit / release governance.

## Design discipline

- Zero dependency: only Node built-ins `fs` / `path`, no npm packages.
- Safe: read-only filesystem, no network, never executes your code.
- Robust: auto-skips `node_modules` / `.git` / `dist`, `.d.ts` files, and files over 5MB; non-git dirs / bad JSON degrade gracefully instead of crashing.
- Low false positives: a character-level tokenizer strips comments and strings (regex-literal aware); `.vue`/`.svelte` extract only `<script>` blocks; `.tsx`/`.jsx` skip `<any>` JSX mis-reads.
- Gate guardrails: every threshold is validated with `Number.isFinite`; non-finite values exit 2 immediately rather than silently passing.

## Roadmap

Rule set is stable. Potential future additions: richer cast-context detection and per-directory scoring rollups. No breaking changes planned.

## Contributing

Issues and pull requests are welcome. Keep it single-file and zero-dependency. It dogfoods itself on its own source.

## ⭐ Star

If typedoctor ever stopped "type rot" from spreading in your codebase, a Star ⭐ helps others find it: https://github.com/huanweide/typedoctor

## Author

[huanweide](https://github.com/huanweide)

## License

MIT
