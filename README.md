# TRAFFIC-X

## Network-Aware Urban Traffic Intelligence & Counterfactual Simulation

**TRAFFIC-X** is a software-only traffic intelligence platform built specifically around the supplied **NeuraX Smart Cities Dataset v2 - Training**.

The dataset contains 436 road segments, 120 nodes, 5-minute traffic observations, temporal train/validation splits, separate forecast labels, incidents, context, roadworks, signal plans, turn restrictions, OD demand profiles, planning candidates and scenario examples.

The product is not a navigation app and not a live traffic-control system.

Its core loop is:

> **Observe → Diagnose → Forecast → Propagate → Simulate → Compare → Recommend**

---

# 1. Problem Understanding

Urban congestion changes rapidly and can be caused by both temporary events and persistent network limitations.

TRAFFIC-X addresses five questions:

1. **What is happening now?**
2. **Why is it happening?**
3. **What is likely to happen in 15–60 minutes?**
4. **How could the congestion propagate through the network?**
5. **What simulated response or planning intervention changes the outcome?**

The final system must therefore combine forecasting with network reasoning and counterfactual simulation.

---

# 2. Dataset Facts

From `DATASET_MANIFEST.json`:

```text
Road segments:             436
Network nodes:              120
Training days:               15
Validation days:              4
Hidden test days:             8
Resolution:                   5 minutes

Training traffic rows:  1,883,520
Validation traffic rows: 502,272
Hidden test rows:       1,004,544
Hidden test scenarios:       36
```

The dataset also includes explicit noise:

```text
missing values
duplicates
spikes
stuck sensors
impossible negative readings
row shuffle
```

Therefore data validation is a first-class part of the implementation.

---

# 3. Dataset Files

```text
traffic_train.csv
forecast_targets_train.csv

traffic_validation.csv
forecast_targets_validation.csv

incidents_train.csv
incidents_validation.csv

context_train.csv
context_validation.csv

roadworks_train.csv
roadworks_validation.csv

network.csv
nodes.csv
signal_plans.csv
turn_restrictions.csv
planning_candidates.csv
od_demand_profiles.csv
scenario_examples.csv
```

---

# 4. Critical Forecast Leakage Rule

The forecast target files are **not model input features**.

```text
forecast_targets_train.csv
forecast_targets_validation.csv
```

are used only for:

```text
training labels
validation ground truth
```

Correct:

```text
Traffic + Network + Context + Incidents + Roadworks
+ OD + Signals + Restrictions
            ↓
      Feature Engineering
            ↓
        ML Model
            ↓
       Forecasts
            ↓
Compare against forecast_targets_*
```

Never create features from future target columns.

---

# 5. Traffic Digital Twin

The system represents the 436-segment road network as a directed graph.

```text
N001 ── R0001 ──> N002
                    │
                    R0003
                    ↓
                   N010
```

Each segment stores:

```text
speed
flow
occupancy
travel time
delay
queue proxy
congestion
capacity
free-flow speed
lanes
importance
road class
```

The traffic state is continuously updated from the supplied observations.

---

# 6. Architecture

```text
                       DATASETS
                           |
                           v
                  +------------------+
                  | Data Validation  |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Feature Engine   |
                  +--------+---------+
                           |
                           v
                  +------------------+
                  | Network Digital  |
                  | Twin / Graph      |
                  +--------+---------+
                           |
          +----------------+----------------+
          |                |                |
          v                v                v
      Traffic          Incident        Historical
       State            /Anomaly        Analysis
          |                |                |
          +----------------+----------------+
                           |
                           v
                    FORECAST ENGINE
                    15/30/45/60 min
                           |
                           v
                  PROPAGATION ENGINE
                           |
                 +---------+---------+
                 |                   |
                 v                   v
          DIVERSION ENGINE     PLANNING ENGINE
                 |                   |
                 +---------+---------+
                           |
                           v
                 COUNTERFACTUAL
                    SIMULATOR
                           |
                           v
                    IMPACT ENGINE
                           |
                           v
                   EXPLAINABILITY
                           |
                           v
                     WEB DASHBOARD
```

---

# 7. Implementation Approach

## Step 1 — Validate data

Because the supplied dataset explicitly contains noise, first build:

```text
Schema validation
Timestamp validation
Duplicate detection
Range validation
Missing-value report
Sensor-quality analysis
Sorting
```

Do not silently discard problematic records.

---

## Step 2 — Build the network graph

Use:

```text
network.csv
nodes.csv
turn_restrictions.csv
signal_plans.csv
```

Build a directed graph:

```text
source_node → target_node
```

Store all network attributes on the edge.

Turn restrictions must be respected by routing and simulation.

---

