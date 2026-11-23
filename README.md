# Build a static site faster — makefun-site, but 3x quicker

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/Sapriyudin/site-generator-speed-01-v2/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/Sapriyudin/site-generator-speed-01-v2/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own static site generator, makefun-site 1.0.0.
`solve.js` returns byte-for-byte the same output tree for every case — same front
matter rules, same Markdown subset, same template dialect, same report — in about a
third of the CPU time.

Built for the make.fun bounty
[Build a static site faster, 22 November 2026](https://make.fun/bounty/site-generator-speed-01-run-20261122)
($100, 10 places × $10).

## Local grade (practice set, median of 7 runs, node v24.22.1, this machine)

| | CPU time, 3-case practice set |
| --- | --- |
| makefun-site 1.0.0 (house baseline) | ~540–660 ms |
| this entry | ~170–190 ms |
| **speedup** | **~3.2–3.4x** |

Correctness: identical output on all 3 practice cases, plus 9 extra cases drawn from
the competition's published generator with three other seeds, plus a 60,000-string
adversarial differential test of the inline Markdown renderer against the reference.

## How it is faster

- **Fused Markdown** — lexing and rendering in one pass (no token array). Plain runs
  are sliced and escaped in bulk, the next special character is found with `indexOf`,
  and escapes, strong, emphasis and links resolve exactly as the reference does
  (including backslash-escape parity).
- **Cheap wins across the pipeline** — skip the `\r` normalisation when a file holds
  no `\r`; front matter parsed by scanning line starts (the body is sliced out, never
  split and re-joined); `slugify` / `longDate` cached per case; each source's card
  computed once (the reference computes it twice for posts); template rendering writes
  into a shared array instead of `out +=` per node.
- **Word count without a regex match array** — a manual scan over the exact JS `\s` set.

Deliberately not used: compiling templates to functions (measured: no gain — the
codegen cost cancels the interpreter saving) and fragment caching (small gain, real
correctness risk).

## Files

- `solve.js` — the entry: a self-contained ES module, no dependencies, exports
  `run(input)`. All per-case state lives inside `run()`; nothing is carried from one
  case to the next.
- `main.ts` — the house driver, unchanged: reads `/input/cases.json`, answers each
  case with `run`, writes `/output/answers.json`.
- `package.json` — `{"type": "module"}`.

## Run it

```sh
node main.ts   # with MAKEFUN_INPUT_DIR / MAKEFUN_OUTPUT_DIR set, or /input + /output
```

Node 22, standard library only, no network, no build step.
