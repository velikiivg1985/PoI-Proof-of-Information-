

---

# PoI
## Proof‑of‑Information Consensus

**Version:** 1.1  

---

## Abstract

Entropy Protocol defines a consensus mechanism in which network security is derived from the *information value of predictions* rather than from computational expenditure or capital concentration. The right to produce a block and receive emission is earned by the quality of a verifiable forecast about the external world.

The specification introduces the following mechanisms:

- **Proof‑of‑Information (PoI)** as the core consensus layer.
- **Resolution Uncertainty Layer** for representing outcomes as probability distributions.
- **Strictly proper scoring rules** for calibrated forecast evaluation.
- **Multi‑Level Reputation** that caps the influence of any single participant.
- **Cryptographically‑Attested Proving Registry** to prevent centralisation of ZK infrastructure.
- **Reputation‑Weighted Adaptive Clustering** resilient to uncertainty manipulation.
- **Cassandra Mechanism** that rewards accurate counter‑consensus predictions without penalising them during consensus formation.
- **Post‑Resolution Dispute Window** with atomic re‑calculation and anti‑spam economics.
- **Energy‑Arbitration** that rewards information throughput per watt rather than absolute power consumption.

As a by‑product the network produces a continuously updated global forecast set with quantified uncertainty, available as a public good.

---

## 1. Scope

The protocol is intended for:
- decentralised networks with block finalisation,
- prediction markets,
- collective data‑verification systems,
- applications that require a combination of consensus, forecasting, and reputation scoring.

The protocol is **not** intended for arbitrary computation outside the context of forecasting and verification.

---

## 2. Terms and Definitions

### 2.1. Predictor Node
A network participant that submits:
- a forecast $\mathcal{P}_i$,
- a zkPoI proof $\pi_i^{inf}$,
- a zkPoDP proof $\pi_i^{prov}$.

### 2.2. Oracle Query
A formalised request $q$ that specifies:
- an identifier,
- the question text,
- resolution criteria,
- a time horizon,
- the resolution format,
- metadata about difficulty and category.

### 2.3. Resolution
The true outcome $\mathcal{D}_{true}$, aggregated from signed data sources and represented as:
- a point value,
- a parametrised distribution,
- or an empirical distribution.

Each resolution is accompanied by:
- *confidence* – the oracle’s self‑assessed certainty,
- *source_divergence* – a measure of disagreement among sources,
- $T_{final}$ – the moment of finalisation,
- $T_{dispute\_end}$ – the moment the dispute window closes.

### 2.4. Calibration Score
A quality metric for forecasts computed with strictly proper scoring rules. For continuous distributions the primary rule is the Continuous Ranked Probability Score (CRPS). The final score is bounded $[0,\, 1.5]$.

### 2.5. Multi‑Level Reputation
A reputation vector  
$$
\vec{R}_i = (R_i^{short},\; R_i^{medium},\; R_i^{long})
$$  
where the components correspond to windows of 100, 500, and 2000 events, respectively.

### 2.6. Effective Reputation
$$
R_i^{eff} = \min\!\Big(R_i^{weighted} \cdot \kappa \cdot (1 + 0.5 \cdot \mathbb{I}_{booster}),\; R^{max}\Big)
$$
where $R_i^{weighted}$ is a weighted average of the multi‑level vector, $\kappa$ is a dynamic cap linked to the network median, and $\mathbb{I}_{booster}$ marks a temporary Breakthrough Booster. $R^{max}$ is an absolute hard cap.

### 2.7. Proving Attestation
A cryptographic proof that a zk‑proof was generated on the claimed hardware class and in the claimed geographic region.

### 2.8. Reputation‑Weighted Entropy
$$
H_w(\mathcal{P}) = -\sum_i w_i\, p_i \log p_i
$$
where the weights $w_i$ are proportional to $R_i^{eff}$.

### 2.9. Cassandra Eligibility
A predicate fixed *before* outcome revelation. A forecast is eligible if it is a statistical outlier, exhibits high complexity, comes from a node with sufficient reputation, shows historical consistency, and concerns a “hard” question.

### 2.10. Post‑Resolution Dispute
A protocol for challenging a resolution after it has been finalised.

