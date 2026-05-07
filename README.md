# IoT Sensor Analysis
 
Methodological foundation for the [Lidless Platform](https://github.com/jimyeong/lidless-controller)'s mould-risk prediction layer — defining the problem, evaluating signal quality, and informing the production model design from real home sensor data.
 
---
 
## What this project is
 
This repo is the exploration phase that informed how mould-risk is detected in the production Lidless Platform. The goal was not to ship a model, but to answer four questions before committing to a production approach:
 
1. How does mould risk manifest in sensor signals in this physical setting?
2. Which sensor channels carry predictive value, and which are noise?
3. What is the right prediction target — instantaneous humidity, dew-point spread, recovery time after moisture events?
4. What modelling approach is honest given the available data volume and labelling cost?
The notebooks are exploratory by design. The framing, labelling decisions, feature interpretation, and conclusions about what to deploy are mine. The modelling code itself was written with LLM assistance — increasingly standard practice across the field. What distinguishes the work is the surrounding judgement, not the typing of sklearn calls.

---

<p align="center">
  <img src="images/image1.jpg" width="45%" />
  <img src="images/image2.jpg" width="45%" />
</p>

![Bathroom humidity analysis](images/humid_analysis.png) 
<img width="423" height="435" alt="Screenshot 2026-04-26 at 16 59 08" src="https://github.com/user-attachments/assets/0b309fa6-2c03-467d-aa64-f3d4445f7f85" /> 


 
---
 
## Why I built this
 
I lost a towel to mould. That was the immediate trigger. The bigger worry was that as a tenant, I had no objective record of humidity conditions in the flat — if a landlord ever pushed mould damage onto me as my responsibility, I'd have nothing to point to. Managing mould "by feel" felt fragile.
 
So I started measuring — with the goal of moving from *"I noticed mould has appeared"* to *"the system flagged the conditions before it formed."*
 
---
 
## What I built (hardware + pipeline)
 
End-to-end, from sensor to database:
 
### Bathroom
- Sonoff Zigbee temperature/humidity sensor (paired via Zigbee USB dongle on a home Linux server)
- Zigbee2MQTT configured to forward sensor messages to MQTT
- PostgreSQL set up locally to store readings; schema and ingestion logic written by me
### Kitchen
- ESP32 with GP2Y dust sensor and DHT22 temperature/humidity sensor
- Firmware adapted from open-source examples; circuit wired on a breadboard and deployed in a waterproof enclosure on the kitchen counter
### Living room
- BME680 sensor (temperature, humidity, pressure, gas resistance)
### Manual labelling
Over ~7.5 days in the bathroom and ~3 days in the kitchen/living room, I logged events by hand:
- Showers, ventilation (window open/closed)
- Cooking, air fryer, microwave usage
That produced ~3,000 labelled events aligned with sensor timestamps — the labelled dataset the analysis is built on.
 
![hardware setup](images/hardware_dev.jpg)
![kitchen enclosure](images/hardware_kitchen.jpg)
 
---
 
## What the data showed
 
Four findings that changed the design direction for the production system:
 
**1. Mould-risk windows after showers are longer than expected.**  
Humidity stays elevated for an extended period after the shower ends. The recovery time matters more than the duration of the shower itself — a 5-minute shower with poor ventilation can produce a longer high-risk window than a 15-minute shower with the window open.
 
**2. Ventilation dominates the recovery curve.**  
Window opening has a much bigger effect on the rate of return to safe humidity than instinct suggests. Without measurement, this would be invisible — humans habituate to ambient humidity and lose the ability to estimate it.
 
**3. The "obvious" metric (humidity %) is a lagging signal.**  
Temperature and dew-point spread respond earlier to moisture events. By the time relative humidity crosses a threshold, the conditions for condensation have often already been present for some time. Predictive features should track the spread, not the level.
 
**4. Class imbalance is structural.**  
Real mould-formation events are rare in any practical labelling window. This forces a deliberate choice between (a) interpretable physical-threshold classification that doesn't need labelled positives, or (b) much longer-term data accumulation before ML training is honest.
 
---
 
## What this informed in the production system
 
The findings above shaped concrete design choices across the Lidless Platform:
 
- **Recovery-time focus over instantaneous humidity.**  
  [Lidless Controller](https://github.com/jimyeong/lidless-controller) calculates dew-point spread via the Magnus formula and tracks cumulative time spent near the dew-point threshold, rather than triggering on humidity peaks alone.
- **Interpretable physical thresholds before ML.**  
  Given small labelled data and the need to explain risk to non-technical users (housing tenants), v1 deliberately uses physically-grounded thresholds rather than a black-box classifier. ML extensions are scoped for v2, once labelled production data accumulates.
- **Sensor-grounded NLP surface.**  
  [Lidless Oracle](https://github.com/jimyeong/lidless-oracle) exposes risk state through Claude API tool use, grounded in actual sensor readings — the chatbot retrieves and explains the data rather than generating speculative advice about mould.
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
 
## Roadmap — v2 ML layer
 
Scoped extensions once labelled production data accumulates beyond the current ~3,000 events:
 
- **Recovery-time prediction** — Random Forest baseline on time-to-safe after moisture events; comparison against the v1 physical-threshold baseline as a fairness check (does ML actually beat physics here?).
- **Drift monitoring** — input-distribution monitoring on sensor channels, with alerting when feature distributions diverge from training-time baselines.
- **Active labelling via the chatbot** — using Lidless Oracle to surface uncertain cases to the user and capture corrections, building labelled data through deployment rather than upfront.
- **Retraining cycle** — periodic retraining triggered by accumulated labelled corrections, with held-out evaluation against the previous model.
---
 
## Repo structure
```
iot-sensor-analysis/
├── README.md
├── humidity_analysis.ipynb              (LLM-assisted exploration, bathroom)
├── kitchen_livingroom_analysis.ipynb    (LLM-assisted exploration, kitchen + living room)
├── data/
│   ├── sample_bathroom_data.csv
│   ├── sample_events/data.csv
│   ├── sample_kitchen_data.csv
│   └── sample_livingroom_data.csv
└── images/
    ├── hardware_dev.jpg
    ├── hardware_kitchen.jpg
    └── ...
```