## Step 3 — Build current traffic state

Join:

```text
traffic observations
+
network metadata
+
context
+
incident state
+
roadwork state
```

For every segment calculate:

```text
current speed
current flow
capacity utilization
delay
queue proxy
congestion
traffic state
```

---

# 8. Traffic State

Use the supplied `congestion_index` as an important observation/model feature rather than blindly replacing it with a new formula.

Possible UI states:

```text
NORMAL
SLOW
HEAVY
CONGESTED
CRITICAL
ABNORMAL
UNKNOWN
```

Thresholds should be derived/validated from the actual training distribution.

---

# 9. Incident Reasoning

Incident records have:

```text
start_time
end_time
segment_id
incident_type
severity
lanes_blocked
```

Use time-aware interval joins.

For each traffic observation, determine whether an incident is active.

Combine incident information with:

```text
speed changes
flow changes
queue changes
neighbor conditions
roadworks
context
```

The system should distinguish:

```text
Observed incident
Possible abnormal event
Traffic anomaly
Unknown cause
```

Do not claim an accident if the available data only supports an anomaly.

---

# 10. Forecasting

Forecast:

```text
+15 min
+30 min
+45 min
+60 min
```

For each horizon:

```text
speed
flow
congestion
```

Start with:

```text
Historical baseline
↓
LightGBM / XGBoost
↓
Validation
```

Then consider:

```text
Spatio-temporal GNN
```

only if it provides measurable validation improvement.

---

# 11. Feature Engineering

Use:

### Traffic history

```text
lagged speed
lagged flow
lagged congestion
rolling averages
rolling standard deviation
rate of change
```

### Network

```text
capacity
lanes
road class
importance
structural bottleneck
upstream state
downstream state
neighbor congestion
```

### Context

```text
temperature
rain
event level
holiday
day of week
time of day
```

### Disruptions

```text
incident active
incident severity
lanes blocked
roadwork active
closure fraction
```

### Demand

```text
OD demand
origin demand
destination demand
affected OD flows
```

### Signal/network metadata

```text
cycle
green ratio
offset
turn restrictions
```

---

# 12. Congestion Propagation

Use the network graph to estimate how congestion can spread.

Example:

```text
R14 🔴
 |
 +----> R15 🟠
 |
 +----> R16 🟡
```

The engine combines:

```text
current congestion
forecast congestion
capacity
connectivity
OD demand
historical behavior
```

Output:

```text
origin segment
affected segments
expected time window
propagation risk
```

---

# 13. Diversion Simulation

TRAFFIC-X is not a navigation app.

It evaluates **network-level diversion scenarios**.

Workflow:

```text
Congested corridor
       ↓
Affected OD flows
       ↓
Feasible alternatives
       ↓
Check capacity
       ↓
Apply simulated demand shift
       ↓
Recalculate network
       ↓
Compare baseline
```

Turn restrictions must be respected.

---

# 14. Counterfactual Simulation

Supported scenario concepts include:

```text
incident
road closure
capacity reduction
roadwork
demand surge
OD diversion
event
alternative signal plan
planning candidate
```

Every simulation runs against a copy of the baseline.

The original dataset is never changed.

---

# 15. Planning Intelligence

The supplied `planning_candidates.csv` contains:

```text
candidate_id
target_segment
intervention_type
capacity_delta_vph
cost_index
feasibility_band
```

Use these candidates for long-term scenario analysis.

Example:

```text
Recurring bottleneck
       ↓
Planning candidate
       ↓
Simulated capacity change
       ↓
Network recalculation
       ↓
Before / after impact
```

No physical construction is performed.

---

# 16. Signal Plans

Signal plans are data for analysis/simulation.

The system can test:

```text
Existing plan
vs
Simulated alternative plan
```

It must never:

```text
connect to a signal controller
send signal commands
claim a signal was actually changed
```

---

# 17. Impact Metrics

For every counterfactual scenario calculate:

### Network

```text
average speed
average travel time
total travel time
critical segment count
congested segment count
queue proxy
network throughput
spillback risk
```

### OD

```text
affected demand
OD travel time
overloaded demand
```

### Local

```text
segment delay
junction delay
capacity utilization
```

---

# 18. Dashboard

The main application contains:

```text
Overview
Forecast
Incidents
Simulate
Bottlenecks
Planning
Data Quality
```

### Overview

Shows:

```text
network map
average speed
critical roads
active incidents
network delay
forecast risk
```

### Forecast

Shows:

```text
Now
+15
+30
+45
+60
```

### Incidents

Shows:

```text
incident
evidence
confidence
affected roads
propagation
```

### Simulate

Shows:

```text
scenario configuration
network preview
run simulation
before/after comparison
```

