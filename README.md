# Ember v8 — tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-11-repo/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-11-repo/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A 4 KB Python chess engine for the [tiny-chess-01](https://make.fun/bounty/tiny-chess-01) bounty on make.fun: one `main.py`, at most 4,096 bytes, Python 3.13 standard library only, deterministic, fixed node budget, no clock.

## Why v8 exists

v7 scored 1,521 locally but 997 on the server. v7 is deterministic, so identical
play should give identical results — the gap must come from the clock: v7 spent
up to 4.1 s per move locally (40,000 nodes) on this VM, so on the server's
single CPU its slowest moves approached or passed the 10 s move limit and were
forfeited. v8 is built for the server: a smaller node budget with a faster
per-node implementation, so the worst move stays far inside the limit.

## Design

- **Board**: 0x88, signed pieces, Zobrist hashing, incremental tapered eval.
- **Search**: PVS, persistent transposition table, null-move pruning (R=2),
  late-move reductions, check extensions, futility pruning at depth 1–2,
  in-search repetition draws, two killers per ply, history heuristic.
- **Quiescence**: captures/promotions, check evasions, MVV-LVA ordering,
  delta pruning. No SEE.
- **Eval**: incremental tapered material + piece-square tables (separate
  middlegame/endgame tables), bishop pair, tempo. `ev()` is O(1).
- **Budget**: 40,000 nodes per move, iterative deepening 1–8. Worst move
  ~3.7 s locally (mean ~1.6 s). No wall clock anywhere; HTTP posts retry
  connection failures up to 4 times (never HTTP errors).

`engine.py` is the readable source; `main.py` is the same source minified
and packed with lzma (see `build8.py` in the author's work notes).

## Local practice (fresh level-19 kit; trial-0 is the server instance)

- trial-0: **Elo 1,545** (4W 1D 1L, spare 216) — reproduced 3x, byte-identical.
- trial-1: Elo 1,364 (3W 1D 2L). trial-2: Elo 1,451 (3W 2D 1L). Mean 1,453.
- Perft depth 3: 6/6 standard positions correct.
- Floor 1,154; trial-0 sits 391 above it.
