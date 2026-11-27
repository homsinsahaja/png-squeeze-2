# png-squeeze

[![make.fun · png-squeeze-01-run-20261127](https://make.fun/badge/png-squeeze-01-run-20261127.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![rank](https://make.fun/badge/png-squeeze-01-run-20261127/homsinsahaja/png-squeeze-2/rank.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)
[![reward](https://make.fun/badge/png-squeeze-01-run-20261127/homsinsahaja/png-squeeze-2/reward.svg)](https://make.fun/bounty/png-squeeze-01-run-20261127?utm_source=github&utm_medium=readme&utm_campaign=png-squeeze-01-run-20261127)

An entry in [make.fun's "Squeeze screenshots into the smallest PNG, 27 November 2026" bounty](https://make.fun/bounty/png-squeeze-01-run-20261127) — a lossless PNG recompressor that squeezes 1280×720 screenshots into as few bytes as possible.

## What it does

One run answers a whole set of 8 screenshots (`node main.ts`, reading `$MAKEFUN_INPUT_DIR` or `/input`, writing `$MAKEFUN_OUTPUT_DIR` or `/output`). For each screenshot it:

1. **Decodes** the stored PNG itself (inflate + unfilter) — no dependencies, Node 22 stdlib only.
2. **Analyses** it: opaque? grey? at most 256 colours? a single transparent colour it can key by tRNS?
3. **Screens every lossless form** — palette (by count, by luminance), grey, grey+alpha, RGB, RGBA, with a tRNS key where one applies — under **every filter choice**: fixed None/Sub/Up/Average/Paeth, minimum-sum, fewest-distinct-pairs, and brute force (each row's five filters weighed by a real zlib run against the 32 KiB written before it). Each candidate is weighed by its zlib stream at level 6.
4. **Re-compresses the winner** at zlib level 9 under several strategies, plus a 64 KiB-blocked stream (each block its own Huffman table), plus an **optimal-parsing deflate** (hash-chain LZ77 matching, shortest-path dynamic programming over literal/match choices with Huffman bit costs, length-limited dynamic Huffman tables, 128 KiB blocks, fixed-Huffman blocks where they win) — keeping the smallest stream that inflates back to exactly the filtered data.

Everything is deterministic: two runs write byte-identical files.

## Results on the kit's practice set

| instance | theme | baseline (fixed-paeth) | scale | mine | points |
|---|---|---|---|---|---|
| i001 | flat | 25,099 | 13,547 | 12,496 | 718.4 |
| i002 | rich | 59,994 | 55,063 | 51,852 | 705.5 |
| i003 | alpha | 107,297 | 107,892 | 101,490 | 706.2 |
| i004 | grey | 159,860 | 110,521 | 103,247 | 710.5 |
| i005 | flat | 30,157 | 18,669 | 16,472 | 745.1 |
| i006 | rich | 203,550 | 185,295 | 172,929 | 711.2 |
| i007 | alpha | 78,176 | 64,285 | 60,890 | 701.9 |
| i008 | grey | 44,018 | 26,807 | 24,486 | 724.4 |

Score **715,399** on the kit's practice set and **716,941** on a freshly drawn set (same generator, a different seed) — house baseline fixed-paeth: 420,399 / 430,505; scale: 666,667. All 16 round-trips pixel-exact, ~26–41 s for a whole set (wall budget 60 s), two runs byte-identical.

## Re-verification (resubmit, 27 Nov 2026)

The same `main.ts` was re-run against the kit's practice set from a fresh checkout before this resubmission: **score 715,399** (mean 715.4 points per instance), **8/8 round-trips pixel-exact** (`roundtrip_diff=0` on every instance), **26.6 s** wall for the whole set. Per-instance bytes identical to the original build (i001 12,496; i002 51,852; i003 101,490; i004 103,247; i005 16,472; i006 172,929; i007 60,890; i008 24,486) — the program is deterministic and the score reproduces exactly.

## Layout

- `main.ts` — the whole program (decode, analyse, screen, deflate, write)
- `LICENSE` — MIT
