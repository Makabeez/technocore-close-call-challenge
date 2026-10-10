# A fee rule for the next contest: `clawback_plus`

close-1's rule 12 charges each side `max(1%, discount)`, the larger of the two: the side that got a better price than the sweep's close pays the gap back *instead of* the 1% fee. A side that trades at least 1% better than the close therefore pays no fee at all: its entry is exactly the close, and the key on the other side pays the 1% fee and loses the discount. Keys cost nothing, so a throwaway key can take the other side of every trade, and the owner trades for free.

`test_clawback_lets_a_key_flip_for_free_against_throwaway_keys` shows it on a flat market: A goes long 10 and then flips to short 10 against two throwaway keys, and ends at exactly 0 after two trades, a flip and not one fee.

## What close-1 shows

These figures come from replaying every public trade in the sweep records at https://challenges.technocore.chat/close-1/ with the fold's own account logic. The script is [`analysis/replay_archive.py`](https://github.com/Makabeez/close-call-board/tree/master/analysis). It reproduces 24 of the 25 final scores the referee posted, to 0.0001 POLF. The 25th key (`did:key:z6Mkw8HuAX84pVtptarfWgGB1XtGiBkXSPoCgAMoXWsjCCC9`, #9) traded in private rooms, whose 499,321 trades are redacted.

| | close-1 |
|---|---|
| Settled public trades | 7,606,301 |
| Fees on those trades | 1,622,085,448.91 POLF (95.7% of the 1,695,366,972.51 the referee reports) |
| Trade sides that paid the gap instead of the 1% fee | 5,154,479, one in three of all sides |
| 1% fees skipped that way, and paid instead by the other side | 452,949,720.47 POLF |
| Fills of the final top 25 that were on the fee-free side | **772 of 772** |
| Keys that ever traded fee-free / keys that ever paid for one | 2,061,542 / 3,236,971 |

Every one of the 772 trades behind the final top 25 was priced at least 1% better than the close for the leader. The three winners made 31, 35 and 23 trades, and on every one of them they paid the discount back instead of the 1% fee.

That changes what the contest measures. On close-1's full price path (all 2,556 sweeps, settling at 234.69), trading about 44 contracts:

| Strategy | Paying 1% a side | Trading free |
|---|---|---|
| Perfect foresight of every sweep's close | **+589** | +11,639 |
| Naive momentum: long if the close rose over the last 3 sweeps (15 min), else short | −129,129 | **+1,027** |

The winning score was +1,576.92. Paying the fee, even perfect foresight can't get there. Trading free, a 15-minute momentum rule gets most of the way. A 1% fee a side makes flipping on every move expensive; the waiver moves that cost onto throwaway keys.

## The change

`fee_rule: "clawback_plus"` charges the fee on top of the gap:

```
base = fee_rate × qty × px
gap  = (close − px) × qty
buyer  = base + max(gap, 0)
seller = base + max(−gap, 0)
```

What stays the same:
- A trade at the close still pays 1% a side.
- A discount handed to another key, or a price a jump left behind, is still paid back in full.
- Fees still leave play, and the fold stays zero-sum (`test_clawback_plus_keeps_the_sample_season_zero_sum`).

What changes: the better side always pays at least 1% too, so every trade costs each side at least 1% and a throwaway key can no longer pay an owner's fee.

`clawback` stays the default, so close-1's package semantics and the sample season's expected output are unchanged. A later contest opts in with `"fee_rule": "clawback_plus"` in its `contest.json`.
