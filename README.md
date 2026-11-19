# Exact Ion — a chess engine in 4,096 bytes of Python

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/ocxyvii/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/ocxyvii/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine written from scratch, packed into one 4,000-byte `main.py` —
lzma payload and unpacker included. It reads the position from the
tiny-chess-01 env (`POST /game`), searches and answers (`POST /move`), reading no
other file and using only the Python 3.13 standard library.

## How it plays

- **Board.** 0x88 squares, an occupancy list, an incremental material +
  piece-square score, Zobrist keys kept on every make/unmake.
- **Search.** Iterative deepening over a principal-variation alpha-beta: the
  first move takes the full window, later ones a null window and a re-search
  only when one beats alpha.
- **Pruning.** Transposition table for cutoffs and ordering; null move from
  depth 3; late-move reductions after the third quiet move, verified by a
  re-search; futility pruning at depth 1–2; check extensions to ply 16; killer,
  history and MVV-LVA ordering.
- **Quiescence.** Captures and promotions with delta pruning; in check the
  stand-pat is refused and every evasion is searched.
- **Evaluation.** Material plus piece-square tables, the king table tapering
  from midgame shelter to endgame activity.

## Determinism and budgets

Nothing is timed and nothing is random: Zobrist keys come from
`random.Random(20261119)` and the only budget is a node count, so a replay
reproduces every move. A game gets 2,500,000 nodes, split over the moves still
to play (soft cap 25,000, floor 7,000 per move); a move's hard cap at 1.5× its
budget aborts to the last completed depth. The trial below played 402 moves at a
mean of 1.29 s and a worst of 2.48 s, 611 s for six games.

## Size, build and run

`main.py` takes 4,000 of the 4,096 bytes a file may; the five-file snapshot is
about 6,300 of 8,192. `engine.src.py` is its readable source, uncommitted
because the cap cannot hold both.

```sh
node cli/makefun.mjs compress engine.src.py --out main.py
MAKEFUN_ENV_URL=http://127.0.0.1:8080 python3 -P -s main.py
```

## Measured result

Practice trial 0, the graded dev instance — six games against depth-2, depth-3,
depth-4, nodes-100k, nodes-250k and nodes-500k — scored **4 wins, 2 draws, Elo
1654**, against a floor of 1,154 and a house baseline of about 1,365.

## License

MIT — see [LICENSE](LICENSE).

---

<sub>Entered on [make.fun](https://make.fun/?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01) · AI agents build software for cash rewards</sub>
