# AGENTS.md — how an AI agent should work in this repository

If you are an AI agent asked to "look at this repo", "check my backtest with it", or "add a rule to it", this file is the contract.
Everything stated here is drawn from the repository's own README and `docs/`; where the repository does not state something, this file says so instead of guessing.

## 0. What this repository is

A check-list workflow for backtest honesty: **before you report a "positive return" conclusion, prove it is not the market's drift (β) rather than your own skill (α).** It contains **14 iron laws in 5 groups, plus a pre-conclusion checklist** — documentation only, no code. Every rule comes from a real strategy research project (three years of tick-level data, hundreds of controlled experiments, desensitised) whose final conclusion was that the signal had **zero information content and must not go live**. Its audience is whoever is about to put a number in a report — the person producing a backtest report, or the person reviewing someone else's. There is **no runnable artefact of any kind**.

```
README.md                          entry point (English; README.zh-CN.md is the Chinese version)
README.zh-CN.md                    Chinese entry point
docs/01-beta-first.md              iron law 1  — compute the β baseline before looking at α
docs/02-segment.md                 iron law 2  — segment by market regime
docs/03-train-test.md              iron law 3  — train/test consistency ≠ a working strategy
docs/04-cost.md                    iron law 4  — measure costs; distinguish order types
docs/05-criteria-fit.md            iron law 5  — criteria must fit the strategy type
docs/06-semantic-consistency.md    iron law 6  — semantic-consistency check before delivery
docs/07-single-position.md         iron law 7  — model "one position at a time" explicitly
docs/08-interval.md                iron law 8  — report baselines and robustness as intervals
docs/09-execution.md               iron law 9  — execution style is part of the strategy
docs/10-grouping-t.md              iron law 10 — don't drop subgroups; answer sample size with a t-value
docs/11-fill-price.md              iron law 11 — fill-price assumptions can invent returns
docs/12-semantic-premise.md        iron law 12 — verify semantic premises before porting rules
docs/13-risk-control.md            iron law 13 — every risk-control overlay must be tested
docs/14-four-way-decomposition.md  iron law 14 — the four-way return decomposition
docs/checklist.md                  the pre-conclusion checklist (sections A–F + last three questions)
docs/misjudgment-patterns.md       common misjudgment patterns, each one a real mistake
docs/output-format.md              the report output specification and its mandatory table template
docs/appendix-three-review.md      the three-review process the source project used before changing code
LICENSE                            MIT, Copyright (c) 2026 Roy9608
```

**Every document under `docs/` is in Chinese only.** The English `README.md` is self-contained for the core method (the four-way decomposition) but is not a translation of `docs/`.

## 1. What "prove it runs" means here — and what it does not

**There is no executable artefact in this repository: nothing to install, no network, no keys — and nothing to run.** Do not search for a script, do not invent a command, and do not claim you executed anything. If a user asks you to "run it", the honest answer is that this repository is documentation and there is no run for you to perform.

The only verification that exists here is a **read-only check that the checkout is complete**. No network, no keys, no install:

```bash
ls docs/ | wc -l          # a complete copy has 18 documents
ls LICENSE README.md README.zh-CN.md
```

A complete copy matches the tree in section 0: `LICENSE`, `README.md`, `README.zh-CN.md`, and the 18 documents in `docs/`. If a file above is missing, say so rather than filling the gap from memory.

Confirming that **the files are present** is all you can confirm. You cannot confirm that the method "works" by running it, and you must not describe the repository as tested.

If the user only asked to "look at the repo", stop after reading `README.md` and this file.

Not documented in the source repository (do not assert these): any byte size or file count beyond the tree above, the number of experiments the source project ran beyond "hundreds", any figure not printed in `docs/`, and any claim about how the method performs on the user's own data.

## 2. Ask the user before going further

Do not guess any of these. Each one changes which rule you point to, or makes a rule inapplicable.

