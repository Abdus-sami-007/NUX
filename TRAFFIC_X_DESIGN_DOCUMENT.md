# TRAFFIC-X — Technical Design Document

## 1. Overview

**TRAFFIC-X** is a software-only AI urban traffic intelligence and counterfactual simulation platform designed for a Hyderabad-like urban road environment.

The system uses the provided traffic, network, incident, context, roadwork, signal, turn-restriction, OD-demand, and planning-candidate datasets to:

- reconstruct the current traffic state,
- identify supported incidents and abnormal traffic conditions,
- forecast traffic conditions at **15, 30, 45, and 60 minutes**,
- reason about congestion propagation through the road network,
- estimate affected OD flows and feasible diversions,
- simulate traffic-management and planning interventions,
- identify recurring bottlenecks,
- compare baseline and counterfactual outcomes,
- present results through an explainable dashboard.

The platform is strictly simulation/advisory. It does **not** require or perform live signal control, camera access, GPS-device integration, roadside sensor integration, municipal infrastructure access, or physical construction.

---

## 2. Design Goals

### 2.1 Primary goals

1. Build a network-aware representation of the supplied 436-segment, 120-node road network.
2. Clean and validate noisy training/validation observations.
3. Produce leakage-safe traffic features.
4. Forecast speed, flow, and congestion for four future horizons.
5. Detect and classify incidents only when supported by the supplied incident/evidence data.
6. Estimate congestion propagation over connected network segments.
7. Evaluate diversion and counterfactual scenarios.
8. Identify recurring structural bottlenecks.
9. Evaluate supplied planning candidates using simulated capacity changes.
10. Clearly distinguish observed, forecast, estimated, simulated, assumed, and target values.

### 2.2 Non-goals

- Live traffic-signal control.
- Real-time camera processing.
- GPS-device tracking.
- Direct roadside sensor integration.
- Municipal infrastructure control.
- Physical road construction.
- Individual turn-by-turn navigation.

---

## 3. Dataset Foundation

The implementation must use the supplied dataset as the source of truth.

### 3.1 Dataset scale

- **436 road segments**
- **120 nodes**
- **15 training days**
- **4 validation days**
- **8 hidden test days**
- **5-minute resolution**
- **1,883,520 training traffic observations**
- **502,272 validation traffic observations**
- **36 test scenarios**

The dataset includes noise such as missing values, duplicates, spikes, stuck sensors, impossible negative readings, and shuffled rows.

### 3.2 Main data sources

| File | Purpose |
|---|---|
| `traffic_train.csv` | Historical traffic observations |
| `traffic_validation.csv` | Validation traffic observations |
| `forecast_targets_train.csv` | Forecast labels |
| `forecast_targets_validation.csv` | Validation labels |
| `network.csv` | Road-segment topology and physical attributes |
| `nodes.csv` | Node coordinates |
| `incidents_train.csv` | Training incident intervals |
| `incidents_validation.csv` | Validation incident intervals |
| `context_train.csv` | Weather/event/calendar context |
| `context_validation.csv` | Validation context |
| `roadworks_train.csv` | Training roadwork intervals |
| `roadworks_validation.csv` | Validation roadwork intervals |
| `signal_plans.csv` | Signal timing metadata |
| `turn_restrictions.csv` | Routing restrictions |
| `planning_candidates.csv` | Candidate interventions |
| `od_demand_profiles.csv` | OD demand profiles |
| `scenario_examples.csv` | Example simulation scenarios |

**Important:** `forecast_targets_*` are labels and must never be used as model input features.

---

## 4. High-Level Architecture

```text
                    ┌──────────────────────────┐
                    │      Supplied Dataset    │
                    └────────────┬─────────────┘
                                 │
                                 ▼
                    ┌──────────────────────────┐
                    │ Data Validation & Quality│
                    └────────────┬─────────────┘
                                 │
              ┌──────────────────┴──────────────────┐
              ▼                                     ▼
     ┌──────────────────┐                  ┌──────────────────┐
     │ Temporal Features│                  │ Network Graph    │
     └─────────┬────────┘                  └─────────┬────────┘
               │                                     │
               └────────────────┬────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │ Current Traffic State   │
                    └────────────┬─────────────┘
                                 │
               ┌─────────────────┼─────────────────┐
               ▼                 ▼                 ▼
      ┌────────────────┐ ┌───────────────┐ ┌───────────────┐
      │ Incident/      │ │ Forecasting   │ │ Context &     │
      │ Anomaly Logic  │ │ 15–60 min     │ │ Roadworks     │
      └───────┬────────┘ └───────┬───────┘ └───────┬───────┘
              └──────────────────┼─────────────────┘
                                 ▼
                    ┌──────────────────────────┐
                    │ Propagation Engine       │
                    └────────────┬─────────────┘
                                 ▼
                    ┌──────────────────────────┐
                    │ OD-Aware Diversion       │
                    └────────────┬─────────────┘
                                 ▼
                    ┌──────────────────────────┐
                    │ Counterfactual Simulator │
                    └────────────┬─────────────┘
                                 ▼
              ┌──────────────────┴──────────────────┐
              ▼                                     ▼
     ┌──────────────────┐                  ┌──────────────────┐
     │ Recurring        │                  │ Planning         │
     │ Bottlenecks      │                  │ Candidates       │
     └─────────┬────────┘                  └─────────┬────────┘
               └────────────────┬────────────────────┘
                                ▼
                    ┌──────────────────────────┐
                    │ Explainable Dashboard    │
                    └──────────────────────────┘
```

