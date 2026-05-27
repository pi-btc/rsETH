# [Research] Correlated Panic: A Random Matrix Theory Analysis of Cross-Pool Withdrawal Dynamics and Safety Module Behavior in the Kelp DAO Bridge Exploit

*Yuruzuc's Working Paper for Aave Community · May 26, 2026 · [Full paper available on (https://github.com/pi-btc/rsETH/blob/main/Kelp_DAO_2026.pdf)]*

---

We apply Random Matrix Theory to 24,050 withdrawal transactions across 29 Aave V3 Ethereum assets (April 16–19, 2026) to quantify what LlamaRisk's post-mortem described qualitatively: the moment depositors stopped making pool-specific decisions and fled the protocol as a whole.

**The headline finding:** the cross-pool withdrawal correlation matrix produced a statistically significant signal at 18:00 UTC on April 18 — sixty minutes before public news broke at 19:00 UTC.

---

## Method in one paragraph

For each hour, we compute the dominant eigenvalue λ₁ of the 29×29 cross-pool withdrawal correlation matrix using a 48-hour rolling window. The Marchenko-Pastur distribution gives us a theoretical upper bound λ+ = 3.16: any λ₁ above this threshold means withdrawal co-movement is too structured to be random noise. Effective rank measures how many genuinely independent behavioral dimensions remain across the 29 pools (maximum = 29, full panic = 1).

---

## The numbers

| Period | λ₁ | Eff. rank | Mean cross-corr |
|---|---|---|---|
| Pre-attack (48H ending 17:00 Apr 18) | 4.52 | 18.8 | 0.022 |
| **Acute phase (48H ending 20:00 Apr 18)** | **15.80** | **5.5** | **0.349** |
| Post-panic (48H ending 12:00 Apr 19) | 8.19 | 11.3 | 0.178 |
| M-P noise ceiling (λ+) | **3.16** | — | — |

Three things stand out.

**60-minute lead.** λ₁ first crossed λ+ at 18:00 UTC (λ₁ = 4.49), 25 minutes after the exploit and 60 minutes before public news. Withdrawals that hour: $132M. The following hour: $2.08B.

**Effective rank collapse.** 29 pools compressed into the behavioral equivalent of 5.5 correlated actors at peak. This is the quantitative expression of what LlamaRisk described as depositors being unable to distinguish rsETH-specific risk from protocol-wide risk. When effective rank is 5.5, that inability is not a behavioral bias — it is an accurate reflection of the withdrawal correlation structure.

**Mean correlation ×16.** Cross-pool mean correlation went from 0.022 at baseline to 0.349 at peak. Knowing one pool was being drained told you a great deal about every other pool, regardless of asset type.

---
## Rolling Analysis

<img width="1961" height="1980" alt="rmt_rolling_timeseries_e" src="https://github.com/user-attachments/assets/5ef82904-81d9-4d57-88b7-49a4e748a9fd" />



*Four time series from top to bottom: λ₁ max eigenvalue (blue dashed line: M-P upper bound λ+ = 3.16; pink shaded region: hours exceeding the bound), effective rank, mean cross-pool correlation, and number of eigenvalues exceeding λ+. Orange dashed vertical line: exploit onset at 17:35 UTC. Yellow dotted vertical line: public news release at 19:00 UTC. Data: Aave V3 Ethereum Subgraph · 24,050 transactions · 29 assets.*

---

## Umbrella

LlamaRisk reports 18,922 of 23,507 staked aWETH holders (80.5%) entered cooldown before slashing triggered. We tried to reconstruct this from on-chain data and could not — the Umbrella stkAaWETH contract has no verified source code on Etherscan and cooldown events are not indexed by the standard Aave subgraph. The 80.5% figure is only available because LlamaRisk ran forensic analysis after the fact.

That opacity is itself a finding. A safety module whose activation dynamics cannot be monitored in real time provides weaker protection than its nominal parameters suggest.

On the game theory: the concentrated exit pattern is consistent with a coordination game (Diamond & Dybvig, 1983), not independent decision-making. The 20-day cooldown begins at initiation, not completion. Initiating cooldown under stress preserves optionality — complete exit if slashing triggers, cancel if conditions normalize. That asymmetry makes cooldown initiation weakly dominant for any staker assigning positive probability to slashing. This is a mechanism design problem, not a parameter calibration problem.

---

## What we'd suggest

**A λ₁/λ+ monitor.** A continuous ratio computed from rolling cross-pool withdrawal flows is a low-cost addition to risk surveillance. The data is on-chain. The computation is cheap. It is sensitive to behavioral contagion specifically — something oracle-based and utilization-based monitors are not designed to catch.

**Index Umbrella cooldown events.** First-class subgraph indexing of cooldown events, with a public real-time ratio of staked to cooldown-initiated positions. A safety module you can only analyze post-hoc provides much weaker guarantees than one you can watch in real time.

---

## Honest caveats

The 1-hour aggregation window means our earliest detectable signal is inherently lagged. λ₁ was already modestly above λ+ before the attack (4.52 vs. 3.16), reflecting normal structural correlations in any lending protocol — a production monitor would need to distinguish "elevated but stable" from "elevated and rising," which requires baseline calibration we haven't done. The Umbrella game-theoretic model is calibrated to a single aggregate data point.

We are not claiming an RMT monitor would have prevented anything. We are claiming it would have produced an alert 60 minutes earlier than public information, and that the methodology for measuring behavioral contagion quantitatively seems worth developing further.

Full methodology, data, and code available on request. Pushback welcome.

---

*Data: Aave V3 Ethereum Subgraph · 24,050 txns · 29 assets · Apr 16–19 2026*

*References: Marchenko & Pastur (1967) · Laloux et al. (1999) · Plerou et al. (1999) · LlamaRisk (April 20 2026) · Diamond & Dybvig (1983)*