### 2.11. Entropy Circuit Breaker
An emergency regime that freezes the clustering parameter $\lambda$ to its conservative minimum if the reputation‑weighted entropy increases for a consecutive number of epochs.

---

## 3. Epoch Model

Each epoch lasts $\Delta t = 1$ hour and consists of the following phases:

1. **Inception** – Oracle queries are generated and question parameters are fixed.
2. **Commit** – Predictors submit forecasts, zkPoI proofs, zkPoDP proofs, and a declaration of energy consumption.
3. **Blind Evaluation & Clustering** – Forecasts are clustered, a judge committee is formed, and a preliminary evaluation is performed.
4. **Consensus** – The top‑$K$ nodes by composite weight finalise a block through a BFT‑style protocol.
5. **Resolution** – Oracles asynchronously supply the ground truth; phantom shares are converted into real tokens; reputations are updated.
5.5. **Post‑Resolution Dispute Window** – A challenge period with arbitration and atomic re‑calculation.
6. **Emergency Mode** – The Veto Council may temporarily halt emission in case of a critical vulnerability.

---

## 4. Formal Dependencies

### 4.1. Entropic Weight
The weight of node $i$ in epoch $t$ is:

$$
W_i(t) = \alpha\, N_i(t) + \beta\, C_i(t) + \gamma\, R_i^{eff}(t) - \delta\, P_i(t)
$$

Default values (governable):
- $\alpha = 0.35$
- $\beta = 0.25$
- $\gamma = 0.30$
- $\delta = 0.10$

### 4.2. Novelty
$$
N_i(t) = \Big[ D_{KL}(\mathcal{P}_i \,\|\, \mathcal{P}_{center(c_i)}) + \Delta_{inter}(c_i) \Big] \cdot \big(0.7 + 0.3 \cdot Consistency_i\big)
$$
where $\Delta_{inter}(c_i)$ is the minimum inter‑cluster KL divergence to any other cluster, and $Consistency_i$ is the historical correlation between the node’s forecasts and the true outcomes (measured over a trailing window).

### 4.3. Complexity
$$
C_i(t) = \log\!\left(\frac{\text{FLOPs}_i}{\text{energy}_i}\right) \cdot \mathbb{I}[\pi_i^{inf}\text{ valid}] \cdot \big(1 + \delta_{data}\, Q_i^{agg}\big) \cdot M_i^{diversity}
$$

- $Q_i^{agg}$ aggregates the quality of data sources used.
- $M_i^{diversity}$ is a multiplier that rewards decentralised proving infrastructure (computed from the Proving Registry).  
  Specifically, let $HHI$ be the Herfindahl‑Hirschman Index of proving power concentration.  
  $$
  M_i^{diversity} = 1.0 + 0.5 \cdot \max(0,\, 0.25 - HHI)
  $$
  This yields a bonus that disappears when $HHI \ge 0.25$ (high concentration).

### 4.4. Effective Reputation
$$
R_i^{weighted}(t) = 0.2\, R_i^{short} + 0.3\, R_i^{medium} + 0.5\, R_i^{long}
$$
$$
R_i^{cap}(t) = \min\!\Big(R_i^{weighted}(t),\; \kappa \cdot \text{median}\big(R^{weighted}(t)\big)\Big)
$$
$$
R_i^{eff}(t) = \min\!\Big(R_i^{cap}(t) \cdot (1 + 0.5 \cdot \mathbb{I}_{booster}),\; R^{max}\Big)
$$
- $\kappa$ is a cap multiplier set by governance (e.g., 3.0).
- $R^{max}$ is the absolute ceiling.
- $\mathbb{I}_{booster} = 1$ only when a Breakthrough Booster is active (see §7).

**Bootstrapping safeguard:** If the median of $R^{weighted}$ is below a minimum threshold (e.g., $0.01$), the cap is temporarily disabled and all nodes operate with their raw $R^{weighted}$ until the network matures.

### 4.5. Penalty
$$
P_i(t) = \sigma\!\left(\frac{|\text{pred}_i - \mu_{c_i}|}{\sigma_{c_i}}\right) + \lambda_{decay} \cdot ClusterHistory(c_i)
$$
$\sigma(\cdot)$ is the logistic function scaled to $[0,1]$. $ClusterHistory(c_i)$ accumulates a decay‑based penalty for clusters that have historically produced poor calibration.

