# Product Requirements Document (PRD)

## TRAFFIC-X
### Network-Aware Urban Traffic Intelligence, Forecasting & Counterfactual Simulation Platform

**Document Version:** 1.0  
**Status:** Implementation Ready  
**Target Environment:** Hyderabad-like dense urban road network  
**System Type:** Software-only AI decision-support and simulation platform

---

# 1. Executive Summary

TRAFFIC-X is a software-only, network-aware urban traffic intelligence platform designed for the challenge environment of a Hyderabad-like urban road network.

The platform consumes organizer-provided traffic, road-network, OD-demand, signal-plan, turn-restriction, incident, context, roadwork, and planning-candidate datasets. It creates a continuously updated simulated view of network conditions, detects congestion and abnormal traffic behavior, reasons about incidents when supported by available data, forecasts traffic states 15–60 minutes ahead, predicts congestion propagation, evaluates diversion strategies, identifies recurring bottlenecks, and simulates potential network/infrastructure interventions.

The system is strictly advisory and simulation-based.

It must not:
- Control real traffic signals.
- Access live cameras.
- Integrate GPS devices.
- Connect to roadside sensors.
- Connect to municipal infrastructure.
- Execute physical construction.
- Issue real-world control commands.

The central product loop is:

**Observe → Diagnose → Predict → Propagate → Simulate → Compare → Recommend**

The system's recommendations must be accompanied by evidence, assumptions, confidence/uncertainty where applicable, and estimated before/after impact.

---

# 2. Problem Statement

Large urban road networks change rapidly. Congestion can emerge from temporary incidents, roadworks, weather, event-driven demand, signal interactions, capacity limitations, turn restrictions, or recurring structural bottlenecks.

A system that only predicts traffic speed is insufficient.

TRAFFIC-X must answer five questions:

1. **What is happening now?**
2. **Why is it happening?**
3. **What is likely to happen in the next 15–60 minutes?**
4. **What simulated/advisory intervention could reduce the impact?**
5. **Which recurring bottlenecks deserve longer-term planning attention?**

The system must reason at the **network level**, rather than treating every road segment as an independent time series.

---

# 3. Product Vision

Build an explainable traffic digital twin that transforms historical and scenario datasets into actionable, measurable, simulation-based traffic intelligence.

### Vision Statement

> Enable traffic planners and decision-makers to understand network conditions, anticipate congestion propagation, test possible responses safely in simulation, and identify recurring network improvements using evidence from data.

---

# 4. Goals

## 4.1 Primary Goals

### G1 — Network State Awareness
Maintain a continuously updated simulated representation of traffic conditions across the supplied road network.

### G2 — Congestion Detection
Identify normal, slow, congested, critical, and abnormal traffic states.

### G3 — Incident Reasoning
Use available incident, traffic, context, and roadwork evidence to detect/classify incidents where the data supports such conclusions.

### G4 — Forecasting
Forecast traffic states for:
- +15 minutes
- +30 minutes
- +45 minutes
- +60 minutes

### G5 — Propagation Prediction
Estimate how congestion on one road segment or junction may affect connected segments.

### G6 — Operational Decision Support
Generate simulated diversion/traffic-management scenarios and estimate their network-level effects.

### G7 — Recurring Bottleneck Detection
Identify locations and time periods with persistent/recurrent congestion after accounting for temporary disruptions where possible.

### G8 — Long-Term Planning Analysis
Evaluate organizer-provided planning candidates through counterfactual simulation and provide estimated before/after impact.

### G9 — Explainability
Every major recommendation must show:
- Evidence
- Assumptions
- Affected network
- Expected impact
- Uncertainty/confidence where applicable

### G10 — Data Leakage Prevention
Forecast target files must never be used as model input features.

---

# 5. Non-Goals and Restrictions

TRAFFIC-X explicitly does NOT implement the following:

| Restricted Capability | Status |
|---|---|
| Live traffic signal control | Prohibited |
| Camera/CCTV access | Prohibited |
| GPS-device integration | Prohibited |
| Roadside sensor integration | Prohibited |
| Municipal infrastructure integration | Prohibited |
| Physical construction | Prohibited |
| Real-world automated traffic control | Prohibited |
| Consumer navigation application | Not the product goal |
| Generic chatbot | Not the product goal |

All actions are:
- Simulated
- Advisory
- Dataset-driven
- Reproducible
- Non-operational

---

# 6. Dataset Scope

Expected files include:

```text
traffic_train.csv
forecast_targets_train.csv
traffic_validation.csv
forecast_targets_validation.csv

incidents_*.csv
context_*.csv
roadworks_*.csv

network.csv
nodes.csv
signal_plans.csv
turn_restrictions.csv

planning_candidates.csv
od_demand_profiles.csv
scenario_examples.csv
```