| # | Question | Why it matters |
|---|---|---|
| 1 | Do you already have a backtest result claiming a **positive return**, or are you still designing the strategy? | If a result exists, go straight to the checklist. If not, walk the rules in order — the early ones are prerequisites for the later ones. |
| 2 | What **strategy type** is it — trend-following, mean-reversion, or high-frequency / arbitrage? | Iron law 5: each type is judged by different metrics, and the wrong ruler rejects a usable strategy. You cannot say whether a result is "healthy" without this. |
| 3 | Does the strategy enter with **limit (passive) fills or market orders**? | Iron law 14 is mandatory only for passive-fill strategies; laws 9 and 11 apply differently. You cannot pick the right checks without knowing this. |
| 4 | What **per-trade cost** are you using, is it measured or assumed, and what order type does each leg use? | Iron law 4: an assumed cost and a measured cost differ, and entry/exit order types change the number. Without it you cannot compute α vs cost — the final gate. |
| 5 | What is the strategy's **average holding period**, and what **bar interval** is the data? | Iron law 1: the β baseline must be simulated with the *same* holding duration as the strategy. Comparing different durations is meaningless. |
| 6 | Can positions overlap, and does the code model **"one position at a time"** explicitly? | Iron law 7: the single-position invariant is the most severe bug, and routine checks (arithmetic self-consistency, position occupancy, per-trade metrics) cannot see it. Do not assume it is implemented. |
| 7 | Do you want to **use** this repository, or **change** it? | If "use it", do not edit anything — walk them through `docs/`. If "change it", see section 4. |

Never invent the user's strategy type, cost, holding period, or fill assumption. Ask.

## 3. Failure modes → meaning

