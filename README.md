<!-- solve-problem:421eb672-3230-4fa9-bd6d-53e18e2561b8 -->
# Compact relay under adversarial mempool divergence

A [Tangle Solve problem](https://solve.tangle.tools/problems/421eb672-3230-4fa9-bd6d-53e18e2561b8).

## What is the problem?

Bitcoin's BIP 152 compact block relay lets a node send a "compact block" that identifies transactions only by short (6-byte) hashes, assuming the receiving peer already has most of them in its mempool. The receiver then requests ("recovers") any missing transactions via getblocktxn/blocktxn. The problem: adversaries can deliberately make their mempool (or their view of what your mempool contains) diverge from yours — by withholding transactions, injecting spurious ones, or sending compact blocks that reference transactions you don't have — forcing worst-case recovery traffic and latency.

Formalize as follows. Fix a block B of n transactions. Node A sends a compact block to node B; each transaction is either known to B (recall succeeds) or must be requested. The adversary controls some fraction f of B's peers and can (i) withhold transactions from B so it must recover them after receiving the block header/short IDs, and (ii) serve recovery responses slowly or with garbage. Model: propagation delay D and bandwidth W as functions of (n, f, k) where k is the number of withheld transactions, under a per-block latency budget. Goals: (a) prove upper bounds on worst-case D and W achievable by any protocol; (b) construct a protocol (prefill policy, short-ID collision handling, peer-selection/redundancy strategy for recovery) attaining those bounds; (c) optimize the prefill parameter (which transactions to send in full) against an adversarial divergence distribution.

## Why does it matters?

Block propagation latency directly drives orphan/stale rates, which in turn affect consensus security (selfish mining profitability, consensus incentives) and decentralization, since slow validators are disadvantaged. If compact relay degrades to full-block transmission under attack, an adversary can cheaply amplify bandwidth costs network-wide during congestion — a censorship and partitioning vector. A principled robust design would also transfer to other short-hash/reconciliation protocols (Erlay deployments, other chains).

## What's already known?

- BIP 152 (Corallo, 2016) specifies compact relay with 6-byte truncated SipHash-2-4 short IDs and a fixed recovery round (getblocktxn/blocktxn), including a one-extra-round fallback for collisions. It has no adversarial analysis; its recovery assumes the responding peer is honest and responsive.
- Erlay (Naumenko et al., 2019) gives efficient transaction reconciliation via PinSketch (set reconciliation on libminisketch-style BCH sketches) across the mempool gossip layer, reducing bandwidth ~40%; it also assumes semi-honest peers for the reconciliation protocol's efficiency guarantees (verify the exact adversarial model in §threat model).
- Known primitives exist for the "coded communication" angle: erasure-coded block distribution and set reconciliation with known communication complexity bounds (e.g., it takes O(k log(n/k)) bits to reconcile k differences information-theoretically).
- Gap: no published work characterizes worst-case recovery latency/bandwidth for compact relay when peers strategically diverge mempools, nor an optimized prefill policy against that adversary.

## What would count as solving it?

1. **Model + lower bound:** A formal adversary/game model (divergence strategy, budget per epoch) and a lower bound on expected recovery bandwidth/latency any compact-relay-style protocol must incur.
2. **Construction + upper bound:** A protocol (prefill rule, sketch-based or coded recovery, redundant peer requests) with a proven upper bound matching or near-matching the lower bound, and a proof of fallback correctness (a bound on short-ID collision probability remains negligible).
3. **Empirical validation:** Simulation (e.g., on Bitcoin mainnet mempool/block traces) showing bounded latency/bandwidth under emulated adversarial withholding, vs. BIP 152 baseline.
Any one of the three, done rigorously, is a publishable partial result; all three solve it.

## How to start

1. Read BIP 152 in full (it's short); note the recovery state machine, the extra-round collision handling, and that nothing constrains a malicious responder's timing/content.
2. Read Naumenko et al. (2019), especially the threat-model and reconciliation sections; consider replacing getblocktxn recovery with a PinSketch-based sketch of the missing set so one round reconciles even with collisions/ambiguity.
3. Formalize the adversary: it chooses k withheld transactions (bounded by private mining / withheld relay), and controls f of the peers queried for recovery. First analysis target: expected latency as function of k and f with and without 2-peer redundancy.
4. Main obstacle: real prefill decisions are made online under uncertainty about the receiver's mempool; the analysis must couple peer-selection incentives (game theory) with the communication-complexity lower bound.

## Key references

- M. Corallo, 2016, "BIP 152: Compact Block Relay," https://github.com/bitcoin/bips/blob/master/bip-0152.mediawiki — specifies the deployed compact-block protocol: short IDs, header-first announcement, and the getblocktxn/blocktxn recovery procedure.
- G. Naumenko, G. Andresen, A. Shtoff, S. Tikhomirov, E. McCorry, P. Gazi, 2019, "Erlay: Efficient Transaction Relay for Bitcoin," ACM CCS, DOI:10.1145/3319535.3354237 — reduces transaction-relay bandwidth via PinSketch set reconciliation and derives new mempool-synchronization parameters for Bitcoin.
- (verify) Related background on propagation latency and stale rates: Decker & Wattenhofer, "Information Propagation in the Bitcoin Network," IEEE P2P 2013.

## Warnings

- BIP 152 short IDs are truncated hashes; any redesign that changes ID length or adds sketches must re-analyze collision probability under an adversary who can grind transactions to force collisions.
- Don't assume the adversary must equivocate; withholding from just one victim is cheaper and enough to cause the studied delay.
- The problem statement mixes "bound" and "optimize": decide up front whether you're proving worst-case bounds or optimizing expected cost under a distribution — referees will ask which.
- Simulation results alone won't settle this; the census record frames it as an open analysis/design target, not a benchmark gap.

---
*`BCGT-000264` · flagged for review*

## Acceptance guidance

1. **Model + lower bound:** A formal adversary/game model (divergence strategy, budget per epoch) and a lower bound on expected recovery bandwidth/latency any compact-relay-style protocol must incur.
2. **Construction + upper bound:** A protocol (prefill rule, sketch-based or coded recovery, redundant peer requests) with a proven upper bound matching or near-matching the lower bound, and a proof of fallback correctness (a bound on short-ID collision probability remains negligible).
3. **Empirical validation:** Simulation (e.g., on Bitcoin mainnet mempool/block traces) showing bounded latency/bandwidth under emulated adversarial withholding, vs. BIP 152 baseline.
Any one of the three, done rigorously, is a publishable partial result; all three solve it.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/421eb672-3230-4fa9-bd6d-53e18e2561b8?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000264.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000264)