**Cassandra exception:** If the Cassandra eligibility predicate is satisfied, the first term is set to zero, so the node is not penalised for being far from its cluster centre.

---

## 5. Adaptive Clustering

The DP‑means clustering parameter $\lambda$ adapts to the reputation‑weighted entropy:

$$
\lambda(t) = \lambda_{base} \cdot \left(\frac{H_w(\mathcal{P}_t)}{H_{max}}\right)^{\gamma}
$$

with $H_{max}$ the entropy of a uniform distribution.  
Governance sets $\lambda_{base}$ and $\gamma$; $\gamma$ is constrained to $[0.5,\, 2.0]$ to prevent instability.

### 5.1. Sybil Protection
Nodes with $R_i^{eff} < 0.5 \cdot \text{median}(R^{eff})$ are excluded from the entropy calculation.

### 5.2. Limit on Parameter Change
$$
|\lambda(t) - \lambda(t-1)| \le 0.2 \cdot \lambda_{base}
$$

### 5.3. Entropy Circuit Breaker
If $H_w(\mathcal{P}_t)$ increases for **three consecutive epochs** while exceeding a threshold $H_{trigger}$ (e.g., $0.7 \cdot H_{max}$), the circuit breaker trips:
- $\lambda$ is forced to a conservative minimum $\lambda_{min} = 0.1 \cdot \lambda_{base}$.
- The breaker remains active until $H_w$ drops below $0.5 \cdot H_{max}$ for two consecutive epochs.
- During the breaker period, the change limiter (§5.2) is overridden.

---

## 6. Resolution Uncertainty Layer

### 6.1. Resolution Format
$$
\mathcal{R} = \big\langle \mathcal{D}_{true},\; confidence,\; source\_divergence,\; \mathcal{S},\; T_{final},\; T_{dispute\_end} \big\rangle
$$

### 6.2. Primary Scoring Rule – CRPS
For continuous outcomes:
$$
CRPS(\mathcal{P}, \mathcal{D}_{true}) = \int_{-\infty}^{\infty} \big(F_{\mathcal{P}}(x) - F_{\mathcal{D}_{true}}(x)\big)^2 dx
$$

### 6.3. Secondary Scoring Rule – Log Score
$$
LogScore(\mathcal{P}, \mathcal{D}_{true}) = \log p_{\mathcal{P}}(x_{obs})
$$

### 6.4. Calibration Score
$$
CalibrationScore = 0.7 \cdot \left(1 - \frac{CRPS}{CRPS_{max}}\right) + 0.3 \cdot clip\!\left(\frac{LogScore - \mu_{log}}{2\sigma_{log}},\; -1,\; 1\right)
$$

If $confidence > 0.9$ **and** $CalibrationScore > 0.8$, a confidence bonus is applied:
$$
CalibrationScore \leftarrow 1.1 \cdot CalibrationScore
$$

The score is then clamped to $[0,\, 1.5]$.

**Oracle calibration incentive:** Oracles whose long‑term confidence values systematically exceed their actual accuracy are penalised via a separate reputation‐decay parameter, preventing confidence inflation.

### 6.5. Reward Conversion
Phantom shares are converted into real tokens after the dispute window closes:

1. Compute each node’s raw entitlement:
   $$
   raw\_tokens_i = phantom\_shares_i \cdot CalibrationScore_i \cdot \big(1 + \varepsilon\,(CalibrationScore_i - 0.5)\big)
   $$
   with $\varepsilon \in [0.2,\, 1.0]$ governable.
2. If $CalibrationScore_i < 0.3$, apply a penalty multiplier $0.5\times$.
3. **Proportional scaling:** Let $T_{total}$ be the total tokens allocated to predictors in this epoch (49.5 % of $E(t)$). The final allocation is:
   $$
   final\_tokens_i = T_{total} \cdot \frac{raw\_tokens_i}{\sum_j raw\_tokens_j}
   $$
   This ensures the emission pool is never exceeded.

---

## 7. Cassandra Mechanism