## 6.1 Forecast Target Isolation

The following files are labels/ground truth and must NOT enter the feature-generation pipeline:

```text
forecast_targets_train.csv
forecast_targets_validation.csv
```

Training flow:

```text
traffic_train
       +
network/context/incident/etc.
       ↓
Feature Engineering
       ↓
Forecast Model
       ↓
Prediction
       ↓
Compare with forecast_targets_train
```

Validation flow:

```text
traffic_validation
       +
network/context/incident/etc.
       ↓
Feature Engineering
       ↓
Forecast Model
       ↓
Predictions
       ↓
Compare with forecast_targets_validation
```

The target files must remain physically and logically separated from feature tables.

---

# 7. Target Users

## 7.1 Primary User

### Traffic/Network Analyst
Needs to:
- Understand current congestion.
- Identify abnormal conditions.
- Investigate causes.
- Forecast future conditions.
- Test diversion strategies.
- Identify recurring bottlenecks.

## 7.2 Secondary User

### Urban Planner / Infrastructure Analyst
Needs to:
- Find recurring bottlenecks.
- Compare planning candidates.
- Estimate potential network-level impact.
- Review evidence supporting an intervention.

## 7.3 Demo/Judging User

### Competition Judge
Needs to quickly understand:
- What is happening.
- What will happen next.
- What the AI recommends.
- Why it recommends it.
- What happens under simulation.
- How the proposed intervention changes measurable outcomes.

---

# 8. Product Principles

1. **Network before isolated roads**
2. **Evidence before recommendations**
3. **Simulation before intervention**
4. **Prediction is not the same as causation**
5. **Uncertainty must be visible**
6. **No target leakage**
7. **No live infrastructure dependency**
8. **Every scenario must be reproducible**
9. **Recommendations must be measurable**
10. **Human decision-makers remain in control**

---

# 9. High-Level System Architecture

```text
                    ORGANIZER DATASETS
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        Traffic Data   Network Data   Context Data
             |             |             |
             +-------------+-------------+
                           |
                           v
                 DATA PROCESSING LAYER
                           |
                           v
                 NETWORK DIGITAL TWIN
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
 Traffic State       Incident/Anomaly   Historical Analysis
 Engine              Reasoning          Engine
        |                  |                  |
        +------------------+------------------+
                           |
                           v
                 FORECASTING ENGINE
                           |
                           v
              PROPAGATION / NETWORK IMPACT
                           |
              +------------+-------------+
              |                          |
              v                          v
       DIVERSION ENGINE          PLANNING ENGINE
              |                          |
              +------------+-------------+
                           |
                           v
                  COUNTERFACTUAL
                    SIMULATION
                           |
                           v
                  IMPACT ANALYSIS
                           |
                           v
                 EXPLAINABILITY LAYER
                           |
                           v
                    WEB DASHBOARD
```

---

# 10. Functional Requirements

## FR-01 — Dataset Ingestion

The system shall:
- Load all supported CSV datasets.
- Validate required columns.
- Validate timestamps.
- Validate identifiers.
- Detect missing values.
- Detect duplicate records.
- Detect inconsistent road/node references.
- Generate a data-quality report.

### Acceptance Criteria

- Invalid records are reported rather than silently discarded.
- Dataset loading errors are visible.
- Each dataset receives a schema/data-quality status.

---

# 11. FR-02 — Network Digital Twin

The system shall construct a directed road graph.

### Node representation

Nodes may represent:
- Intersections
- Junctions
- Network connection points

### Edge representation

Each road segment may contain:
- Edge ID
- Start node
- End node
- Direction
- Length
- Capacity
- Free-flow speed
- Road class
- Current traffic state
- Turn restrictions

### Acceptance Criteria

- Every valid road segment can be mapped to the network graph.
- Invalid references are flagged.
- Turn restrictions can be applied during routing/simulation.
- The network can be visualized.

---

# 12. FR-03 — Traffic State Estimation

For each road segment, calculate a traffic state.

Possible states:

```text
NORMAL
SLOW
CONGESTED
HEAVY
CRITICAL
ABNORMAL
UNKNOWN
```

Candidate features:

- Current speed
- Traffic volume
- Capacity utilization
- Speed reduction from free-flow
- Travel time
- Historical baseline
- Neighboring segment state
- Queue proxy
- Context indicators

Example congestion score:

```text
congestion_score =
1 - current_speed / free_flow_speed
```

The exact thresholds must be configurable and validated against the dataset.

