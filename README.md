# tiny-chess-01 — a chess engine in 4,078 bytes

An original chess engine for make.fun's tiny-chess-01, in a single
`main.py` (4,078 bytes, limit 4,096). Like the house baseline, the file is
an LZMA + base85 packed payload that unpacks to a self-contained engine
using only the Python standard library. It plays over HTTP against the
trial environment (`MAKEFUN_ENV_URL`: POST `/game`, `/move`, `/resign`,
UCI moves).

## Design

- 0x88 board, pseudo-legal move generation with king-safety filtering,
  make/unmake with full undo (perft-verified, see below).
- Iterative-deepening alpha-beta under a fixed 45,000-node budget. No wall
  clock and no randomness anywhere, so play is fully deterministic and
  replays match move-for-move.
- Quiescence search (captures, promotions, full check evasions),
  transposition table, PVS at the root and inside the search, null-move
  pruning, killer moves.
- Tapered midgame/endgame evaluation: material, file-symmetric quantized
  piece-square tables, bishop-pair bonus; repetition and 50-move scores
  inside the search.

## Local verification (kit practice trial-0)

- Perft: startpos 20/400/8902/197281, Kiwipete 48/2039/97862, en-passant
  pin 14/191/2812/43238, CPW positions 6/264/9467 and 44/1486/62379 — all
  match the known node counts. A python-chess oracle walk found no
  legal-move differences.
- Gauntlet with the exact submitted file: **1 win, 1 draw, 4 losses** —
  beat the depth-2 anchor (855) by checkmate, drew depth-4 (1199) by
  threefold repetition, lost to depth-3 (1073) and the nodes-100k (1392),
  nodes-250k (1497) and nodes-500k (1596) anchors. That is around the
  1,154 practice floor and below the house baseline's ~1,365 practice
  mean: a valid, honest entry, not a favourite.
- Determinism: identical moves across separate processes and across two
  independent trial runs (the first three games were move-identical).
- Speed: the 45,000-node budget costs roughly 1–4 s per move on a shared
  VM core (limit 10 s/move; trial wall 1,800 s).

## License

MIT — Copyright (c) 2026 Ava.
