# Construction Project Procurement — Process Improvement Analysis

> **Five-stage raw material procurement process redesign for a construction project, validated through process flow diagrams, discrete-event simulation, M/G/1 queueing analysis, and Monte Carlo schedule risk modelling.**

---

## Table of Contents

- [Project Overview](#project-overview)
- [Repository Structure](#repository-structure)
- [Methodology](#methodology)
  - [Process Flow Diagrams](#1-process-flow-diagrams-as-is--to-be)
  - [Discrete-Event Simulation](#2-discrete-event-simulation-des)
  - [M/G/1 Queueing Analysis](#3-mg1-queueing-analysis)
  - [Monte Carlo Schedule Risk](#4-monte-carlo-schedule-risk-modelling)
- [Key Results](#key-results)
- [Diagrams](#diagrams)
- [Data Collection & Assumptions](#data-collection--assumptions)
- [Dependencies](#dependencies)

---

## Project Overview

This project analyses and redesigns the raw material procurement process for a construction project across five functional stages:

| Stage | Description |
|-------|-------------|
| **1 — Purchasing Request** | Need identification through to budget approval and specification |
| **2 — Bidding** | Supplier shortlisting, RFQ issuance, and quotation receipt |
| **3 — Evaluation** | Technical and financial review through to supplier qualification |
| **4 — Negotiations** | Contract negotiation, finalisation, and agreement signing |
| **5 — Purchase Order** | Release of PO for 2,000 units |

The AS-IS process (17 stages, 4 rework decision gates) was analysed for bottlenecks, queue instability, and rework cost. A TO-BE process was designed to address the findings through automation, digitalisation, and parallel review tracks. The improvement was validated statistically through simulation, not assumed.

---

## Repository Structure

```
├── procurement_flow_FIXED.py         # AS-IS & TO-BE process flow diagrams
├── procurement_analytics_v3.py       # Systems engineering analytics dashboard
├── procurement_monte_carlo.py        # Monte Carlo schedule risk dashboard
├── outputs/
│   ├── procurement_flow_FIXED.png    # Flow diagram figure (AS-IS + TO-BE)
│   ├── procurement_analytics_v3.png  # Analytics dashboard (7 charts + KPI panel)
│   └── procurement_monte_carlo.png   # Monte Carlo dashboard (7 charts)
└── README.md
```

---

## Methodology

### 1. Process Flow Diagrams (AS-IS / TO-BE)

**File:** `procurement_flow_FIXED.py`

Two swim-lane flow diagrams were produced — one for the current manual process (AS-IS) and one for the improved/automated process (TO-BE). Each diagram shows:

- All 17 process nodes with assigned owner roles (Site Manager, Finance Dept, Procurement Team, Evaluation Committee, Contracts Manager, Legal/Director, Director/PM)
- Four decision gates with rework loop probabilities
- Directional flow (left-to-right within each stage, top-to-bottom across stages)
- Rework paths in red with rejection rates labelled
- Approved paths in green
- TO-BE improvements highlighted in teal with callout badges showing per-stage cycle time reductions
- Stats strip at the bottom of each diagram: mean cycle time, standard deviation, P95, and average rework loops per job

The diagrams are rendered at 24×30 inches at 160 dpi using `matplotlib` with custom drawing primitives (rounded rectangles, decision diamonds, polyline arrows with arrowheads).

---

### 2. Discrete-Event Simulation (DES)

**File:** `procurement_analytics_v3.py`

A discrete-event flow simulation was implemented from scratch to model 2,000 procurement jobs passing serially through all 17 stages. Each stage samples service time from a truncated Normal distribution (minimum 0.5 days).

**Rework model:** Decision gates use a geometric loop model. At each gate, the job re-enters the loop with probability `p` until it passes. Each rework iteration costs 80% of the base service time (learning-curve assumption).

```
E[rework delay] = p / (1 − p) × μ × 0.8
```

**Parameters used:**

| Stage | AS-IS μ (days) | TO-BE μ (days) | Improvement |
|-------|---------------|---------------|-------------|
| Budget Approval | 10 | 5 | ↓ 50% |
| Shortlist Suppliers | 12 | 5 | ↓ 58% |
| RFQ | 14 | 7 | ↓ 50% |
| Technical Review | 10 | 6 | ↓ 40% |
| Financial Review | 9 | 6 | ↓ 33% |
| Negotiations | 15 | 8 | ↓ 47% |
| Sign Agreement | 4 | 2 | ↓ 50% |

**Rework probability reductions:**

| Decision Gate | AS-IS | TO-BE | Reduction |
|---------------|-------|-------|-----------|
| Budget Approval | 30% | 10% | ↓ 67% |
| Shortlist Suppliers | 25% | 8% | ↓ 68% |
| Qualified Suppliers | 20% | 7% | ↓ 65% |
| Negotiations Finalized | 35% | 12% | ↓ 66% |

**Simulation outputs** (per job, averaged over 2,000 runs):

| Metric | AS-IS | TO-BE | Δ |
|--------|-------|-------|---|
| Mean cycle time | 138.8 days | 93.6 days | **−45.3 days (↓ 32.6%)** |
| Std deviation | 17.9 days | 9.0 days | ↓ 50% |
| P95 worst case | 169.4 days | 108.7 days | ↓ 36% |
| Avg rework loops/job | 1.56 | 0.40 | ↓ 75% |

---

### 3. M/G/1 Queueing Analysis

**File:** `procurement_analytics_v3.py`

Each stage was modelled as an independent M/G/1 queue with arrival rate λ = 0.12 jobs/day (approximately one procurement job every eight working days).

**Utilisation factor:** `ρ = λ · E[S]`

**Queue waiting time** (Pollaczek-Khinchine formula):

```
Wq = ρ / (1 − ρ) · E[S] / 2 · (1 + Cs²)
```

where `Cs² = Var[S] / E[S]²` is the squared coefficient of variation of service time.

When `ρ ≥ 1` the queue is theoretically unstable (Wq → ∞). The AS-IS process has **7 saturated stages**; the TO-BE reduces this to **1 marginal stage** (Negotiations Finalized, ρ = 1.04).

**AS-IS saturated stages (ρ ≥ 1.0):**

| Stage | ρ (AS-IS) | ρ (TO-BE) |
|-------|-----------|-----------|
| Shortlist Suppliers | 1.87 | 0.65 |
| Negotiations | 1.81 | 0.95 |
| RFQ | 1.68 | 0.84 |
| Budget Approval | 1.63 | 0.66 |
| Negotiations Finalized | 1.38 | 1.04 ⚠ |
| Technical Review | 1.20 | 0.72 |
| Financial Review | 1.10 | 0.72 |

The analytics dashboard presents: bottleneck map (utilisation sorted by severity), Wq per stage, process variability (CoV = σ/μ), rework cost per gate, improvement waterfall, and a one-at-a-time sensitivity tornado.

---

### 4. Monte Carlo Schedule Risk Modelling

**File:** `procurement_monte_carlo.py`

A separate Monte Carlo simulation was built following the same 17-stage structure, adding two additional stochastic layers on top of the DES:

- **External risk events:** Each stage has a calibrated risk event probability (scaled from σ/μ and ρ). Risk penalties are sampled from a log-normal distribution.
- **Queue wait noise:** M/D/1 queue wait is sampled with ±30% stochastic variation per contract.

The engine runs **2,000 simulations × 500 contracts per simulation** (1,000,000 contract instances total). Service times are sampled from PERT-fitted Beta distributions.

**Schedule risk outputs:**

| Metric | AS-IS | TO-BE | Δ |
|--------|-------|-------|---|
| P50 (median schedule) | 434 days | 228 days | **↓ 47.5%** |
| P90 (near-worst case) | 467 days | 245 days | ↓ 47.5% |
| Risk factor P90/P50 | 1.07× | 1.07× | Maintained |

> **Note:** The DES cycle time (≈139 days) and Monte Carlo P50 (434 days) operate at different scales. The DES measures pure flow time per job. The Monte Carlo adds queue wait inflation, risk event penalties, and rework admin delays across the full contract population, producing higher aggregate durations. Both point to the same relative improvement (~33–48% compression) from the TO-BE interventions.

**Top schedule risk drivers (AS-IS, Pearson correlation with total duration):**

| Rank | Stage | r |
|------|-------|---|
| 1 | Shortlist Suppliers | 0.43 |
| 2 | Negotiations | 0.41 |
| 3 | RFQ | 0.39 |
| 4 | Budget Approval | 0.31 |

These four stages are all addressed in the TO-BE process, confirming the redesign targets the highest-impact nodes.

---

## Key Results

```
┌─────────────────────────────────────────────────────────────────┐
│                    HEADLINE IMPROVEMENTS                        │
├──────────────────────────────┬──────────────┬───────────────────┤
│ Metric                       │   Change     │   Impact          │
├──────────────────────────────┼──────────────┼───────────────────┤
│ Mean cycle time (DES)        │ 139d → 94d   │ ↓ 32.6%          │
│ Schedule P50 (Monte Carlo)   │ 434d → 228d  │ ↓ 47.5%          │
│ Std deviation                │ 18d → 9d     │ ↓ 50%            │
│ P95 worst case               │ 169d → 109d  │ ↓ 36%            │
│ Avg rework loops per job     │ 1.56 → 0.40  │ ↓ 75%            │
│ Saturated queue stages (ρ≥1) │ 7 → 1        │ ↓ 86%            │
│ Stages with Wq → ∞           │ 7 → 0        │ Eliminated       │
└──────────────────────────────┴──────────────┴───────────────────┘
```

---

## Diagrams

### Process Flow Diagrams

Two swim-lane diagrams showing all 17 process nodes, 4 decision gates, rework paths, and improvement callouts.

![Process Flow Diagrams](outputs/procurement_flow_FIXED.png)

---

### Systems Engineering Analytics Dashboard

Eight-panel dashboard: cycle time distribution, bottleneck map (ρ), M/G/1 queue waiting time, rework cost, improvement waterfall, process variability (CoV), sensitivity tornado, and KPI impact panel.

![Analytics Dashboard](outputs/procurement_analytics_v3.png)

---

### Monte Carlo Schedule Risk Dashboard

Seven-panel dashboard: schedule distribution, KPI panel, step duration breakdowns (AS-IS & TO-BE), bottleneck queue wait comparison, S-curve CDF, and schedule risk tornado (Pearson correlation).

![Monte Carlo Dashboard](outputs/procurement_monte_carlo.png)

---

## Data Collection & Assumptions

### Service Time Parameters

Stage durations (μ, σ) were calibrated using a combination of:

- **Industry benchmarks** for construction procurement (RFQ periods, evaluation timelines, contract award durations)
- **Subject matter expert estimates** for administrative steps (PR submission, budget approval, sign-off)
- **PERT heuristics** — pessimistic = μ + 3σ, optimistic = max(0.5, μ − 3σ) — to constrain the Beta distribution to realistic bounds

All service times are floored at 0.5 days (minimum half-day effort).

### Arrival Rate

λ = 0.12 jobs/day was selected to represent a project with approximately one new procurement request every eight working days, consistent with a mid-scale construction project procuring multiple material categories concurrently.

### Rework Probabilities

Rework gate probabilities were estimated from:

- Typical budget approval rejection rates in public-sector construction (30% AS-IS, reduced to 10% with auto-check controls)
- Supplier shortlisting failure rates driven by incomplete market mapping (25% AS-IS, 8% TO-BE with e-sourcing registry)
- Supplier disqualification rates at technical/financial review (20% AS-IS, 7% TO-BE with pre-qualification)
- Negotiation breakdown rates (35% AS-IS — highest in the process — reduced to 12% TO-BE with structured e-negotiation)

### TO-BE Intervention Assumptions

| Intervention | Mechanism | Basis |
|---|---|---|
| Auto Budget Check | ERP integration, pre-validated cost models | Eliminates manual finance routing |
| Digital E-Sourcing | Supplier registry with pre-qualification data | Reduces shortlisting from 12d to 5d |
| RFQ Platform | Standardised e-solicitation, auto-bid comparison | Reduces RFQ from 14d to 7d |
| Parallel Reviews | Simultaneous technical + financial evaluation | Replaces sequential 19d with parallel 6d each |
| e-Negotiate | Structured digital negotiation with clause library | Reduces negotiation from 15d to 8d |
| e-Signature | Digital signing workflow | Reduces sign agreement from 4d to 2d |

### Simulation Design

| Parameter | Value | Rationale |
|---|---|---|
| DES runs | 2,000 jobs | Sufficient for stable mean and percentile estimates |
| Monte Carlo simulations | 2,000 × 500 contracts | 1M contract instances for robust tail distribution |
| Rework iteration cost | 0.8 × μ | Learning-curve assumption — second attempt is faster |
| Random seeds | Fixed (42 / 99) | Reproducible results |
| Service time distribution | Truncated Normal (DES) / PERT Beta (MC) | PERT preferred for MC as it enforces finite support |

---

## Dependencies

```
numpy
pandas
matplotlib
scipy
```
