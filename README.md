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

```
[PASS] testFuzz_activeRoundTrip(uint96,uint96,uint16)   (runs: 3000)
[PASS] testFuzz_sequenceSolvency(uint8[8],uint96[8])    (runs: 3001)
```

**Caveats.** No conclusion is drawn about the dependency surface. 100% coverage is not a proof of
absence. The "real funds" evidence was on-chain deployment activity rather than a DefiLlama TVL figure.

---

## 2. PairPad (`par`) — token launchpad (Robinhood Chain)

**Target.** `pardotfamily/par` — a self-authored Uni v4 no-hook launchpad (one launch = one token +
one pool, full supply into a permanently locked single-sided position) on Robinhood Chain (4663).

**Scope & coverage.** `contracts/src/` = 23 files, **3,882 / 3,882 SLOC (100%)**. `lib/` (71 `.sol`
files) is out of scope.

**Why this looked like a fair target.** No CI; the README states verbatim *"The contracts have not
been audited."*; created 2026-09-02, last push 2026-09-21; 7 stars; single-EOA governance; and a fee
escrow contract holding **0.4599 ETH** on-chain (real funds, sitting in its own contract).

**Honest counter-evidence.** Its tests are **not** thin — 16 test files / 3,005 test SLOC, including
a mainnet-fork full lifecycle. So this was "unaudited + no CI + young", *not* "barely tested".

**Why nothing held.** The numeric / auth / order families and both mandatory passes all reached the
code and returned "does not hold":

- the escrow's credit-from-caller flow can only ever move value *out*;
- the locker's `--0` burn relies on an `+=` underflow that reverts — the underflow *is* the guard;
- `_flush` repays bottom-up; `solidity ^0.8.26` plus `FullMath` throughout.

26 candidates, all falsified.

**Caveats.** Cantina was unreachable at the time, so "not on any of the four platforms" is verified
3/4. The escrowed funds are real but small. Several negatives are reasoning-level rather than
PoC-pinned. 100% coverage ≠ no defects.

---

## Method notes

- Candidates close by derivation, by test, or by an explicit "false" — never by running out of time.
- Nothing here was sent to any project, and no payment was involved: these are read-only reviews of
  public code.
- Un-audited ≠ broken, and audited ≠ safe. These reports say what I checked, not what exists.