### Acceptance Criteria

- Each valid road segment receives a state where sufficient data exists.
- State classification is visible on the map.
- Missing/insufficient data produces UNKNOWN rather than a fabricated state.

---

# 13. FR-04 — Anomaly Detection

The system shall identify traffic behavior that deviates significantly from expected patterns.

Possible methods:
- Rolling z-score
- Historical baseline deviation
- Isolation Forest
- Statistical change-point detection
- ML anomaly scoring

Examples:

```text
Expected speed: 35 km/h
Observed speed: 14 km/h
Deviation: high
```

The system should consider:
- Time-of-day baseline
- Day-of-week baseline
- Historical traffic patterns
- Neighboring roads
- Context
- Roadworks
- Known incidents

### Output

```text
Anomaly score
Severity
Timestamp
Location
Evidence
Potential explanations
```

---

# 14. FR-05 — Incident Reasoning

The system shall combine multiple evidence sources to identify possible incidents.

Evidence may include:
- Sudden speed drop
- Sudden volume change
- Queue growth
- Known incident record
- Roadwork record
- Context/event/weather record
- Neighboring segment behavior

Incident categories may include:

```text
ACCIDENT
ROADWORK
EVENT_SURGE
WEATHER_RELATED
DEMAND_SURGE
UNKNOWN_ABNORMAL_EVENT
```

The system must not infer a specific incident type when the data does not support it.

### Output Example

```text
Incident: Possible abnormal event
Location: J14-J15
Confidence: 0.84

Evidence:
- 58% speed reduction
- 37% queue growth
- No active roadwork
- Incident record nearby
```

---

# 15. FR-06 — Traffic Forecasting

The system shall forecast future traffic states.

### Horizons

```text
15 minutes
30 minutes
45 minutes
60 minutes
```

### Recommended development strategy

#### Baseline
- Historical mean
- Previous interval
- Seasonal/time-of-day baseline

#### ML model
- XGBoost or LightGBM

#### Advanced optional model
- Temporal GNN
- Spatio-temporal GNN
- LSTM/GRU

The project should not depend on a complex deep-learning model for the MVP.

### Features

Possible features:

```text
Current speed
Current volume
Historical speed
Historical volume
Capacity
Road class
Time-of-day
Day-of-week
Neighboring road states
OD demand
Signal-plan metadata
Turn restrictions
Roadworks
Incidents
Context
```

### Prohibited feature

Future values from:

```text
forecast_targets_train.csv
forecast_targets_validation.csv
```

must never be used as model inputs.

---

# 16. FR-07 — Forecast Evaluation

Evaluate forecasts using appropriate metrics.

Recommended:

- MAE
- RMSE
- MAPE where safe
- WAPE
- R² where meaningful

Also evaluate classification performance for predicted traffic states:

- Accuracy
- Precision
- Recall
- F1

Evaluation should be reported by:
- Forecast horizon
- Road segment
- Traffic condition
- Peak/off-peak period

---

# 17. FR-08 — Congestion Propagation

The system shall model how congestion can spread through connected road segments.

Inputs:
- Road graph
- Current traffic state
- Forecast state
- Capacity
- Connectivity
- Historical propagation patterns
- OD demand

Example:

```text
J14 Critical
    |
    +----> J15 predicted Heavy
    |
    +----> J16 predicted Congested
```

Output:

```text
Origin:
J14

Likely affected:
J15
J16

Estimated propagation window:
15–45 minutes

Risk:
High
```

This is a network-impact prediction, not a guarantee of real-world propagation.

---

# 18. FR-09 — OD Demand Analysis

Use:

```text
od_demand_profiles.csv
```

to understand:
- Origin
- Destination
- Demand volume
- Time-dependent demand

OD demand shall be used to estimate which flows are affected by congestion and which flows could potentially be redirected in simulation.

---

# 19. FR-10 — Diversion Recommendation

The system shall identify feasible alternative paths using:
- Road network
- Current traffic
- Forecast traffic
- Capacity
- OD demand
- Turn restrictions

It shall NOT simply choose the shortest path.

### Diversion workflow

```text
Identify congested corridor
        ↓
Identify affected OD flows
        ↓
Find feasible alternatives
        ↓
Check capacity
        ↓
Apply simulated diversion
        ↓
Recalculate network conditions
        ↓
Compare against baseline
```

### Output

```text
Scenario:
20% of affected OD demand diverted

Estimated impact:
Travel time: -14%
Queue length: -22%
Critical segments: 7 → 5
Alternative corridor overload: No
```

All values must be explicitly labeled as simulated estimates.

---

