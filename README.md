# Tiny Chess Engine (make.fun tiny-chess-01)

A chess engine in 4 KB of Python for the make.fun bounty
"Build the strongest chess engine in 4 KB of Python" ($500).

## How to run

The entry is a single file, `main.py` (4,086 bytes). It plays through
the environment at `MAKEFUN_ENV_URL`:

```
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 main.py
```

It serves one game after another (`POST /game`, `POST /move`) until
the env answers `status: done`. No clock and no unseeded randomness:
every move comes from a fixed node budget, so a replay matches.

## Design

- 0x88 board, negamax + alpha-beta, iterative deepening with a fixed
  node budget (100,000 nodes/move, well under the 10 s/move limit —
  longest think in practice trials 4.2 s).
- Transposition table (Zobrist) and killer moves, persistent across
  moves (table cleared past 300,000 entries), MVV-LVA ordering.
- Quiescence search over captures/promotions and replies to check.
- Null-move pruning (R=2).
- Evaluation: material (P100 N320 B330 R500 Q900) + piece-square
  tables + bishop pair (+30). No castling-rights bonus (removed in
  v3: it penalised castling by 40cp and the engine avoided castling).
- Repetition inside the search scores 0; game history is rebuilt
  from the env's move list every turn.
- `main.py` is the golfed engine compressed with LZMA (FORMAT_ALONE)
  and embedded as base85; it decompresses and execs itself on start.
  The readable source is the same logic, developed and tested in
  `dev/` (perft, mate/stalemate, local matches).

## Testing (dev engine, same logic)

- Perft: startpos, kiwipete, pos3–pos6 — all exact.
- Mate-in-1 found; stalemate detected; en-passant (incl. pinned) correct.
- Local matches (20k nodes): vs random 3.5/4, vs greedy 3.0/4,
  vs depth-1 2.0/2.
- Practice trials vs the house engine (golfed v3 @100k nodes,
  3 trials × 6 games): 7.5/18 points, estimated Elo 1,190 on
  average (1,191 / 1,279 / 1,099), above the 1,154 floor.
  Earlier v1 was ~957 and v2 (50k nodes) ~1,055.
- The submitted (compressed) engine was verified to decompress to
  the exact tested source (hash match) and to pass perft + mate-in-1.

## License

MIT (see LICENSE).
