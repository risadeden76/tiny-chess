# tiny-chess

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/risadeden76/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/risadeden76/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

An original 0x88 chess engine in Python — alpha-beta search, fixed node budget, phase-interpolated king table — packed into 3,536 bytes.

## Run it

```sh
python3 main.py
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=ekkx10fx&utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>

## Files

- `main.py` — the entry, **3,536 bytes** (under the 4,096 cap). It is an
  lzma-packed form of the readable source; it decompresses and execs itself
  at import. Stdlib only, no other files read.
- `LICENSE` — MIT.

## Engine

- Bytearray 0x88 board; the side to move is always White via a rank-flip
  rotation, so there are no colour branches and castling squares are
  constant. Positions are rebuilt by replaying the game moves from the
  start position (no FEN parser).
- Alpha-beta (PVS) with iterative deepening and a **fixed node budget per
  move (deterministic — no clock, no randomness)**: 200,000 nodes for
  plies < 80, 120,000 for plies 80–139, 60,000 after that.
- Transposition table, 2 killer moves per ply, history heuristic,
  MVV-LVA capture ordering, null-move pruning, quiescence over
  captures/promotions with delta pruning (all replies when in check).
- Evaluation: material + formula piece-square tables (least-squares fit
  to a standard table set) + bishop pair, with a **phase-interpolated
  king table** (middlegame back-rank safety blending into endgame
  centralisation).
- Protocol: talks to the make.fun env over HTTP (`/game`, `/move`,
  `/resign`), replies in UCI, filters moves against the env's legal list.

## Validation

- Perft matches python-chess on startpos 20/400/8902/197281 and Kiwipete
  48/2039/97862 — for both the raw source and the packed `main.py`
  (identical best move, score and node count at depth 3).
- Practice trials (local env server, 6 games each): trial-0 **4W 1D 1L,
  Elo 1,545**; trial-1 3W 0D 3L, Elo 1,279; trial-2 3W 1D 2L, Elo 1,364.
  Mean **1,396**, above the 1,154 practice floor and the 1,365 sunfish
  baseline mean. Wall time 843–1,007 s (limit 1,800 s); max move ~3.1 s
  (limit 10 s).
- A 240k node budget was tried and REVERTED: it scored only 3.0/6
  (Elo 1,279) on trial-0 vs 4.5/6 at 200k.
- Determinism: same position/budget always returns the same move.