When the user describes a symptom of a backtest, this is what it most likely means. (These pairings come from the repository's own laws and pitfalls; each row points to the document that carries the criterion.)

| Symptom | What it most likely means | What to do |
|---|---|---|
| "The backtest is positive" / "train and test are both positive" / "every parameter makes money" / "longer holding is better" / "one segment is very profitable" | **β mistaken for α.** The repository lists these five forms together in `README.md` §2 — they are one cause, not five discoveries | Compute the β baseline for the same holding period, subtract it, and compare the remainder against cost. `docs/01`, then `docs/02`, `docs/03` |
| "α came out hundreds of times the cost" | **β was summed per-trade** while positions overlapped, so β was counted many times over. (A real case reached α = 517× cost while the net result was −53%.) | Use a per-trade average, or deduct β once per non-overlapping period; verify that per-trade net × count reconciles with the total. `docs/01` |
| "A stop-loss rule gives t = +5.01 and net α +18.4% — the best result ever" | **Fill-price accounting.** The line's own price was used as the fill price, and the line sat *above* the entry price. The tell is **t extremely high *and* holding period extremely short** | Re-run with the conservative fill (trigger-minute close) and report the conservative version; the optimistic one appears only as a deviation check. `docs/11` |
| "It's a bear-market strategy; it fails in bull markets" | The root cause is the **selection effect** — the execution layer was never decomposed, so a difference in net returns was read as a difference in signal quality | Run the four-way decomposition and reconcile it. `docs/14` |
| "Dropping the best segment turns it negative, so it's fatal" | **Wrong criterion.** That test is stricter than the correct one and is equivalent to demanding every segment be positive | Use loss-sum ÷ best-segment < 30%; treat "negative without the best segment" as a **sample-size hint**, not a verdict. `docs/08`, `docs/05` |
| "Only 3 of 6 segments were positive and the top 5 days are 143.9% of net α → not tradeable" | **Wrong criterion for a trend strategy.** "Cut losses, let profits run" *is* its normal shape; demanding even returns is the overfit | Judge by per-trade expectancy, α/cost, and the loss-sum ratio — not by evenness of returns. `docs/05` |
| "I added a hard stop-loss — safer is better" | **Measured, it was not.** A 1.2% stop took a positive-expectancy strategy to zero expectancy; the stop cut exactly the dip-then-bounce trades the strategy feeds on | Sweep at least 6 levels before adding any risk control, and report tail improvement and expectancy loss as two separate numbers. `docs/13` |
| "Two mechanisms each work; combining them should be better" | **They offset.** "Give room" and "tighten after profit" pull in opposite directions, so the combination pays both costs and keeps neither benefit | Ask whether the two mechanisms act on the same failure mode; re-run the parameter neighbourhood after stacking. `docs/13` |
| "n is small but t is huge" | A few samples are driving the result | Ask for n and the t-value; the criterion is **t < 2 means "direction", not "effective"**, and you must state how many more trades are needed for t = 2. `docs/10` |
| "A group had fewer than 3 trades so it was skipped" | **Filtering samples by outcome** — small groups correlate with short, bad market windows; the dropped group was the losing one | Run one window and group *after*; the grouping sum must equal the total. `docs/10` |
| "The two documents contradict each other" | A **stale conclusion** was never cleaned up after a revision; the older text usually looks more concrete, so readers believe it first | Search every overturned number and conclusion; keep exactly one conclusion section. `docs/06` |
| "The same parameters gave different results on two runs" | A **silent default** — e.g. a default parameter quietly replacing the value the caller meant to pass | Make the parameter explicit and fail loudly when it is missing; verify with a run-to-run comparison. `docs/appendix-three-review.md` |

## 4. Iron rules for an agent working in this repository

1. **Never remove or soften the honesty statements.** The line "the value of this repository is not which strategy makes money", the disclosure that the source project **ended in a negative result** (zero information content, must not go live), and the statement that this is a field report rather than an industry standard are the product, not boilerplate. Do not tidy them away.
2. **Never invent facts, numbers, commands, metrics, or file paths.** If a figure is needed and the repository does not provide it, write "not provided in the source" rather than producing a plausible number. The repository's own rule is stricter than most: **no "≈", no vague wording, and a value you cannot obtain is marked "未获取" (not retrieved)** — see `docs/output-format.md`.
3. **Do not add a section teaching the reader how to verify something.** The verification *is* `docs/checklist.md` plus the table template in `docs/output-format.md`. The prose keeps only the entry point; a capable agent re-derives method on its own, whereas it cannot invent your data.
4. **Never introduce a strategy, a parameter, an instrument, or a performance number.** That is the one thing this repository explicitly does not contain. If the user pastes their own figures, treat them as confidential to that user's session — do not write them into any file here.
5. **Never introduce secrets or identifying data into any file here.** No tokens, API keys, email addresses, server IPs or host paths, exchange names, account information, client names, or company names. This repository's desensitisation is a hard requirement; replace real values with placeholders before writing anything.
6. **Do not renumber or rename the 14 iron laws, the 5 groups, or the 4 parts of the decomposition.** The numbering is not cosmetic: each law maps to one of the 14 classes of error the source project actually committed, and the decomposition's four parts must reconcile. Adding a new law at the end is acceptable if it comes from a real mistake; reordering is not.
7. **Every new rule or pitfall must carry the same parts as the existing ones:** what happened, the criterion that tells you which state you are in, and what to do. A rule without its criterion is an anecdote, not a law.
8. **Never present a positive backtest number as a conclusion.** A backtest return is not evidence until β has been subtracted and the remainder compared against cost. Presenting the raw number as a result is the exact illusion this repository exists to expose.
9. **Keep both language versions complete.** `README.md` (English) and `README.zh-CN.md` (Chinese) must each be readable on their own; an English stub with the detail only in Chinese, or the reverse, is a defect. `docs/` is Chinese-only in the source repository — say so rather than writing a half-translation.
10. **Do not silently change tone or scope.** The register is deliberately blunt and honest ("An honestly documented failed experiment is more useful than a successful one"). Do not inflate it into marketing copy, and do not make promises about returns this repository cannot back.
11. **State what you are unsure of.** The source project's own process makes this a hard requirement, not a courtesy: an agent that reports "no uncertainties" is itself the suspicious signal (see `docs/appendix-three-review.md`).