# 20. FR-11 — Counterfactual Scenario Engine

The system shall allow users to modify scenario parameters without changing the original dataset.

Supported scenario types:

- Incident
- Road closure
- Partial capacity reduction
- Roadwork
- Demand increase
- Demand decrease
- OD diversion
- Alternative signal-plan scenario
- Planning candidate
- Event surge
- Context/weather scenario

### Important

Signal-plan scenarios are simulations only. The system does not control signals.

---

# 21. FR-12 — Scenario Isolation

Every simulation must:
- Copy the baseline network state.
- Apply scenario changes only to the simulation copy.
- Preserve original data.
- Record scenario parameters.
- Produce a reproducible result.

Example:

```text
Scenario ID: SCN-042
Base dataset: validation
Affected edge: R145
Capacity reduction: 40%
Demand diversion: 20%
Timestamp: 09:00
```

---

# 22. FR-13 — Before/After Impact Analysis

For each scenario calculate:

### Network metrics

- Average speed
- Average travel time
- Total travel time
- Network throughput
- Number of congested segments
- Number of critical segments
- Queue proxy
- Spillback risk

### OD metrics

- OD travel time
- Affected demand
- Unserved/overloaded demand where applicable

### Local metrics

- Junction delay
- Road segment delay
- Queue proxy
- Capacity utilization

---

# 23. FR-14 — Recurring Bottleneck Detection

The system shall identify locations where congestion repeatedly occurs.

Candidate indicators:

```text
Congestion frequency
Average severity
Duration
Peak-period recurrence
Historical persistence
Capacity utilization
Incident-adjusted recurrence
Roadwork-adjusted recurrence
```

Example:

```text
Junction J14

Occurrences: 18/month
Typical window: 08:00–09:30
Average severity: High
Known incidents: Low
Roadwork contribution: Low

Classification:
Recurring bottleneck candidate
```

---

# 24. FR-15 — Planning Candidate Evaluation

Use:

```text
planning_candidates.csv
```

as the source of candidate long-term interventions.

For each candidate:
1. Identify affected network area.
2. Apply the candidate in simulation.
3. Recalculate traffic impact.
4. Compare against baseline.
5. Report assumptions.
6. Report estimated impact.

Possible candidate types:
- Turning lane
- Intersection modification
- Capacity expansion
- Grade separation
- Network connection
- Other organizer-provided planning candidates

No physical construction is performed.

---

# 25. FR-16 — Signal Plan Analysis

Use:

```text
signal_plans.csv
```

to understand current network conditions and optionally simulate alternative plans.

The system must never:
- Connect to a traffic controller.
- Send signal commands.
- Claim that a signal was actually changed.

Correct output:

> Simulated alternative signal allocation estimates a reduction in predicted junction delay.

Incorrect output:

> The system changes the traffic signal to reduce congestion.

---

# 26. FR-17 — Explainability

Every recommendation must have an explanation panel.

Example:

```text
RECOMMENDATION

Divert 20% of affected OD demand.

WHY?

• Current corridor speed is 42% below baseline.
• Capacity utilization exceeds threshold.
• Downstream congestion is predicted within 15 minutes.
• Alternative corridor has spare simulated capacity.

EXPECTED IMPACT

Travel time: -11%
Queue proxy: -19%
Critical segments: 6 → 4

ASSUMPTIONS

• Demand remains within scenario range.
• Turn restrictions remain unchanged.
• No new incident occurs.
```

---

# 27. FR-18 — Confidence and Uncertainty

The system should distinguish:

- Observed fact
- Model prediction
- Simulation result
- Recommendation
- Assumption

Example labels:

```text
OBSERVED
PREDICTED
SIMULATED
ESTIMATED
ASSUMED
```

Where possible, show prediction intervals or confidence bands.

---

# 28. Dashboard Requirements

## 28.1 Overview Dashboard

Display:
- Network map
- Current traffic state
- Congested segments
- Critical segments
- Active incidents
- Forecast alerts
- Network KPIs

---

## 28.2 Forecast Dashboard

Display:

```text
Road | Now | +15 | +30 | +45 | +60
```

with:
- Traffic state
- Speed
- Volume
- Forecast uncertainty

---

## 28.3 Incident Dashboard

Display:
- Incident location
- Type
- Severity
- Evidence
- Confidence
- Affected roads
- Propagation forecast

---

## 28.4 Simulation Dashboard

Controls:

```text
Scenario
Affected segment
Capacity change
Demand change
OD diversion
Time window
```

Buttons:

```text
RUN SIMULATION
RESET
COMPARE
```

---

## 28.5 Planning Dashboard

