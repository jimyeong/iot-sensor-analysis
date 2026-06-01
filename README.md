# IoT Sensor Analysis — Why Mould is Returning

A diagnostic, data-driven investigation into why mould began returning to a bathroom three months after professional cleaning — despite an active monitoring system — and what actually moves the conditions that grow it.

This is the methodological foundation for the [Lidless Platform](#how-this-informed-the-production-system)'s mould-risk prediction layer.

## The story

In February 2026, after losing a towel to mould, I built a sensor-based alert system for the bathroom — a Sonoff Zigbee humidity sensor feeding a backend that fired alerts when humidity stayed elevated. Around the same time, I had the bathroom ceiling professionally cleaned. The mould was visibly removed.

Three months later, the early signs of regrowth appeared on the same ceiling. The monitoring system had been running the whole time. So the question became: **given that the alert system was active, why did the conditions for mould growth persist — and what intervention actually changes them?**

The series answers this in two parts:

- **Notebook 1 — diagnosis.** The original alert watched the wrong metric. Humidity is a consequence; the cause is surfaces reaching near-saturation, which dew-point spread captures and relative humidity does not.
- **Notebook 2 — intervention.** Having identified the right metric, I added a circulation fan as a controlled variable and measured whether it actually reduces time spent in the risk zone.

## Notebook 1 — diagnosis: humidity was the wrong metric

The original production alert rule, conceptually: *"if at least 90% of humidity readings in the last hour exceed 60%, fire an alert."*

Re-evaluated against historical data, that rule fired often — but it was measuring the wrong thing. Mould doesn't germinate because air is humid; it germinates because the air touching a cold surface reaches saturation. The **dew-point spread** (air temperature − dew point) measures how close conditions are to that point. A small spread means surfaces only slightly colder than the room air are already at risk.

Dew point is computed via the Magnus-Tetens approximation; spread ≤ 3°C is used as a deliberately conservative warning threshold (condensation strictly begins at 0°C, but sustained near-saturation is enough for mould, and a room-air sensor can't measure the actual surface temperature — the 3°C buffer absorbs that uncertainty).

**Key findings (notebook 1):**
- The original humidity rule fired frequently over the month — frequent enough that I felt informed.
- Re-evaluated with dew-point spread (≤ 3°C), the same data showed **121 risk events, 51 of them sustained beyond 30 minutes** — a burden the humidity rule had largely missed.
- The visible regrowth on the ceiling, three months on, confirmed the data: mould-favourable conditions were being met regularly despite the alerts firing.

## Notebook 2 — intervention: does a circulation fan actually help?

Having established the right metric, I tested an intervention. I added a circulation fan, put Zigbee smart sockets on it and on the dehumidifier, and recorded what happened around each shower. The fan was introduced on **2026-05-11**, giving a natural before/after split.

To make the comparison fair I used **matched 20-day windows** (the 20 days before vs the 20 days after fan introduction) and verified the two periods were comparable on temperature and absolute moisture before drawing conclusions.

**Key findings (notebook 2):**
- **Recovery after a humidity peak got faster** — median time to bring spread back to a safe level fell from ~14.7 to ~8.6 minutes (about 40% quicker).
- **Cumulative time in the risk zone (spread ≤ 3°C) fell ~21%** — from roughly 40 to 30 minutes per day.
- This held *even though* the after-fan period was absolutely more humid (mean dew point up ~0.95°C, spread essentially flat). The improvement runs against the seasonal drift, not with it — which strengthens the attribution to the fan. RH alone would have been misleading here, since it falls automatically as temperature rises; spread is the temperature-robust metric.
- **The reframe that matters most:** faster recovery is not the main lever. Mould that has already set hyphae into a surface activates based on *how long* conditions stay favourable — cumulative exposure — not how quickly any single event clears. Speed is secondary; duration is what grows mould.

**Method notes (notebook 2):**
- Timestamps parsed as UTC then converted to `Europe/London` (avoids a silent one-hour shift).
- Risk-zone episodes de-duplicated so a single event flickering around the threshold is counted once; exposure totals are duration-weighted and normalised per day, so unequal window lengths don't distort the comparison.
- Device ON/OFF state is not yet labelled within each recovery episode; the before/after split assumes consistent window and dehumidifier behaviour across periods. Removing that residual confounding is the next step.

## What I claim

- **Data pipeline, sensor integration, manual labelling**: mine
- **Problem framing and threshold selection**: mine
- **Physics interpretation and metric choice**: mine
- **Experiment design (matched windows, confound checks) and findings**: mine
- **Modelling code itself**: written with LLM assistance, which is increasingly standard practice. What distinguishes the work is the surrounding judgement — metric choice, confound control, and interpretation — not the typing of pandas calls.

## How this informed the production system

The findings shaped concrete design choices in the Lidless Platform:

- **Dew-point spread as primary risk metric** instead of raw humidity. The Lidless Controller calculates spread via the Magnus formula and tracks cumulative exposure time.
- **Cumulative exposure, not instantaneous readings**, as the quantity to minimise — a direct consequence of notebook 2's reframe.
- **Interpretable physical thresholds before ML.** Given small labelled data and the need to explain risk to non-technical users, the system deliberately uses physically-grounded thresholds. ML extensions are scoped for later, once labelled production data accumulates.
- **Sensor-grounded NLP surface.** Lidless Oracle exposes risk state through Claude API tool use, grounded in actual sensor readings rather than generative speculation.

## Production status

| Layer | Repo | Stack |
|---|---|---|
| Sensor ingestion | OpsCheck | Node.js, Fastify, PostgreSQL, MQTT QoS-1, RabbitMQ, outbox pattern |
| Risk analysis | Lidless Controller | Python, Magnus formula, dew-point analysis |
| BFF / orchestration | Lidless Hermes | Node.js, Apollo GraphQL |
| User-facing chatbot | Lidless Oracle | React Native, Claude API, multilingual |

## Next in this series

Notebook 2 measured the fan as a *manually operated* variable — I switched it on by hand after each shower. The limitation is structural: a flat with no extract ventilation, operated by hand, can't fully prevent risk-zone accumulation. You can't not shower.

The next experiment closes the loop into a crude **dMEV-style** (decentralised mechanical extract ventilation) automation, built from parts already proven here: the Zigbee humidity sensor detects the shower spike, and the Zigbee socket drives the circulation fan to high automatically. The question: with that loop closed, does the bathroom cross into the risk zone at all?

> **Safety caveat (applies to existing growth):** running a fan over a surface that already has established mould disperses spores into the air — onto other surfaces and into the lungs. The fan is a *preventive* tool for clean surfaces, not a remedy for existing growth. Remove mould properly first; only then does air movement help rather than spread the problem.

## Repo structure

```
iot-sensor-analysis/
├── README.md
├── mould-risk-analysis.ipynb        (notebook 1 — diagnosis)
├── mould-risk-analysis-v2.ipynb     (notebook 2 — fan intervention)
├── data/
│   ├── bathroom.csv                 (notebook 1)
│   ├── humid_temp_sensor.csv        (notebook 2)
│   ├── contact_sensor.csv           (notebook 2)
│   └── smart_socket.csv             (notebook 2)
└── images/
    ├── ceiling_mould.jpeg
    ├── honeywell_fan.jpeg
    ├── window_sensor.jpeg
    └── ...
```

## Hardware (current setup)

- **Sonoff Zigbee temperature/humidity sensor** in bathroom
- **Sonoff Zigbee USB dongle** on home Linux server
- **Zigbee2MQTT** forwarding to MQTT broker
- **PostgreSQL** for sensor data, indexed for time-range queries
- **Sonoff SNZB-04P window contact sensor**
- **Sonoff S31 Lite ZB smart plugs × 2** — on Meaco dehumidifier and Honeywell HT900E circulation fan

## License

MIT