---

## 5. Core System Loop

TRAFFIC-X follows:

> **Observe → Diagnose → Forecast → Propagate → Simulate → Compare → Recommend**

### Observe
Construct the latest known state from traffic observations and contextual data.

### Diagnose
Use incidents, roadworks, traffic deviations, network structure, and context to explain abnormal conditions.

### Forecast
Predict traffic conditions at +15, +30, +45, and +60 minutes.

### Propagate
Estimate how congestion or disruption can spread to connected segments.

### Simulate
Apply a scenario or intervention to a copy of the baseline network state.

### Compare
Calculate before/after network-level and segment-level impacts.

### Recommend
Present evidence-backed simulated responses and planning insights.

---

# 6. Data Engineering Design

## 6.1 Data ingestion

Each CSV is loaded into a typed data-processing layer.

The ingestion layer must:

1. Parse timestamps.
2. Normalize identifiers.
3. Validate required columns.
4. Detect duplicate records.
5. Detect missing values.
6. Detect impossible numerical values.
7. Preserve source data separately from cleaned data.
8. Record quality statistics.

Recommended processing stack:

- Pandas
- NumPy
- Pydantic for API/data validation

---

## 6.2 Traffic data validation

For every traffic observation:

```text
timestamp
segment_id
speed_kmh
flow_vph
occupancy_pct
travel_time_min
free_flow_time_min
delay_min
queue_length_veh
congestion_index
sensor_quality
```

Quality checks should include:

- missing speed/flow/occupancy,
- negative speed,
- negative flow,
- invalid occupancy,
- inconsistent travel time,
- duplicate timestamp-segment records,
- abnormal spikes,
- stuck sensor behavior,
- low sensor quality.

Invalid observations should not simply be deleted. The pipeline should record whether values were:

- valid,
- repaired,
- imputed,
- rejected,
- unavailable.

---

## 6.3 Temporal feature engineering

Features should be generated only from information available at the forecast origin.

Examples:

### Traffic lags

```text
speed_lag_5m
speed_lag_10m
speed_lag_15m
speed_lag_30m
flow_lag_5m
flow_lag_15m
congestion_lag_5m
congestion_lag_15m
```

### Rolling statistics

```text
speed_mean_15m
speed_mean_30m
speed_mean_60m
flow_mean_15m
congestion_mean_15m
congestion_max_30m
```

### Time features

```text
hour
day_of_week
minute_of_day
is_peak
holiday_flag
```

### Context

```text
temperature_c
rain_intensity
event_level
event_id
```

No future target values may enter these features.

---

# 7. Network Graph Design

The road network is represented as a directed graph:

```text
source_node → target_node
```

Each edge corresponds to a segment from `network.csv`.

### Edge attributes

```text
segment_id
road_class
lanes
free_flow_speed_kmh
capacity_vph
length_km
grade_pct
signal_id
structural_bottleneck
importance
peak_capacity_factor
```

### Node attributes

From `nodes.csv`:

```text
node_id
x
y
lat
lon
```

NetworkX can be used for the initial implementation.

---

# 8. Current Traffic State Engine

For each segment, the system maintains a state object:

```text
segment_id
timestamp
speed
flow
occupancy
travel_time
delay
queue_length
congestion_index
capacity
sensor_quality
incident_state
roadwork_state
```

### Traffic state classification

The UI may expose states such as:

```text
NORMAL
SLOW
HEAVY
CONGESTED
CRITICAL
ABNORMAL
UNKNOWN
```

Thresholds should be derived and validated from the supplied training distribution rather than chosen arbitrarily.

---

# 9. Incident Reasoning

Incidents are interval-based.

Each incident contains:

```text
incident_id
start_time
end_time
segment_id
incident_type
severity
lanes_blocked
```

The feature pipeline performs a time-aware interval join between traffic timestamps and incident intervals.

### Incident features

```text
incident_active
incident_type
incident_severity
lanes_blocked
incident_duration
```

### Evidence hierarchy

The system should distinguish:

1. **Observed incident** — directly supported by the incident dataset.
2. **Traffic anomaly** — supported by unusual traffic behavior.
3. **Possible cause** — plausible but not directly confirmed.
4. **Unknown cause** — insufficient evidence.

The system must not claim that an accident occurred merely because congestion increased.

---

# 10. Roadwork Reasoning

Roadworks are also interval-based:

```text
work_id
segment_id
start_time
end_time
closure_fraction
work_type
```

Derived features:

```text
roadwork_active
closure_fraction
work_type
```

Roadwork effects can modify effective segment capacity in simulation.

---

# 11. Forecasting Architecture

## 11.1 Forecast targets

The system predicts:

### +15 minutes

```text
speed
flow
congestion
```

### +30 minutes

```text
speed
flow
congestion
```

### +45 minutes

```text
speed
flow
congestion
```

### +60 minutes

```text
speed
flow
congestion
```

---

## 11.2 Baseline model

The first implementation should use:

- historical/lag baseline,
- LightGBM or XGBoost,
- temporal validation.

A more advanced spatio-temporal model can be introduced only if validation demonstrates measurable improvement.

Potential advanced stack:

- PyTorch
- PyTorch Geometric
- graph-based temporal forecasting

---

## 11.3 Forecast feature groups

### Historical traffic

- lagged speed
- lagged flow
- lagged congestion
- rolling averages
- rolling maxima
- traffic trend

### Network

- lanes
- capacity
- free-flow speed
- road class
- importance
- structural bottleneck
- peak capacity factor

### Neighbor state

- upstream congestion
- downstream congestion
- neighboring speed
- neighboring flow

### Context

- weather
- event level
- holiday
- hour/day

### Incidents

- active incident
- severity
- blocked lanes
- incident type

### Roadworks

- active roadwork
- closure fraction
- work type

### Demand

- relevant OD demand features

---

# 12. Forecast Evaluation

The temporal split must be preserved.

```text
Training:
2026-01-01 → 2026-01-15

Validation:
2026-01-16 → 2026-01-19
```

Evaluation metrics:

- MAE
- RMSE
- WAPE where appropriate

Metrics should be reported by:

- horizon,
- target,
- peak/non-peak where useful,
- segment/network aggregate.

Random train/test splitting should not be used for the main time-series evaluation.

---

# 13. Propagation Engine

The propagation engine uses the directed road graph.

### Basic flow

```text
Affected segment
      ↓
Find connected segments
      ↓
Identify upstream/downstream relationships
      ↓
Inspect current congestion
      ↓
Inspect capacity
      ↓
Use forecast state
      ↓
Estimate propagation
```

For each affected segment the engine can calculate:

```text
affected_segment
propagation_direction
estimated_start
estimated_duration
severity
downstream_segments
upstream_segments
```

The implementation should avoid claiming exact propagation timing unless supported by the trained model or simulation assumptions.

---

# 14. OD-Aware Diversion

The platform is not intended to provide individual navigation.

Instead, it estimates how aggregated OD demand could be affected.

`od_demand_profiles.csv` provides:

```text
od_id
origin_node
destination_node
base_demand_vph
purpose
```

### Diversion process

```text
Affected segment
      ↓
Identify impacted OD pairs
      ↓
Generate feasible alternative paths
      ↓
Apply turn restrictions
      ↓
Check route capacity
      ↓
Assign a simulated fraction of demand
      ↓
Recalculate network conditions
      ↓
Compare baseline vs diversion
```

Turn restrictions from `turn_restrictions.csv` must be respected.

---

# 15. Counterfactual Simulation Engine

The simulation engine must preserve the baseline state and operate on a copy.

## 15.1 Supported scenario types

Based on the supplied dataset and scenario structure:

- incident,
- road closure,
- capacity reduction,
- roadwork,
- demand surge,
- OD diversion,
- event,
- alternative signal plan,
- planning candidate.

## 15.2 Simulation sequence

```text
Load baseline
      ↓
Create scenario copy
      ↓
Apply intervention
      ↓
Modify effective capacity/demand/routing
      ↓
Run network-flow approximation
      ↓
Calculate segment results
      ↓
Calculate network metrics
      ↓
Compare against baseline
```

---

# 16. Intervention Modeling

Planning candidates contain:

```text
candidate_id
target_segment
intervention_type
capacity_delta_vph
cost_index
feasibility_band
```

The candidate should be evaluated by applying its supplied capacity change inside the simulation.

The system must label these as **simulated planning interventions**, not completed infrastructure projects.

---

# 17. Before/After Metrics

For each simulation, calculate:

### Network-level

```text
total_delay
average_speed
average_travel_time
total_queue
congested_segments
critical_segments
network_congestion_index
```

### Segment-level

```text
speed_change
flow_change
delay_change
queue_change
congestion_change
```

### Impact summary

```text
baseline
counterfactual
absolute_change
percentage_change
affected_segments
```

---

# 18. Recurring Bottleneck Detection

Recurring bottlenecks are identified using historical traffic data.

Possible indicators:

- congestion frequency,
- congestion severity,
- duration,
- peak-hour recurrence,
- structural bottleneck flag,
- importance,
- persistence after excluding incident/roadwork periods.

A bottleneck should not be classified as purely structural when the observed congestion is explained mainly by temporary incidents or roadworks.

---

# 19. Planning Analysis

The planning module connects recurring bottlenecks to `planning_candidates.csv`.

Example workflow:

```text
Recurring bottleneck
       ↓
Find matching planning candidate
       ↓
Apply candidate capacity_delta_vph
       ↓
Run counterfactual simulation
       ↓
Measure network impact
       ↓
Show cost_index
       ↓
Show feasibility_band
```

The UI should present the candidate, its assumptions, and simulated impact rather than claiming that a project is approved or constructed.

---

# 20. Backend Architecture

Recommended backend:

**FastAPI + Python**

```text
backend/
├── app/
│   ├── api/
│   ├── services/
│   ├── schemas/
│   └── core/
├── ml/
│   ├── preprocessing/
│   ├── features/
│   ├── forecasting/
│   ├── anomaly/
│   └── evaluation/
├── network/
│   ├── graph.py
│   ├── routing.py
│   └── propagation.py
└── simulation/
    ├── engine.py
    ├── scenarios.py
    └── impact.py
```

### API modules

Suggested endpoints:

```text
GET  /health
GET  /network
GET  /traffic/current
GET  /traffic/{segment_id}
GET  /forecast/{segment_id}
GET  /incidents
GET  /incidents/{incident_id}
GET  /propagation/{segment_id}
GET  /bottlenecks
GET  /planning/candidates
POST /simulation/run
POST /simulation/compare
GET  /data-quality
```

---

# 21. Frontend Architecture

Recommended stack:

- Next.js
- TypeScript
- Tailwind CSS
- shadcn/ui
- MapLibre GL JS
- Recharts
- Lucide

```text
frontend/
├── app/
├── components/
├── features/
├── lib/
└── types/
```

---

# 22. Dashboard Design

## 22.1 Overview

Show:

- network map,
- current congestion,
- average speed,
- total delay,
- active incidents,
- forecast risk,
- critical segments.

## 22.2 Forecast

Show:

- +15m,
- +30m,
- +45m,
- +60m,
- segment trend,
- top deteriorating segments.

## 22.3 Incidents

Show:

- incident type,
- severity,
- blocked lanes,
- affected segment,
- propagation,
- supporting evidence.

## 22.4 Simulation

Provide:

- scenario selector,
- target segment,
- incident/capacity configuration,
- diversion settings,
- before/after comparison.

## 22.5 Bottlenecks

Show:

- recurrence,
- severity,
- duration,
- time window,
- structural-bottleneck information.

## 22.6 Planning

Show:

- candidate,
- target segment,
- intervention type,
- simulated impact,
- cost index,
- feasibility band.

## 22.7 Data Quality

Show:

- missing values,
- duplicate records,
- invalid values,
- sensor-quality issues,
- rejected/repaired records,
- target/input separation.

---

# 23. Explainability and Trust

Every result should carry a provenance label.

### Status labels

```text
OBSERVED
FORECAST
SIMULATED
ESTIMATED
ASSUMPTION
TARGET
```

Examples:

- Current speed → `OBSERVED`
- 30-minute congestion → `FORECAST`
- Counterfactual speed → `SIMULATED`
- Affected OD flow → `ESTIMATED`
- Demand diversion percentage → `ASSUMPTION`
- Validation target → `TARGET`

Forecast target values must not appear in normal prediction views.

---

# 24. Database Design

For a production-style implementation, PostgreSQL + PostGIS can store:

```text
segments
nodes
traffic_observations
incidents
roadworks
context
signals
turn_restrictions
od_demand
planning_candidates
scenarios
simulation_runs
simulation_results
forecast_results
bottlenecks
```