Display:
- Recurring bottlenecks
- Candidate interventions
- Baseline metrics
- Scenario metrics
- Estimated impact
- Assumptions

---

# 29. Map Requirements

Use an interactive network map.

Road colors:

```text
Green  = Normal
Yellow = Slow
Orange = Heavy
Red    = Critical
Purple = Abnormal
Gray   = Unknown
```

Map interactions:
- Click road
- Show current state
- Show forecast
- Show historical profile
- Show neighboring roads
- Show incidents
- Launch simulation

---

# 30. Recommended Technology Stack

## Frontend

```text
React
TypeScript
Tailwind CSS
MapLibre GL JS or Leaflet
Recharts / Plotly
```

## Backend

```text
Python
FastAPI
Pandas
NumPy
NetworkX
Scikit-learn
XGBoost / LightGBM
```

## Advanced ML

Optional:

```text
PyTorch
PyTorch Geometric
```

## Database

```text
PostgreSQL
PostGIS (if geographic coordinates are available)
```

## Deployment

Possible:

```text
Frontend → Vercel/Render
Backend  → Render
Database → PostgreSQL
ML       → Backend service
```

For the competition demo, local deployment is acceptable if required.

---

# 31. Backend API Requirements

Suggested endpoints:

```text
GET  /api/network
GET  /api/traffic/current
GET  /api/traffic/{edge_id}
GET  /api/forecast/{edge_id}
GET  /api/incidents
GET  /api/bottlenecks
GET  /api/planning-candidates

POST /api/simulations
POST /api/simulations/{id}/run
GET  /api/simulations/{id}
GET  /api/simulations/{id}/impact

GET  /api/dashboard/summary
GET  /api/data-quality
```

---

# 32. Suggested Data Model

## RoadSegment

```text
edge_id
source_node
target_node
length
capacity
free_flow_speed
road_class
direction
```

## TrafficObservation

```text
timestamp
edge_id
volume
speed
travel_time
occupancy_if_available
```

## Incident

```text
incident_id
timestamp
location
type
severity
status
```

## Forecast

```text
timestamp
edge_id
horizon
predicted_speed
predicted_volume
traffic_state
lower_bound
upper_bound
```

## Scenario

```text
scenario_id
scenario_type
created_at
parameters
baseline_reference
result
```

## PlanningCandidate

```text
candidate_id
candidate_type
location
affected_edges
parameters
```

---

# 33. ML Feature Engineering

## Temporal features

```text
hour
minute_bucket
day_of_week
weekend
peak_period
holiday/event indicator
```

## Traffic features

```text
current_speed
current_volume
lag_speed_1
lag_speed_2
lag_volume_1
lag_volume_2
rolling_speed_mean
rolling_speed_std
speed_change
volume_change
```

## Network features

```text
upstream congestion
downstream congestion
neighbor mean speed
neighbor congestion count
node degree
edge centrality
path importance
```

## Context features

```text
roadwork active
incident active
weather/context indicators
event indicator
```

## Demand features

```text
OD demand
origin demand
destination demand
demand change
```

---

# 34. Forecasting Model Strategy

## Phase 1 — Baseline

Implement:

```text
Historical average
Last-value baseline
Seasonal baseline
```

## Phase 2 — ML

Train:

```text
XGBoost / LightGBM
```

for each forecast horizon or using horizon as a feature.

## Phase 3 — Network-aware model

If time and data quality permit:

```text
Spatio-temporal GNN
```

Compare against the baseline.

The advanced model should only be adopted if it provides measurable improvement.

---

# 35. Simulation Engine Strategy

The simulation engine should initially use a simplified network-flow model.

For each road segment:

```text
demand
capacity
speed
travel time
congestion state
```

A simplified relationship can estimate congestion as demand approaches/exceeds capacity.

More advanced simulation can incorporate:
- Flow conservation
- Capacity constraints
- Travel-time functions
- OD assignment
- Spillback approximation

The objective is not microscopic vehicle simulation.

The objective is **fast network-level scenario comparison**.

---

# 36. Recommended MVP

The first working version should contain only:

### Phase 1

```text
CSV ingestion
↓
Network graph
↓
Traffic visualization
↓
Congestion detection
↓
Basic forecasting
↓
Dashboard
```

### Phase 2

```text
Incident reasoning
↓
Propagation
↓
Diversion simulation
```

### Phase 3

```text
Recurring bottlenecks
↓
Planning candidates
↓
Counterfactual comparison
↓
Explainability
```

This avoids spending the entire development period on an overly complex AI model.

---

# 37. Demo Scenario

The primary demonstration should follow a complete story.