### 7.1. Eligibility
A forecast is eligible for the Cassandra bonus if **all** of the following hold (fixed before resolution):

- $C_1$: It is a statistical outlier (distance $> 2\sigma$ from its cluster centre).
- $C_2$: Its complexity $C_i(t)$ exceeds the network’s 80th percentile.
- $C_3$: $R_i^{eff} \ge 0.5 \cdot \text{median}(R^{eff})$.
- $C_4$: Historical consistency $Consistency_i > 0.7$.
- $C_5$: The question is tagged as **hard**.

### 7.2. Bonus Calculation
The bonus is **continuous**, not binary, and decays smoothly if accuracy is below a high benchmark:

Define an over‑performance margin:
$$
m_i = \max(0,\; CalibrationScore_i - median(CalibrationScore))
$$

The multiplier is:
$$
\kappa_i = 1 + 2 \cdot m_i \cdot f(CalibrationScore_i)
$$
where $f(x)$ is a soft threshold function:
$$
f(x) = \begin{cases}
0, & x \le 0.7 \\
\frac{x - 0.7}{0.25}, & 0.7 < x \le 0.95 \\
1, & x > 0.95
\end{cases}
$$

The final cap is $\kappa_i \le 3.0$.

- At $CalibrationScore_i = 0.8$, $f(0.8) = 0.4$, so with $m_i = 0.2$ the multiplier is $1 + 2\cdot 0.2\cdot 0.4 = 1.16$.
- At $CalibrationScore_i \ge 0.95$ the full bonus is applied.
- Below 0.7 no bonus is possible; no additional penalty is applied (the “catastrophic cliff” is removed).

### 7.3. Limitations
- The bonus is applied only post‑resolution.
- A node may receive the Cassandra bonus at most once every 50 epochs.
- If a node receives the bonus, its subsequent 10 forecasts are flagged for audit; systematic attempts to game the outlier criteria trigger a reputation penalty.

---

## 8. Post‑Resolution Dispute Window

### 8.1. Initiation
A disputer provides:
- a deposit,
- the `question_id`,
- the grounds for dispute,
- an `evidence_hash`.

### 8.2. Committee Composition
The arbitration panel consists of **7 members**:
- 3 randomly selected oracles (excluding those whose resolution is being challenged),
- 2 randomly selected judges,
- 2 members drawn from a dedicated dispute‑resolution pool (non‑oracle, non‑judge participants with high reputation).

This ensures that no single group dominates and that direct conflicts of interest are eliminated.

### 8.3. Verdict
- **To overturn** a resolution: at least 5 of the 7 committee members must concur.
- If the dispute is **rejected**:
  - 50 % of the deposit is burned,
  - 50 % is returned to the disputer,
  - the disputer receives a `frivolous_disputer` flag (accumulated flags increase future deposit requirements).
- If the dispute is **upheld**:
  - the deposit is returned in full,
  - the original oracles are slashed (a portion of their stake is burned and partially awarded to the committee),
  - `CalibrationScore` is re‑calculated,
  - phantom shares are re‑converted atomically.

### 8.4. Atomicity Invariant
$$
\text{dispute.status} \in \{\text{Pending}, \text{Final}\}
$$
Upon transition to `Final`:
- re‑calculation and slashing are executed in a single atomic transaction,
- intermediate states are never exposed.

---

## 9. zkPoI

A predictor constructs:
$$
\pi_i^{inf} = \text{Prove}(model\_hash,\; data\_commitment,\; output,\; witness)
$$

Properties:
- completeness,
- soundness,
- zero‑knowledge.

Supported model types:
- MLP,
- Transformer (up to 6 layers),
- gradient boosting.

Verification target:
- $\le 10$ seconds on a standard CPU in Phase 1.5.

---

## 10. Proving Registry

Each proving node stores:
- address,
- hardware class,
- geographic region,
- attestation hash,
- number of proofs generated,
- success rate,
- average proving time,
- epoch of last activity,
- blacklist flag.

**Consensus‑relevant metrics:**
- $HHI$ (Herfindahl‑Hirschman Index of proof production),
- geographic diversity score (1 – fraction of proofs from top region),
- hardware diversity score (1 – fraction of proofs from top hardware class).

