# site-generator-speed-01

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/cryptokom-tech/site-generator-speed-01/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/cryptokom-tech/site-generator-speed-01/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01)

A faster drop-in for make.fun's own makefun-site 1.0.0: `solve.js` answers
exactly as makefun-site does on every case — the same front matter, template
dialect, Markdown subset, slug rule, link resolution and build report, byte
for byte — and runs about 2× faster than the reference on the practice set
(about 2.6× on `run()` CPU alone; the rest is the harness's fixed JSON
parse/stringify). `main.ts` is the harness driver: it reads every case from
`/input/cases.json`, answers each with `run`, and writes `/output/answers.json`.

## The program

- `solve.js` (dependency-free ES module) exports `run(input)`, returning the
  output tree `{path: text}` for one case's `{site, templates, files}`. Same
  pipeline as makefun-site, faster primitives:
  - Markdown inline rendering finds the next special character with the
    regex engine and slices plain runs whole, instead of one character at a
    time; a string with no special characters is returned as-is.
  - Block lexing dispatches on character codes instead of running a regex
    per line; fence bodies are escaped in one pass.
  - Slugs are cached per case (titles, tags and headings repeat across a
    site); pure-ASCII text skips Unicode normalisation.
  - Word counts are scanned by hand instead of allocating a match array.
  - Templates are compiled to JavaScript functions (`new Function`) with
    their includes inlined — the include graph is a DAG, so inlining
    terminates and the include depth is known statically. Loop bindings
    resolve at compile time; a cyclic include falls back to the included
    tree interpreter. Nothing is cached from one case to the next: every
    cache and every compiled function is created inside `run()`.
- `main.ts` (Node 22) runs `node /bot/main.ts`: reads `/input/cases.json`,
  answers each case alone with `run`, writes `/output/answers.json`. It
  turns V8's incremental marking off, as the house baseline's driver does.

## Verification

- Practice set: 3/3 cases identical to makefun-site 1.0.0 (kit scorer).
- 150 further cases drawn from the kit's own generator with fresh seeds:
  0 mismatches against the reference.
- 55 hand-made edge cases (cyclic and self includes, 10-deep include
  chains, a loop variable named `loop`, every template parse error, Markdown
  and front-matter torture, slug collisions, link resolution, empty sites):
  0 mismatches against the reference.

## License

MIT. See [LICENSE](LICENSE).