## Step 1 — Normal network

Show:

```text
Average speed: 34 km/h
Critical roads: 1
```

## Step 2 — Introduce a simulated disruption

Example:

```text
Road R14 capacity reduced by 40%
```

## Step 3 — Detect

System reports:

```text
Abnormal congestion detected
```

## Step 4 — Forecast

```text
+15 min → R14 critical
+30 min → R15 congested
+45 min → R16 congested
```

## Step 5 — Propagation

Animate:

```text
R14 → R15 → R16
```

## Step 6 — Diversion simulation

Try:

```text
10%
20%
30%
```

diversion of affected OD demand.

## Step 7 — Compare

Show:

```text
20% diversion

Travel time:
31.2 → 26.4 min

Critical segments:
8 → 5

Queue proxy:
4.2 → 2.9 km
```

## Step 8 — Long-term analysis

Show that the same area repeatedly becomes congested in historical data.

Then evaluate relevant planning candidates.

---

# 38. Success Metrics

## Technical

### Forecasting

Target metrics should be established after inspecting the dataset.

Measure:
- MAE
- RMSE
- WAPE/MAPE where appropriate
- F1 for traffic-state classification

### Incident detection

Measure:
- Precision
- Recall
- F1

### Anomaly detection

Measure:
- Precision at top-K alerts
- Detection delay where labels exist

### Propagation

Measure:
- Correctly identified affected segments
- Time-to-propagation error

---

# 39. Product-Level Success Metrics

The system should demonstrate:

1. A clear current network state.
2. Accurate future traffic predictions.
3. Meaningful propagation prediction.
4. Feasible diversion scenarios.
5. Quantifiable before/after impact.
6. Detection of recurring bottlenecks.
7. Planning-candidate comparison.
8. Explainable recommendations.
9. Zero use of prohibited live infrastructure.
10. Zero forecast-target leakage.

---

# 40. Non-Functional Requirements

## Performance

Dashboard:
- Initial load target: <5 seconds on competition hardware/network.
- Typical API response: <2 seconds.
- Scenario simulation target: <10 seconds for normal demo-sized scenarios.

## Reliability

- Invalid data must not crash the entire system.
- Simulation failures must return actionable errors.
- Baseline data must never be modified by a simulation.

## Reproducibility

Every scenario should store:
- Input parameters
- Dataset/version
- Model version
- Timestamp
- Results

## Security

- No credentials embedded in source code.
- No unauthorized external infrastructure connections.
- No live device tracking.
- Dataset uploads must be validated.

---

# 41. UI/UX Requirements

The interface should prioritize:

### At-a-glance understanding

Within 10 seconds, a judge should understand:
- Where congestion is.
- What is getting worse.
- What will happen next.
- What the system recommends.

### Evidence-first recommendations

Never show only:

> "Divert traffic."

Instead:

```text
Recommendation
+
Evidence
+
Expected impact
+
Assumptions
```

### Simulation clearly labeled

Use badges:

```text
OBSERVED
FORECAST
SIMULATED
ESTIMATED
```

This prevents users from confusing simulation outputs with real-world measurements.

---

# 42. Error Handling

Examples:

### Missing traffic observation

```text
Traffic data unavailable.
State: UNKNOWN
```

### Missing network mapping

```text
Edge E123 cannot be mapped to the road network.
```

### Insufficient history

```text
Forecast unavailable:
insufficient historical observations.
```

### Simulation infeasible

```text
Scenario rejected:
alternative corridor exceeds configured capacity.
```

---

# 43. Data Quality Dashboard

Include a small admin/data-quality view.

Display:

```text
Dataset                    Status

traffic_train.csv          ✓
network.csv                ✓
nodes.csv                  ✓
incidents.csv              ✓
roadworks.csv              ✓
signal_plans.csv           ✓
turn_restrictions.csv      ⚠
planning_candidates.csv    ✓
```

Show:
- Missing values
- Duplicate records
- Unknown IDs
- Timestamp gaps
- Invalid references

---

# 44. Model Explainability

For tree models such as XGBoost/LightGBM, expose:
- Feature importance
- SHAP explanations where practical

Example:

```text
Why is J14 predicted to become critical?

Current volume          ██████████
Upstream congestion     ████████
Historical peak demand  ██████
Roadwork                ████
Time-of-day             ███
```

This is supplementary evidence, not causal proof.

---

# 45. Architecture for Implementation

Recommended repository:

```text
traffic-x/
│
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── maps/
│   │   ├── charts/
│   │   └── api/
│
├── backend/
│   ├── app/
│   │   ├── api/
│   │   ├── models/
│   │   ├── services/
│   │   ├── simulation/
│   │   ├── forecasting/
│   │   ├── detection/
│   │   └── network/
│
├── ml/
│   ├── preprocessing/
│   ├── features/
│   ├── training/
│   ├── evaluation/
│   └── models/
│
├── data/
│   ├── raw/
│   ├── processed/
│   └── schemas/
│
├── notebooks/
│
├── tests/
│
├── docs/
│
└── README.md
```

---

# 46. Development Roadmap

## Sprint 1 — Dataset Understanding

- Inspect every CSV.
- Infer schemas.
- Identify primary/foreign keys.
- Analyze timestamps.
- Build data-quality report.
- Verify forecast target isolation.

**Deliverable:** Dataset dictionary + clean data pipeline.

---

## Sprint 2 — Network Foundation

- Build road graph.
- Map traffic observations to edges.
- Implement turn restrictions.
- Visualize network.

**Deliverable:** Interactive traffic network.

---

## Sprint 3 — Traffic Intelligence

- Current traffic state.
- Congestion score.
- Anomaly detection.
- Historical baseline.

**Deliverable:** Current-state intelligence.

---

## Sprint 4 — Forecasting

- Historical baseline.
- XGBoost/LightGBM.
- Validation pipeline.
- 15/30/45/60-minute forecasts.

**Deliverable:** Forecast engine.

---

## Sprint 5 — Incident + Propagation

- Incident correlation.
- Roadwork correlation.
- Context integration.
- Network propagation.

**Deliverable:** Incident/propagation engine.

---

## Sprint 6 — Simulation

- Scenario model.
- Capacity modifications.
- OD demand changes.
- Diversion engine.
- Before/after comparison.

**Deliverable:** Counterfactual simulator.

---

## Sprint 7 — Planning

- Recurring bottleneck detection.
- Planning candidate integration.
- Candidate simulations.
- Impact comparison.

**Deliverable:** Planning analysis.

---

## Sprint 8 — Dashboard + Demo

- Final UI.
- Explainability.
- Scenario playback.
- Demo dataset.
- Performance optimization.
- Testing.

**Deliverable:** Competition-ready product.

---

# 47. Priority Matrix

| Feature | Priority |
|---|---|
| Dataset ingestion | P0 |
| Data validation | P0 |
| Network graph | P0 |
| Traffic state | P0 |
| Forecasting | P0 |
| Dashboard | P0 |
| Target leakage prevention | P0 |
| Incident reasoning | P1 |
| Propagation | P1 |
| Diversion simulation | P1 |
| Counterfactual engine | P1 |
| Bottleneck detection | P1 |
| Planning candidates | P1 |
| Explainability | P1 |
| Advanced GNN | P2 |
| Advanced uncertainty | P2 |
| Highly detailed microscopic simulation | P3 |

---

# 48. Key Risks and Mitigations

## Risk 1 — Forecasting is accurate but system feels generic

**Mitigation:** Make network propagation and counterfactual simulation central to the demo.

## Risk 2 — Data leakage

**Mitigation:** Physically isolate forecast target files from feature generation.

## Risk 3 — Recommendations are unrealistic

**Mitigation:** Enforce network topology, capacity, OD demand, and turn restrictions during simulation.

## Risk 4 — AI claims unsupported causes

**Mitigation:** Distinguish correlation/evidence from causal certainty.

## Risk 5 — Simulation produces unrealistic improvements

**Mitigation:** Apply capacity constraints and compare against baseline.

## Risk 6 — Overengineering with GNN

**Mitigation:** Establish a strong classical ML baseline before attempting GNN.

## Risk 7 — Judges confuse simulation with real traffic control

**Mitigation:** Clearly label all outputs as observed, predicted, or simulated.

---

# 49. Final Product Flow

The complete product should behave like this:

```text
                    TRAFFIC DATA
                         |
                         v
                CURRENT NETWORK STATE
                         |
              +----------+----------+
              |                     |
              v                     v
       ANOMALY/INCIDENT        HISTORICAL
          REASONING             PATTERNS
              |                     |
              +----------+----------+
                         |
                         v
                    FORECAST
                 +15/+30/+45/+60
                         |
                         v
                PROPAGATION MODEL
                         |
                         v
              AFFECTED NETWORK AREA
                         |
              +----------+----------+
              |                     |
              v                     v
        DIVERSION SCENARIO    PLANNING SCENARIO
              |                     |
              +----------+----------+
                         |
                         v
                COUNTERFACTUAL
                  SIMULATION
                         |
                         v
                 BEFORE vs AFTER
                         |
                         v
               EVIDENCE + IMPACT
                         |
                         v
                 DECISION SUPPORT
```