If an attestation does not match the claimed hardware or region, the node is blacklisted and its diversity contribution is zeroed for all epochs until appeal.

---

## 11. zkPoDP

For data sources:
$$
\pi_i^{prov} = \text{Prove}\!\left(\bigwedge_{s \in sources} \text{ValidSignature}(s.data,\; s.sig) \land \text{TimestampInRange}(s.ts)\right)
$$

**Revocation:**
- A revocation certificate is issued by the committee.
- Certificates are accumulated in a Merkle accumulator.
- At commit time, a non‑membership proof against the accumulator is required.
- Historical forecasts are evaluated against the accumulator state at the commit block.

Invariant:
$$
\text{Verify}(\pi^{prov}) = \text{true} \implies \forall s \in sources:\; \text{ValidSignature}(s) \land \neg \text{Revoked}(s, commit\_block)
$$

---

## 12. Energy Arbitration

### 12.1. Energy Attestation
Every predictor must provide, alongside their forecast, a **signed energy attestation** from a registered measurement module (e.g., a trusted hardware enclave or a certified smart‑plug API). The attestation reports energy consumed for the inference task.

### 12.2. Challenge Conditions
A challenge may be filed if:
- the declared energy deviates from the cluster median by more than 30 %, **or**
- the declared energy deviates from the node’s own historical median by more than 50 %.

### 12.3. Arbitration
- Panel: 7 randomly selected energy arbitrators (separate from dispute committee).
- Confirmation threshold: 5/7.
- If the challenge is **confirmed**:
  - 30 % of the node’s phantom shares are burned,
  - the challenger’s deposit is returned with a premium from the burned shares.
- If the challenge is **rejected**:
  - 50 % of the deposit is burned,
  - the challenger receives a `frivolous_challenger` flag.

---

## 13. Tokenomics

Total emission per epoch:
$$
E(t) = E_0 e^{-\lambda t}, \quad E_0 = 10^7,\quad \lambda = 0.05
$$

Maximum total emission:
$$
E_{total} = \frac{E_0}{\lambda} = 2 \times 10^8
$$

**Distribution (governable):**
| Recipient              | Share  |
|------------------------|--------|
| Predictors             | 49.5 % |
| Cassandra Booster Pool | 0.5 %  |
| Judges                 | 20.0 % |
| Oracles                | 15.0 % |
| Data providers         | 10.0 % |
| Proving incentives     | 1.0 %  |
| Treasury               | 4.0 %  |

Predictor rewards are distributed according to §6.5. Any unused portion of the Booster Pool returns to the treasury at epoch end.

---

## 14. Security

The protocol explicitly considers the following attack classes and their countermeasures:

| Attack class              | Primary defence                                      |
|---------------------------|------------------------------------------------------|
| Sybil                     | Reputation‑weighted entropy, min‑stake, cap          |
| Cartel / collusion        | Multi‑level reputation, diversity multipliers        |
| Oracle manipulation       | Uncertainty layer, dispute window, confidence penalty|
| Reputational oligarchy    | Dynamic cap, Breakthrough Boosters, hard R_max       |
| $\lambda$‑attack          | Change limiter, Circuit Breaker, Sybil filter        |
| Structured noise          | Consistency‑weighted novelty, historical audit       |
| Judge conservatism        | Economic reward for overturned resolutions           |
| Energy griefing           | Energy arbitration, FLOPS/energy metric              |
| Proving centralisation    | Proving Registry, HHI‑based diversity multiplier     |
| Critical ZK bug           | Emergency brake via Veto Council, formal verification |

---

## 15. Governance

**DAO‑governed parameters:**
- $\alpha, \beta, \gamma, \delta$ – weight coefficients,
- $\lambda_{base}, \gamma$ – clustering parameters,
- $\varepsilon$ – reward booster,
- scoring thresholds,
- dispute parameters,
- list of authorised data sources and oracles.

**Quorum:** 10 % of circulating supply.  
**Passing threshold:** 66 % approval.  
**Voting period:** 7 days.

**Veto Council:**
- 7 experts,
- rotation every 6 months,
- power to halt emission in an emergency,
- any council decision can be overturned by the DAO within 7 days.
