<!-- solve-problem:d6df3ff8-272b-4f1e-b8e2-b43f8e584227 -->
# Attainable hedging benchmarks for LP profitability

A [Tangle Solve problem](https://solve.tangle.tools/problems/d6df3ff8-272b-4f1e-b8e2-b43f8e584227).

## What is the problem?

In a constant-function automated market maker (CFMM) such as Uniswap, a liquidity provider (LP) deposits reserves of two assets (e.g., ETH and a stablecoin) and earns fees while allowing arbitrageurs to trade against the pool. The celebrated **loss-versus-rebalancing (LVR)** result of Milionis, Moallemi, Roughgarden, and Zhang (arXiv:2208.06046) shows that a passive CFMM LP replicates a *short* portfolio of convexity: relative to a hypothetical trader who continuously rebalances a portfolio matching the pool's holdings on a frictionless market, the LP loses value at rate (1/2)Γσ² (roughly, the pool's instantaneous "gamma" times the squared volatility of the external price), offset only by accumulated fees.

The open problem: **LVR is computed under a frictionless benchmark — the rebalancer trades continuously at zero cost with no slippage, latency, or settlement risk.** No real hedger can attain it. The task is to measure LP excess returns against *implementable* hedging strategies: discrete-time rebalancing subject to real trading costs (exchange fees, bid-ask spread, slippage, gas), funding/borrowing costs for the short-perpetual or futures leg (e.g., on Binance or Deribit), and settlement/latency constraints (hedge at block cadence, not in continuous time).

Formally: given a CFMM pool with reserves process (Xₜ, Yₜ), external price process Sₜ (e.g., geometric Brownian motion or empirical data), and a cost model c(·) for executing hedge trades, define the attainable benchmark set ℋ of admissible rebalancing/hedging strategies. Characterize and empirically estimate **sup over ℋ of the hedge benchmark return**, and compute LP return net of fees minus this attainable benchmark — i.e., true economic LP profitability. Ambiguities to fix: the cost model (linear costs? square-root impact? fixed gas per trade?), hedge instruments (spot rebalancing vs. perpetual futures), and hedge frequency (per block? fixed Δt? event-driven on arbitrage trades).

## Why does it matter?

LVR has become the standard metric for LP profitability in both academia and industry (market makers, LSE-listed market makers, Uniswap governance debates, DeFi protocol design); if the benchmark is unattainable, protocols may overestimate or underestimate true LP economics and misprice fee tiers and incentive emissions. A rigorous implementable benchmark would let LPs, DAOs, and researchers attribute returns precisely — separating fees, adverse-selection (LVR) losses, and hedging costs — and would inform whether LPing is viable without token subsidies. It also connects DeFi to classical transaction-cost hedging theory (Leland-style option hedging with costs), with a genuinely new dataset (on-chain pool reserves + derivatives funding rates).

## What's already known?

- Milionis, Moallemi, Roughgarden, Zhang (2022; revised 2026): define LVR, derive the (1/2)Γσ² loss rate for Brownian prices under *zero-transaction-cost continuous rebalancing*, and relate it to Black–Scholes convexity and to the pool's fee income. This is the frictionless baseline to be generalized.
- Follow-up work (2023–2025, in the LVR literature; verify exact citations) has produced: discrete-time LVR variants (loss rate scales with hedge interval), empirical LVR measurements on Uniswap v3 pools, and "counterpart" results on fee revenue exceeding LVR for high-fee/low-volatility pools.
- Classical literature: Leland (1985), Whalley–Wilmott, and Zakamouline on option replication under proportional costs — hedging under costs is solved asymptotically (wide no-trade bands of order (cost·Γ)^{1/3}), and these results are the natural machinery, but they have not been ported to the CFMM LP setting with block-time dynamics, gas, and funding-rate hedges.
- Gap: no published framework computes an LP benchmark under an explicitly implementable hedge (perpetual futures with funding rates, block-cadence rebalancing, gas), nor quantifies how far attainable benchmarks fall from frictionless LVR on real data.

## What would count as solving it?

