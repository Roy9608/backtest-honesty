# Backtest Honesty Check

English | [简体中文](README.zh-CN.md)

**Before you say "this strategy makes money," run these 14 checks first.**

The most dangerous thing about a backtest report is not a calculation error — it's **mistaking the market's drift for your own skill**.
This repository provides a check-list workflow for catching exactly that, before you ever report a "positive return" conclusion.

---

## What problem does this repository solve?

A positive backtest has five common causes. Only one of them is real:

| What you see | What you conclude | What it actually is |
|---|---|---|
| Positive backtest | The strategy works | **You ate β (market drift)** |
| Train and test both positive | It's robust | **Both halves ate β** |
| Every parameter makes money | Not parameter-sensitive | **β dominates; parameters don't matter** |
| Longer holding = higher return | The signal needs time | **Longer horizons accumulate more β** |
| One segment is very profitable | The strategy adapted | **That segment's market was simply good** |

**The common thread: mistaking β for α.**

The 14 iron laws in this repository turn each of these situations into an executable check.

---

## Quick start

### If you have 5 minutes

These three rules catch the most problems:

1. **[Rule 1: Compute the β baseline before looking at α](docs/01-beta-first.md)**
2. **[Rule 7: Explicitly model "one position at a time"](docs/07-single-position.md)**
3. **[Rule 11: Fill-price assumptions can invent returns out of thin air](docs/11-fill-price.md)**

### If you're producing a full backtest report

Work through the [checklist](docs/checklist.md) item by item, with the table templates in the [output format](docs/output-format.md).

### If you're reviewing someone else's backtest

Start from [common misjudgment patterns](docs/misjudgment-patterns.md) and check one by one.

---

## The 14 iron laws

### Group 1: Return attribution (the most fundamental)

| # | Rule | In one sentence |
|---|---|---|
| 1 | [Compute the β baseline before α](docs/01-beta-first.md) | Only what remains after removing market drift is your skill |
| 2 | [Segment by market regime, don't just take the total](docs/02-segment.md) | "Up 30% in 90 days" doesn't mean it rose every day |
| 3 | [Train/test consistency ≠ a working strategy](docs/03-train-test.md) | When both halves eat β, consistency means nothing |

### Group 2: Costs & execution

| # | Rule | In one sentence |
|---|---|---|
| 4 | [Measure costs; distinguish entry/exit order types](docs/04-cost.md) | Don't ask for one number; ask what order type each leg used |
| 9 | [Execution style is part of the strategy](docs/09-execution.md) | Queue depth is not a free cost reduction |
| 14 | [Four-way return decomposition](docs/14-four-way-decomposition.md) | Mandatory for passive-fill strategies, or you'll book execution gains as signal skill |

### Group 3: Criteria & robustness

| # | Rule | In one sentence |
|---|---|---|
| 5 | [Criteria must fit the strategy type](docs/05-criteria-fit.md) | The wrong ruler rejects a usable strategy |
| 8 | [Report baselines and robustness as intervals](docs/08-interval.md) | β itself has sampling noise; don't report a single number |
| 10 | [Don't drop subgroups; answer sample size with a t-value](docs/10-grouping-t.md) | Dropping losing subgroups = filtering samples by outcome |

### Group 4: Modeling correctness

| # | Rule | In one sentence |
|---|---|---|
| 7 | [Explicitly model "one position at a time"](docs/07-single-position.md) | The most severe bug — and invisible to routine checks |
| 11 | [Fill-price assumptions can invent returns](docs/11-fill-price.md) | Always use the conservative assumption; "too good to be true" means bug |
| 12 | [Verify semantic premises before porting rules](docs/12-semantic-premise.md) | The same rule name can mean different things in two strategies |
| 13 | [Every risk-control overlay must be tested](docs/13-risk-control.md) | Never rely on "safer / more is better" intuition |

### Group 5: Delivery

| # | Rule | In one sentence |
|---|---|---|
| 6 | [Run a semantic-consistency check before shipping documents](docs/06-semantic-consistency.md) | Old conclusions read punchier; readers will trust them first |

---

## Where this comes from

Not copied from a textbook.

It comes from a real strategy research project: the author spent three years on tick-level data and ran
hundreds of controlled experiments, **committing all 14 classes of errors along the way** —
each error was distilled into an iron law the moment it was discovered.

The final conclusion: the signal had **zero information content and must not go live**.

> **The value of this repository is not "which strategy makes money." It's "how to tell whether a strategy is lying to you."**

An honestly documented failed experiment is more useful than a successful one.

---

## The core method: four-way return decomposition

The single most valuable item in this repository.

The net return of a passive-fill (limit-order) strategy can — and must — be decomposed and measured in four parts:

```
Net return = ① raw signal quality + ② selection effect + ③ price advantage − ④ cost
```

| Part | How to measure | What it answers |
|---|---|---|
| ① Raw signal quality | All signals, market orders (100% fill) | Does the signal itself have any α |
| ② Selection effect | Limit-order *selection*, market-order *pricing* | Are the fills a biased subsample of all signals |
| ③ Price advantage | Limit price vs market price | The mechanical spread (usually equals queue depth) |
| ④ Cost | Actual fee schedule | — |

**Reconciliation**: ① + ② + ③ − ④ must equal the measured per-trade net.
If it doesn't, the decomposition or the code is wrong.

### Why this is mandatory

Skip it, and you'll book mechanical execution gains as signal skill.

A real measurement makes the point: the same strategy in two different market windows had
**nearly identical raw signal quality** (difference < 0.002pp) yet net returns differing by more than 5× —
**100% of the gap came from the selection effect**.

> **Mechanism**: to get filled X bp below mid, price must dip X bp.
> In a falling market, a dip commonly mean-reverts (oversold bounce); in a rising market, a dip means the trend broke (regime change).
> **The same action means opposite things in the two regimes.**

---

## How to use this

1. **Don't expect it to raise your returns.** Its job is to make you stop early when you're heading the wrong way.
2. **Read the rules in order; don't skip.** The first few are prerequisites for the later ones.
3. **Actually tick the checklist.** "This should be fine" is the most dangerous sentence in backtesting.

---

## Language note

The detailed rule-by-rule write-ups under `docs/` are currently **in Chinese only**.
This README is self-contained for the core method; if you machine-translate the docs, watch for terminology drift.

---

## License

MIT — use freely, no attribution required. If it saves you from one expensive mistake, that's enough.