### Bottlenecks

Shows:

```text
recurrence
severity
time window
incident-adjusted persistence
```

### Planning

Shows:

```text
planning candidates
simulation impact
cost index
feasibility band
```

### Data Quality

Shows:

```text
missing data
duplicates
invalid values
sensor-quality issues
target isolation
```

---

# 19. Recommended Technology Stack

| Layer | Technology |
|---|---|
| Frontend | Next.js + TypeScript |
| Styling | Tailwind CSS |
| UI | shadcn/ui |
| Map | MapLibre GL JS |
| Charts | Recharts |
| Backend | FastAPI |
| ML | XGBoost / LightGBM |
| Advanced ML | PyTorch / PyTorch Geometric |
| Network | NetworkX |
| Data | Pandas / NumPy |
| Database | PostgreSQL + PostGIS |
| Simulation | Custom Python network-flow engine |
| Optional simulation | SUMO |
| Deployment | Vercel + Render + PostgreSQL |

---

# 20. Repository Structure

```text
traffic-x/
│
├── frontend/
│   ├── app/
│   ├── components/
│   ├── features/
│   ├── lib/
│   └── types/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── services/
│   │   ├── schemas/
│   │   └── core/
│   │
│   ├── ml/
│   │   ├── preprocessing/
│   │   ├── features/
│   │   ├── forecasting/
│   │   ├── anomaly/
│   │   └── evaluation/
│   │
│   ├── network/
│   │   ├── graph.py
│   │   ├── routing.py
│   │   └── propagation.py
│   │
│   └── simulation/
│       ├── engine.py
│       ├── scenarios.py
│       └── impact.py
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── schemas/
│
├── notebooks/
├── tests/
└── docs/
```

---

# 21. Development Phases

## Phase 1 — Dataset foundation

```text
Inspect
→ Validate
→ Clean
→ Build schema
→ Prevent leakage
```

## Phase 2 — Network

```text
Graph
→ Traffic mapping
→ Turn restrictions
→ Map
```

## Phase 3 — AI

```text
State detection
→ Anomaly detection
→ Incident reasoning
→ Forecasting
```

## Phase 4 — Network intelligence

```text
Propagation
→ OD reasoning
→ Diversion
```

## Phase 5 — Counterfactuals

```text
Scenario engine
→ Simulation
→ Before/after
```

## Phase 6 — Planning

```text
Bottlenecks
→ Planning candidates
→ Candidate simulation
```

## Phase 7 — Product

```text
Dashboard
→ Explainability
→ Demo scenarios
→ Testing
```

---

# 22. Evaluation

The validation period is temporal:

```text
Training:
2026-01-01 → 2026-01-15

Validation:
2026-01-16 → 2026-01-19
```

Do not randomly shuffle rows into train/validation.

Forecast metrics:

```text
MAE
RMSE
WAPE
```

Evaluate separately for:

```text
15m
30m
45m
60m
```

Also evaluate:

```text
traffic-state classification
incident detection
propagation accuracy
```

where appropriate labels are available.

---

# 23. Demo Story

The best demo is an end-to-end network event.

```text
Normal network
      ↓
Disruption
      ↓
Traffic anomaly
      ↓
Incident reasoning
      ↓
15–60 min forecast
      ↓
Congestion propagation
      ↓
Diversion simulation
      ↓
Before / after impact
      ↓
Recurring bottleneck
      ↓
Planning candidate
      ↓
Counterfactual comparison
```

The judge should see the entire intelligence loop rather than only a forecasting graph.

---

# 24. What Makes TRAFFIC-X Different

TRAFFIC-X is not:

```text
Google Maps
Navigation app
Generic chatbot
Live signal controller
CCTV system
GPS tracker
Roadside IoT system
```

It is:

> **A software-only network-aware AI decision-support and counterfactual simulation platform.**

Its differentiator is:

```text
Traffic State
+
Forecasting
+
Incident Reasoning
+
Network Propagation
+
OD-aware Diversion
+
Counterfactual Simulation
+
Recurring Bottlenecks
+
Planning Evaluation
+
Explainability
```

---

# 25. Final Product Statement

> **TRAFFIC-X uses the supplied 5-minute, network-level traffic observations and supporting datasets to understand current urban traffic conditions, detect abnormal behavior, reason about incidents and roadworks, forecast speed/flow/congestion 15–60 minutes ahead, estimate congestion propagation, simulate OD-aware diversion strategies, identify recurring bottlenecks, and evaluate provided planning candidates through before/after counterfactual analysis.**

The product's core philosophy is:

> **Don't just predict traffic. Understand the network, predict what happens next, safely test what-if scenarios, and show the evidence behind every recommendation.**
