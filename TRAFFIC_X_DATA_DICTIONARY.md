# TRAFFIC-X Dataset & Implementation Data Dictionary

## 1. Dataset identity

The supplied dataset is **NeuraX Smart Cities Dataset v2 - Training**.

The manifest specifies:

- 436 road segments
- 120 network nodes
- 15 training days
- 4 validation days
- 8 hidden test days
- 5-minute resolution
- 1,883,520 training traffic observations
- 502,272 validation traffic observations
- 1,004,544 hidden-test observations
- 36 hidden/test scenarios

The dataset explicitly separates forecast targets from traffic observations and expects network-aware reasoning around incidents, propagation, diversions, recurring bottlenecks and counterfactual interventions.

---

## 2. Dataset split

| Split | Period | Traffic rows | Traffic timestamps | Purpose |
|---|---|---:|---:|---|
| Train | 2026-01-01 → 2026-01-15 | 1,883,520 | 4,320 | Model training |
| Validation | 2026-01-16 → 2026-01-19 | 502,272 | 1,152 | Model selection/evaluation |
| Hidden test | 8 days | 1,004,544 | — | Final evaluation; not supplied |

All traffic timestamps are at 5-minute resolution.

---

## 3. Traffic observations

### `traffic_train.csv`
### `traffic_validation.csv`

Columns:

| Column | Meaning |
|---|---|
| `timestamp` | Observation timestamp |
| `segment_id` | Road segment identifier |
| `source_node` | Start node |
| `target_node` | End node |
| `speed_kmh` | Observed speed |
| `flow_vph` | Traffic flow in vehicles/hour |
| `occupancy_pct` | Occupancy percentage |
| `travel_time_min` | Observed travel time |
| `free_flow_time_min` | Free-flow travel time |
| `delay_min` | Delay relative to free flow |
| `queue_length_veh` | Queue-length proxy in vehicles |
| `congestion_index` | Supplied congestion indicator |
| `sensor_quality` | Observation quality indicator |

### Important implementation rule

The traffic observation tables are the main **feature source**.

They contain 436 segments × 5-minute observations.

Do not assume that `sensor_quality = 1` means the value is perfect; use it as a feature/quality signal and investigate missing, duplicated, stuck and noisy observations.

---

## 4. Forecast targets

### `forecast_targets_train.csv`
### `forecast_targets_validation.csv`

Columns:

| Column | Meaning |
|---|---|
| `timestamp` | Forecast origin time |
| `segment_id` | Road segment |
| `target_speed_15m` | Speed 15 minutes ahead |
| `target_flow_15m` | Flow 15 minutes ahead |
| `target_congestion_15m` | Congestion 15 minutes ahead |
| `target_speed_30m` | Speed 30 minutes ahead |
| `target_flow_30m` | Flow 30 minutes ahead |
| `target_congestion_30m` | Congestion 30 minutes ahead |
| `target_speed_45m` | Speed 45 minutes ahead |
| `target_flow_45m` | Flow 45 minutes ahead |
| `target_congestion_45m` | Congestion 45 minutes ahead |
| `target_speed_60m` | Speed 60 minutes ahead |
| `target_flow_60m` | Flow 60 minutes ahead |
| `target_congestion_60m` | Congestion 60 minutes ahead |

### Leakage rule

These files are **labels only**.

Never merge target columns into feature tables.

The forecast training examples have fewer timestamps than the traffic observations because the target horizon requires future observations.

---

## 5. Network

### `network.csv`

Columns:

| Column | Meaning |
|---|---|
| `segment_id` | Road segment |
| `source_node` | Start node |
| `target_node` | End node |
| `road_class` | Road classification |
| `lanes` | Number of lanes |
| `free_flow_speed_kmh` | Free-flow speed |
| `capacity_vph` | Base capacity |
| `length_km` | Segment length |
| `grade_pct` | Grade |
| `signal_id` | Associated signal |
| `structural_bottleneck` | Supplied structural-bottleneck flag |
| `importance` | Network importance score |
| `peak_capacity_factor` | Peak-period capacity multiplier |

### Implementation

Build a directed graph from:

```text
source_node → target_node
```

Each edge should retain all network attributes.

Do not independently invent road capacity when `capacity_vph` is available.

---

## 6. Nodes

### `nodes.csv`

| Column | Meaning |
|---|---|
| `node_id` | Network node |
| `x` | Synthetic/network coordinate |
| `y` | Synthetic/network coordinate |
| `lat` | Latitude |
| `lon` | Longitude |

Use `lat/lon` for map rendering and `x/y` where useful for graph/layout calculations.

---

## 7. Incidents

### `incidents_train.csv`
### `incidents_validation.csv`

| Column | Meaning |
|---|---|
| `incident_id` | Incident identifier |
| `start_time` | Start |
| `end_time` | End |
| `segment_id` | Affected segment |
| `incident_type` | Incident category |
| `severity` | Severity |
| `lanes_blocked` | Number of blocked lanes |

Observed training incident types include:

```text
stalled_vehicle
demand_surge
accident_like
road_closure
lane_blockage
```

Use interval joins against traffic timestamps.

For each traffic observation, derive features such as:

```text
incident_active
incident_severity
lanes_blocked
incident_type
```

Do not leak future incident information. Only use incident information that would be available at the forecast origin.

---

## 8. Context

### `context_train.csv`
### `context_validation.csv`

| Column | Meaning |
|---|---|
| `timestamp` | Context timestamp |
| `temperature_c` | Temperature |
| `rain_intensity` | Rain intensity |
| `event_level` | Event intensity |
| `event_id` | Event identifier |
| `holiday_flag` | Holiday indicator |
| `day_of_week` | Day of week |
| `hour` | Hour/time representation |

Context is timestamp-aligned and can be joined directly to traffic observations.

