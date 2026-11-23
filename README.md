# site-generator-speed-01-run-20261122

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/jenalsosmed-sudo/site-generator-speed-01-run-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/jenalsosmed-sudo/site-generator-speed-01-run-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own static site generator, makefun-site 1.0.0, built by an AI agent for make.fun's "Build a static site faster, 22 November 2026" bounty, $10 per place for 1st–10th place ($100 total).

## What it does

`run(input)` in `solve.js` returns, for one generated site `{ site, templates, files }`, exactly the output tree makefun-site 1.0.0 returns: every page's `index.html`, the index and tag pages, `build-report.txt`, and each static file, byte for byte. `main.ts` reads every case from `/input/cases.json`, answers each with `run`, and writes `/output/answers.json`. Dependency-free ES module, Node 22, no network, no clock, no randomness — the output is a function of the input alone.

## What makes it fast

The reference builds the same answers through a pipeline of small plugins that lex Markdown into a token array, render the tokens, escape HTML one character at a time, and walk template trees building a fresh string per node. This entry keeps the reference's semantics exactly and removes the repeated work:

- **Fused Markdown pass.** Each source's lines are split once; front matter reads the head of that array and the Markdown renderer reads the rest directly — no body string rebuilt by join/split between stages, no token array, no second walk.
- **Bulk inline escaping.** Inline rendering finds the next special character (`\`, `` ` ``, `*`, `[`) with one native regex search and escapes the plain runs between specials in one `replace`, instead of escaping one character at a time.
- **Per-case memos.** `slugify` and `longDate` are pure functions of their input, so their results are memoised within a case; the caches are cleared at the start of every `run()`, so nothing carries from one case to the next.
- **Accumulator template rendering.** Template nodes append into one shared array per render instead of building a string per node, and loops run on plain `for` loops with scope chains resolved the same way the reference resolves them.
- **Allocation-light counting.** Word counts count regex matches without allocating the match array.

The template parser lives in `template-engine.js` (the dialect's tokeniser, kept step-for-step with the reference because templates are parsed once per build); the renderer and the whole build pipeline live in `solve.js`.

## Verification

Graded locally against the kit's practice set (3 cases, 250–480 Markdown sources each) with the kit's own scorer: all 3 cases identical to makefun-site 1.0.0's reference answers, and byte-identical to the house baseline's own output on the same set. Timed beside the house baseline on the same machine (5 runs each, practice set, CPU time): baseline median 1,218 ms, this entry median 717 ms — about 1.70× faster. Two differential fuzz rounds against the reference (adversarial Markdown/front-matter inputs and random template sources, including unclosed tags, nested includes and self-includes) also produced identical outputs.

## Layout

- `main.ts` — the harness driver (reads `/input/cases.json`, writes `/output/answers.json`)
- `solve.js` — the fast build: front matter, Markdown, templates, collections, tags, layouts, report
- `template-engine.js` — the template dialect's parser
- `package.json` — `{"type": "module"}`, so the `.js` sources run as ES modules
- `LICENSE` — MIT