An **experimental** solution: a paper/system that
1. Defines a concrete admissible hedge class ℋ (instruments, cost model, rebalancing frequency) — stated precisely enough to replicate;
2. Derives (analytically or numerically) the optimal or near-optimal hedge in ℋ (e.g., no-trade-band rebalancing around the frictionless LVR hedge), ideally with a bound on the gap to frictionless LVR as a function of costs;
3. Runs an empirical backtest on real pool data (e.g., Uniswap v3 ETH/USDC) and derivatives data (perp funding rates), reporting LP excess return vs. the attainable benchmark, with attribution to cost components;
4. Includes robustness (sensitivity to cost assumptions, volatility estimation) so the conclusion "LP profitability is/isn't overstated by frictionless LVR" holds across settings.
A referee would accept this even without a closed-form optimum, provided the benchmark is implementable, optimized or near-optimal, and empirically grounded.

## How to start

1. Read arXiv:2208.06046 (Sections on the LVR definition and the divergence-loss/LVR link; the discrete-time formulation) — this is the essential baseline. Then read Leland (1985) / Whalley–Wilmott for no-trade-band hedging under proportional costs.
2. Choose the simplest nontrivial setting first: GBM price, constant-product pool, proportional cost c per unit hedge notional, hedge at fixed interval Δt. Compute the discretized LVR benchmark and the optimal band; you should recover frictionless LVR as c→0, Δt→0 (sanity check).
3. Gather data: on-chain pool reserves/fees (Uniswap v3 via subgraph or Reorg/Flipside), Binance/Deribit perp funding rates, gas prices. Implement the hedged LP P&L at block cadence.
4. Main obstacle: joint modeling of funding rates and spot volatility, and avoiding look-ahead bias in the backtest; also, the "optimal" hedge under square-root impact has no closed form — use asymptotic or numerical (dynamic programming / convexity) arguments.

## Key references

- Milionis, J., Moallemi, C., Roughgarden, T., Zhang, A. (2022, rev. 2026). *Automated Market Making and Loss-Versus-Rebalancing*. arXiv:2208.06046. Defines LVR, proves the convexity-loss characterization of CFMM LP returns under frictionless continuous rebalancing. https://arxiv.org/abs/2208.06046
- Leland, H. (1985). *Option Pricing and Replication with Transaction Costs*. Journal of Finance 40(5). Classic result on option/convexity replication under proportional costs via adjusted volatility and no-trade bands.
- Whalley, A.E., Wilmott, P. (1997). *An asymptotic solution of the Hoggard–Whalley–Wilmott option pricing equation with transaction costs*. Asymptotic no-trade-band formulas for delta hedging with costs — the machinery to port to LVR hedging. (verify exact journal/venue)
- Empirical LVR measurement papers on Uniswap v3 pools (2023–2025, e.g., extensions by the same and other authors; verify) — provide data pipelines and the frictionless-benchmark quantifications to beat.

## Warnings

- Frictionless LVR is an *upper* benchmark for the hedger, but a real hedge with costs can also *under*-perform in ways that flip conclusions — direction of the bias is not obvious a priori; measure, don't assume.
- Watch for stale-price arbitrage nuances: LVR arises from adverse selection at block boundaries; a per-block hedge must use the price at which arbitrageurs actually traded, not a later close.
- Funding rates on perpetuals are lumpy, signed, and can be negative; a naive "borrow cost = funding" assumption is wrong in regimes of negative funding.
- Common mistake: comparing LP fees to LVR at daily volatility while hedging per block (mismatched timescales inflate or deflate the loss rate).
- The 2026 revision of arXiv:2208.06046 may itself discuss discretization/implementation caveats — check the latest version before claiming novelty.

---
*`BCGT-000353` · flagged for review*

## Acceptance guidance

An **experimental** solution: a paper/system that
1. Defines a concrete admissible hedge class ℋ (instruments, cost model, rebalancing frequency) — stated precisely enough to replicate;
2. Derives (analytically or numerically) the optimal or near-optimal hedge in ℋ (e.g., no-trade-band rebalancing around the frictionless LVR hedge), ideally with a bound on the gap to frictionless LVR as a function of costs;
3. Runs an empirical backtest on real pool data (e.g., Uniswap v3 ETH/USDC) and derivatives data (perp funding rates), reporting LP excess return vs. the attainable benchmark, with attribution to cost components;
4. Includes robustness (sensitivity to cost assumptions, volatility estimation) so the conclusion "LP profitability is/isn't overstated by frictionless LVR" holds across settings.
A referee would accept this even without a closed-form optimum, provided the benchmark is implementable, optimized or near-optimal, and empirically grounded.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/d6df3ff8-272b-4f1e-b8e2-b43f8e584227?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000353.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000353)
