# AirSentinel

Fault detection for air handling units (AHUs). AirSentinel reads every unit in a fleet of
buildings, names the fault, points at the part that has failed, and gives the technician the
repair steps.

**Live:** https://airsentinel.twinstackstudio.com

![AirSentinel command centre](./assets/airsentinel-dashboard.png)

## What it does

- **Diagnosis** — a gradient-boosted model (LightGBM) names one of 12 faults from a unit's
  readings: stuck dampers, passing or stuck valves, fouled coil, clogged filter, slipping
  fan belt, duct leakage, drifted sensors. An isolation forest flags behaviour that is
  unusual without a name.
- **Where and how to fix it** — each fault maps to a part on an animated schematic of the
  unit, with symptoms, likely causes, repair steps, tools and time (`backend/kb.py`).
- **Command centre** — fleet health, the fleet drawn as a city (one floor per unit), a live
  counter of the money open faults are burning, early warnings, and a time machine that
  replays the last seven days.
- **Ask Sentinel** — questions in plain words ("what is wrong with NH 8?", "how much are we
  wasting?"). Figures come straight from the fleet data; only open questions go to a
  language model.
- **Fault Lab** — break a unit on purpose and watch the model find it.
- **Work orders** — a printable job card for any open fault.
- Command palette (Ctrl+K), spoken briefing, alert sounds, the unit's fan as sound.

## The data, honestly

There is no public dataset of a city's AHUs with their faults labelled, so the fleet is
**simulated**: 8 buildings, 64 units, one year of readings every 15 minutes (2.24 million
rows) with 928 fault episodes. Each unit is a physical model (mixing box, filter, coils,
fan, duct, zones) under real control sequences; a fault is injected where it physically
happens and the controls react to it.

Scores are measured only on units the model never saw (4-fold, grouped by unit):

| Measure | Result |
| --- | --- |
| Single readings named correctly | 97.8% |
| 3-hour diagnoses named correctly | 97.5% |
| Fault episodes named correctly | 96.4% |
| False alarms on healthy readings | 0.41% |
| Anomaly detector AUC | 0.951 |

These numbers describe the simulated fleet. A real building would need its own history
before they could be trusted.

## Architecture

```mermaid
flowchart LR
  SIM[Physical AHU simulator<br/>8 buildings · 64 units · 1 year] --> F[Feature builder]
  F --> LGB[LightGBM<br/>12 named faults]
  F --> IF[Isolation Forest<br/>unnamed anomalies]
  LGB --> API[Flask API]
  IF --> API
  KB[Fault knowledge base<br/>part · causes · repair steps] --> API
  API --> CP[Ask Sentinel copilot<br/>rules first, language model for open questions]
  API --> UI[React 19 command centre<br/>live twin · Fault Lab · work orders]
```

## Source code

This is the public showcase. The source code is kept in a private repository. Try the live demo above, and [contact TwinStack Studio](https://twinstackstudio.com/contact) for a walkthrough or a similar project.

© 2026 TwinStack Studio. All rights reserved. See [LICENSE](./LICENSE).
