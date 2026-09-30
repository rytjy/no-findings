# no-findings

Two reviews where I read **all** of the in-scope code and nothing was reportable.

Most reviewers can show a finding. Far fewer can show a review that ends in *"I read it all and the
candidates didn't hold"* — because it doesn't look good. I publish these because the honest output of
a review is not always a bug list.

| Target | Chain | Scope | Coverage | Candidates | Result |
|---|---|---|---|---|---|
| **Fathom** (DLMM) | Robinhood Chain (4663) | `src/` · 19 files | **2,018 / 2,018 SLOC (100%)** | 13 | none committable |
| **PairPad** (`par`) | Robinhood Chain (4663) | `contracts/src/` · 23 files | **3,882 / 3,882 SLOC (100%)** | 26 | none committable |

## What "coverage: X/Y SLOC (100%)" means — and what it does not

Every line of the in-scope source was read, and every `file:line` in the ledger was re-read from that
output. It does **not** mean the code is defect-free: it means **no candidate survived my own
filters**. Dependency code is out of scope and carries no security conclusion.

---

## 1. Fathom — DLMM (Robinhood Chain)

**Target.** `FathomPools/fathom-contracts` — a discrete-bin AMM (Liquidity Book design, not a
line-by-line fork) on Robinhood Chain (chainId 4663). Single commit `8bbf48b` "initial public
release"; no TODO/FIXME in the tree.

**Scope & coverage.** `src/` = 19 files, **2,018 / 2,018 SLOC (100%)**. Libraries
(OpenZeppelin `uniswap-hooks`, `v4-core`, `v4-periphery`, `forge-std`) are out of scope.

**What was checked.** File-level sweep (19 items), numeric/accounting (5), order/context (5),
method-level paths (4); 13 candidates (C-01…C-13), each closed by a derivation or an explicit
falsification.

**Why nothing held**

1. The code is self-authored **and already tests the three properties I would attack**: fee
   round-trip through the active bin (unit test + 256-run fuzz), the oracle guard band only ever
   moving *toward* the oracle, and the buyback reference price being unmanipulable without elapsed
   time.
2. The candidate families had real, non-privileged entry points — the result is "did not hold",
   not "not examined".
3. The accounting is **locally decidable**, so instead of asserting, I ran an independent fuzz probe:
   8-step randomized mint/swap/burn sequences, asserting `balanceOf(pair) == getReserves()` after
   every step — 3,001 runs, all passed.