---

## 9. Roadworks

### `roadworks_train.csv`
### `roadworks_validation.csv`

| Column | Meaning |
|---|---|
| `work_id` | Roadwork identifier |
| `segment_id` | Affected segment |
| `start_time` | Start |
| `end_time` | End |
| `closure_fraction` | Fraction of capacity/lane availability affected |
| `work_type` | Work category |

Training work types include:

```text
lane_maintenance
resurfacing
utility_work
```

Create time-aware features:

```text
roadwork_active
closure_fraction
work_type
```

---

## 10. Signal plans

### `signal_plans.csv`

| Column | Meaning |
|---|---|
| `signal_id` | Signal identifier |
| `node_id` | Controlled node |
| `cycle_s` | Signal cycle length |
| `green_ratio` | Green allocation ratio |
| `offset_s` | Offset |

These are **input data for analysis/simulation**.

They do not imply live signal control.

---

## 11. Turn restrictions

### `turn_restrictions.csv`

| Column | Meaning |
|---|---|
| `node_id` | Junction |
| `from_segment` | Incoming segment |
| `to_segment` | Outgoing segment |
| `restriction` | Turn restriction type |

Observed restrictions include:

```text
no_turn
time_window
```

The routing/simulation engine must respect these constraints.

---

## 12. OD demand

### `od_demand_profiles.csv`

| Column | Meaning |
|---|---|
| `od_id` | OD pair identifier |
| `origin_node` | Origin |
| `destination_node` | Destination |
| `base_demand_vph` | Base demand |
| `purpose` | Demand purpose |

Use OD demand for:

- affected-flow estimation
- route assignment
- diversion scenarios
- network impact calculation

The supplied table is a profile, not a live GPS feed.

---

## 13. Planning candidates

### `planning_candidates.csv`

| Column | Meaning |
|---|---|
| `candidate_id` | Candidate identifier |
| `target_segment` | Target road segment |
| `intervention_type` | Candidate intervention |
| `capacity_delta_vph` | Simulated capacity change |
| `cost_index` | Relative cost index |
| `feasibility_band` | Feasibility category |

The system should evaluate these candidates through simulation.

It must not represent them as approved construction projects.

---

## 14. Scenario examples

### `scenario_examples.csv`

| Column | Meaning |
|---|---|
| `scenario_id` | Scenario identifier |
| `scenario_type` | Scenario category |
| `start_time` | Scenario start |
| `end_time` | Scenario end |
| `target_segment` | Affected segment |
| `incident_type` | Scenario incident type |
| `severity` | Scenario severity |
| `candidate_interventions` | Expected intervention evaluation guidance |

These examples should drive repeatable demonstrations and scenario tests.

---

## 15. Noise and data-quality handling

The manifest explicitly indicates noise including:

- missing values
- duplicates
- spikes
- stuck sensors
- impossible negative readings
- row shuffle

Therefore the ingestion pipeline must include:

```text
Schema validation
↓
Timestamp validation
↓
Duplicate detection
↓
Range validation
↓
Missing-value handling
↓
Sensor-quality checks
↓
Outlier/spike detection
↓
Sort by timestamp + segment_id
```

Never silently delete suspicious observations.

Keep a data-quality report.

---

## 16. Feature engineering

### Temporal

```text
minute_of_day
day_of_week
is_weekend
peak_period
cyclic_hour_sin
cyclic_hour_cos
```

### Traffic history

```text
speed_lag_5m
speed_lag_10m
speed_lag_15m
speed_lag_30m
flow_lag_5m
flow_lag_15m
congestion_lag_5m
rolling_speed_mean
rolling_speed_std
rolling_flow_mean
speed_delta
flow_delta
```

### Network

```text
upstream_mean_speed
downstream_mean_speed
upstream_congestion
downstream_congestion
neighbor_congestion_count
node_degree
segment_importance
structural_bottleneck
```

### Context

```text
rain_intensity
event_level
holiday_flag
temperature_c
```

### Disruptions

```text
incident_active
incident_severity
lanes_blocked
roadwork_active
closure_fraction
```

### Network metadata

```text
capacity_vph
lanes
length_km
grade_pct
peak_capacity_factor
signal_cycle
green_ratio
```

### Demand

```text
origin_demand
destination_demand
affected_od_count
```

---

## 17. Forecast training design

Use a **direct multi-horizon prediction design**.

Targets:

```text
15m
30m
45m
60m
```

Each horizon can predict:

```text
speed
flow
congestion
```

Recommended first model:

```text
LightGBM / XGBoost
```

Train separate models per horizon/target or use a multi-output wrapper.

The first goal is a strong validation baseline, not maximum model complexity.

---

## 18. Validation protocol

Use the provided temporal validation period.

Do not randomly split the traffic rows.

Recommended:

```text
Train:
2026-01-01 → 2026-01-15

Validation:
2026-01-16 → 2026-01-19
```

This preserves temporal ordering.

Evaluate:

```text
MAE
RMSE
WAPE
```

and traffic-state classification metrics where applicable.

Report results separately for:

```text
15m
30m
45m
60m
```

---

## 19. Final challenge architecture

The system should have four intelligence layers:

### Layer A — State

```text
What is happening?
```

### Layer B — Prediction

```text
What will happen?
```

### Layer C — Network reasoning

```text
How will it propagate?
```

### Layer D — Counterfactual decision support

```text
What happens if we change something?
```

This keeps forecasting from becoming the entire project.

---

## 20. Important implementation rule

Do not build a generic chatbot around the data.

The primary product is:

```text
Network visualization
+
Forecasting
+
Incident reasoning
+
Propagation
+
Simulation
+
Planning analysis
```

An AI explanation panel may explain results, but the core intelligence must come from the structured models and simulation engine.
