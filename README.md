# IoT Sensor Analysis — Why Mould is Returning

A diagnostic re-analysis of three months of bathroom sensor data, 
investigating why mould began returning three months after 
professional cleaning despite an active monitoring system.

This is the methodological foundation for the 
[Lidless Platform](https://github.com/jimyeong/lidless-controller)'s 
mould-risk prediction layer.

---

## The story

In February 2026, after losing a towel to mould, I built a sensor-
based alert system for the bathroom — a Sonoff Zigbee humidity 
sensor feeding into a backend that triggered alerts when air 
humidity stayed elevated. Around the same time, I had a 
professional clean the bathroom ceiling. The mould was visibly 
removed.

Three months later, I'm seeing early signs of regrowth — subtle 
discolouration, scattered specks beginning to form on the same 
ceiling.

This notebook is the diagnostic: **given that the original alert 
system was active, why did the underlying conditions for mould 
growth persist?**

---

## What this notebook does

1. **Reproduces the original production alert rule** (60% humidity 
   threshold + 90% ratio over 1-hour window) and evaluates how 
   often it fired against historical data.

2. **Reframes the problem using dew-point physics** — explains why 
   humidity alone is insufficient and introduces dew-point spread 
   as the predictive metric.

3. **Re-analyses the same dataset** with the corrected metric, 
   revealing that conditions for mould growth were met far more 
   frequently than the original rule captured.

4. **Quantifies the limitations** of the current single-sensor 
   setup — why T/RH alone cannot identify which interventions 
   actually move recovery time.

5. **Scopes the next phase** of sensor expansion (window state, 
   dehumidifier/fan power monitoring) to enable multivariate 
   analysis.

---

## Key findings

- The original humidity-based alert rule fired 2,128 times over 
  one month — frequent enough that the user felt informed.
- Re-evaluated with dew-point spread (≤ 3°C as warning threshold), 
  the same data shows **121 risk events**, with **51 sustained 
  beyond 30 minutes**.
- Average recovery time after a high-risk event: **28 minutes**, 
  with a substantial tail extending past 60 minutes.
- The visible result on the ceiling, three months on, confirms 
  the data: mould-favourable conditions were being met regularly 
  despite the alerts firing.

---

## What I claim

- **Data pipeline, sensor integration, manual labelling**: mine
- **Problem framing and threshold selection**: mine
- **Physics interpretation and metric choice**: mine
- **Findings and their implications for production design**: mine
- **Modelling code itself**: written with LLM assistance, which is 
  increasingly standard practice. What distinguishes the work is 
  the surrounding judgement, not the typing of sklearn calls.

---

## How this informed the production system

The findings shaped concrete design choices in the 
[Lidless Platform](https://github.com/jimyeong/lidless-controller):

- **Dew-point spread as primary risk metric** instead of raw 
  humidity. [Lidless Controller](https://github.com/jimyeong/lidless-controller) 
  calculates spread via the Magnus formula and tracks cumulative 
  exposure time.
- **Interpretable physical thresholds before ML.** Given small 
  labelled data and the need to explain risk to non-technical 
  users, v1 deliberately uses physically-grounded thresholds. ML 
  extensions are scoped for v2 once labelled production data 
  accumulates from the expanded sensor set.
- **Sensor-grounded NLP surface.** [Lidless Oracle](https://github.com/jimyeong/lidless-oracle) 
  exposes risk state through Claude API tool use, grounded in 
  actual sensor readings rather than generative speculation.

---

## Production status

The exploration here is deployed across the Lidless Platform:

| Layer | Repo | Stack |
|---|---|---|
| Sensor ingestion | [OpsCheck](https://github.com/jimyeong/ops-check-service) | Node.js, Fastify, PostgreSQL, MQTT QoS-1, RabbitMQ, outbox pattern |
| Risk analysis | [Lidless Controller](https://github.com/jimyeong/lidless-controller) | Python, Magnus formula, dew-point analysis |
| BFF / orchestration | [Lidless Hermes](https://github.com/jimyeong/lidless-hermes) | Node.js, Apollo GraphQL |
| User-facing chatbot | [Lidless Oracle](https://github.com/jimyeong/lidless-oracle) | React Native, Claude API, multilingual |

---

## Next in this series

The analysis here ran on what was available: bathroom air T/RH 
only. The limitations are documented in the notebook — recovery 
dynamics vary, but the data cannot identify **why** any given 
recovery was fast or slow.

The next iteration expands the sensor coverage:

- **Window contact sensor** (Sonoff SNZB-04P) — captures open/closed state
- **Smart plug on dehumidifier** (Sonoff S31 Lite ZB) — runtime + power
- **Smart plug on circulation fan** (Honeywell HT900E) — active air movement

The next notebook will analyse recovery dynamics across these 
intervention modes — answering which combination actually moves 
recovery time, and by how much. Data collection began on 
2026-05-11; the analysis will appear here once sufficient labelled 
events have accumulated.

---

## Repo structure

```
iot-sensor-analysis/
├── README.md
├── mould-risk-analysis.ipynb    (this analysis)
├── data/
│   └── bathroom.csv
└── images/
    ├── ceiling_mould.jpeg
    ├── honeywell_fan.jpeg
    ├── window_sensor.jpeg
    └── ...
```

---

## Hardware (current setup)

- **Sonoff Zigbee temperature/humidity sensor** in bathroom
- **Sonoff Zigbee USB dongle** on home Linux server
- **Zigbee2MQTT** forwarding to MQTT broker
- **PostgreSQL** for sensor data, indexed for time-range queries
- **Sonoff SNZB-04P window contact sensor** (newly added)
- **Sonoff S31 Lite ZB smart plugs** × 2 (newly added) — on Meaco dehumidifier and Honeywell HT900E circulation fan

---

## License

MIT