For the prototype, CSV/Parquet-backed processing can be used before database migration.

---

# 25. Caching and Performance

The full training dataset contains millions of rows, so the system should avoid repeatedly scanning raw CSVs.

Recommended approach:

```text
Raw CSV
   ↓
Validated/cleaned Parquet
   ↓
Feature store / cached aggregates
   ↓
API
```

Precompute:

- segment metadata,
- graph,
- rolling traffic features,
- historical bottleneck statistics,
- OD lookup tables.

Scenario simulations should operate on in-memory network state where practical.

---

# 26. Testing Strategy

## Unit tests

Test:

- data validation,
- feature generation,
- graph creation,
- turn restrictions,
- routing,
- propagation,
- simulation calculations,
- metric calculations.

## Integration tests

Test:

```text
Dataset → Features → Model → Forecast API
Dataset → Graph → Simulation → Results API
```

## Leakage tests

Explicitly verify:

- future traffic observations are not used,
- forecast target columns are excluded from model features,
- future incidents are not visible before their start time,
- future roadwork information is not incorrectly exposed,
- validation data is not used during training.

---

# 27. Model Development Phases

## Phase 1 — Data foundation

- Validate all datasets.
- Build schemas.
- Clean traffic data.
- Build network graph.
- Create data-quality report.

## Phase 2 — Current-state intelligence

- Build current traffic state.
- Add incident and roadwork joins.
- Build network visualization.

## Phase 3 — Forecasting

- Implement baseline.
- Implement LightGBM/XGBoost.
- Validate four horizons.
- Store forecast results.

## Phase 4 — Propagation

- Build graph traversal.
- Estimate affected segments.
- Connect propagation with forecasts.

## Phase 5 — Simulation

- Implement scenario engine.
- Add capacity changes.
- Add OD-aware diversion.
- Calculate before/after impact.

## Phase 6 — Planning

- Detect recurring bottlenecks.
- Evaluate planning candidates.
- Add planning dashboard.

## Phase 7 — Integration

- Connect frontend and backend.
- Add explainability labels.
- Add data-quality interface.
- Run complete scenario tests.

---

# 28. Recommended Repository

```text
traffic-x/
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

# 29. Key Design Principles

1. **Dataset-first** — implementation must follow the supplied schemas.
2. **No target leakage** — forecast target files are labels only.
3. **Network-aware** — segment-level predictions are interpreted through topology.
4. **Time-aware** — future information must never enter past states.
5. **Simulation-only interventions** — no live infrastructure control.
6. **Evidence-based incident reasoning** — do not infer unsupported incident types.
7. **OD-aware analysis** — diversions operate on aggregated demand, not individual GPS users.
8. **Explainability** — every important output has a source/provenance label.
9. **Before/after comparison** — every counterfactual intervention is evaluated against a baseline.
10. **Validation-driven complexity** — advanced models are added only when they improve validation results.

---

# 30. Final System Flow

```text
             DATASET
                │
                ▼
       DATA QUALITY CHECK
                │
                ▼
       FEATURE ENGINEERING
                │
        ┌───────┴────────┐
        ▼                ▼
   NETWORK GRAPH     ML FEATURES
        │                │
        └───────┬────────┘
                ▼
        CURRENT STATE
                │
      ┌─────────┼─────────┐
      ▼         ▼         ▼
 INCIDENT   FORECAST   ROADWORK
 ANALYSIS   15–60 MIN   ANALYSIS
      │         │         │
      └─────────┼─────────┘
                ▼
        PROPAGATION MODEL
                │
                ▼
        OD-AWARE DIVERSION
                │
                ▼
       COUNTERFACTUAL SIM
                │
        ┌───────┴────────┐
        ▼                ▼
  BOTTLENECKS       PLANNING
        │                │
        └───────┬────────┘
                ▼
       BEFORE / AFTER IMPACT
                │
                ▼
       EXPLAINABLE DASHBOARD
```

---

## 31. Success Criteria

A complete TRAFFIC-X implementation should be able to demonstrate:

- reliable ingestion of the supplied dataset,
- clear data-quality reporting,
- network visualization of all 436 segments,
- current traffic-state reconstruction,
- supported incident identification,
- 15/30/45/60-minute forecasting,
- network propagation analysis,
- OD-aware simulated diversion,
- counterfactual scenario comparison,
- recurring bottleneck identification,
- planning-candidate evaluation,
- transparent assumptions and provenance,
- no target leakage,
- no live infrastructure dependency.

This design keeps the system aligned with the supplied training/validation data and the software-only nature of the challenge.