---

# 50. Final Product Definition

TRAFFIC-X is **not**:

- Google Maps
- A navigation app
- A live signal controller
- A CCTV analytics system
- A GPS tracking platform
- A roadside IoT system
- A generic traffic chatbot

TRAFFIC-X is:

> **A software-only, network-aware AI decision-support and simulation platform that uses organizer-provided datasets to understand urban traffic conditions, detect abnormal behavior, reason about incidents, forecast traffic 15–60 minutes ahead, predict congestion propagation, evaluate simulated diversion strategies, identify recurring bottlenecks, and quantify the potential impact of network/planning interventions.**

### Core differentiator

**Prediction + Network Reasoning + Counterfactual Simulation + Evidence**

The final system should allow a judge to select a traffic problem and go from:

**"What is happening?"**

→ **"What will happen?"**

→ **"Why?"**

→ **"What if we do this?"**

→ **"What is the estimated impact?"**

without requiring access to any live municipal or physical infrastructure.

## Evaluation, Robustness & Judge Alignment

TRAFFIC-X is validated against the judging criteria through explicit measurable experiments.

### Detection

Congestion and supported incident detection use Precision, Recall, F1-score and false-alarm measures. Results are segmented by relevant traffic conditions where sufficient data exists.

### Forecasting

Forecast accuracy is measured for +15, +30, +45 and +60 minute speed, flow and congestion targets using MAE, RMSE and WAPE where appropriate. Evaluation uses the supplied temporal training/validation split and excludes forecast target files from feature engineering.

### Recommendation quality

Every recommendation is evaluated through a baseline-versus-counterfactual simulation. Operational value is measured through delay, speed, travel time, queue, congestion and network-spillover changes, while feasibility constraints, assumptions and limitations are displayed.

### Robustness

The evaluation suite deliberately perturbs data and demand to test missing observations, sensor noise, duplicates/spikes, low-quality observations, changed OD demand, and unseen temporal/scenario conditions. Performance degradation is reported explicitly.

### Explainability and confidence

Each result carries a provenance label: `OBSERVED`, `FORECAST`, `SIMULATED`, `ESTIMATED`, `ASSUMPTION`, or `TARGET`. Forecasts and recommendations expose uncertainty, evidence, assumptions and limitations. Confidence must be derived from measurable model/evidence conditions and must not be an arbitrary score.

### Reproducibility

Experiments record dataset version, model version, feature configuration, random seed, training/validation period, hyperparameters, dependency versions and evaluation metrics.

### Simulation-only constraint

All actions, diversion plans, traffic-management responses, construction/network suggestions and signal-plan changes remain **simulated or advisory only**. TRAFFIC-X does not directly control or modify real-world traffic infrastructure, vehicles, roads, signals or municipal systems.

## Judge-Aligned Requirements Addendum

### Explainability and confidence

Every alert, forecast and advisory must expose evidence, uncertainty/confidence, assumptions and limitations. Outputs carry provenance labels: `OBSERVED`, `FORECAST`, `SIMULATED`, `ESTIMATED`, `ASSUMPTION`, or `TARGET`.

### Robustness

The preprocessing pipeline explicitly handles missing values, duplicates, impossible readings, spikes, stuck sensors and low-quality observations. A robustness harness performs controlled data dropout, sensor-noise injection, demand shifts and unseen temporal/scenario tests and reports performance degradation.

### Evaluation

A dedicated evaluation module performs time-based validation and tracks:

- congestion/incident Precision, Recall and F1,
- false-alarm rate,
- forecast MAE/RMSE/WAPE,
- forecast performance at 15/30/45/60 minutes,
- counterfactual recommendation impact.

### Continuous update

The system supports chronological replay of the supplied 5-minute dataset. Each timestamp updates the network state, congestion/incident analysis, forecast, propagation estimates and dashboard without requiring live infrastructure integration.

### Named intelligence methods

The baseline implementation uses LightGBM for forecasting, Isolation Forest/anomaly scoring for traffic anomalies, NetworkX for network reasoning, capacity-constrained routing with OD demand for diversion, custom counterfactual network-flow simulation, and historical frequency/severity/duration analysis for recurring bottlenecks.

### Parallel architecture

The historical recurring-bottleneck branch is independent of the current forecast branch and feeds planning/intervention simulation. Both branches feed the explainability/confidence layer and dashboard.

### Simulation-only constraint

All actions, diversion plans, construction/network suggestions, capacity changes and signal-plan changes remain simulated or advisory only. No real-world traffic infrastructure is controlled or modified.
