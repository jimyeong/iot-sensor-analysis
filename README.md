# IoT Sensor Analysis

Personal project exploring whether cheap home sensors and a basic data pipeline can do something useful — specifically, help me see when mould risk is building up in my flat before I notice it visually.

--- 

## Why I built this

I lost a towel to mould. That was the immediate trigger. The bigger worry was that as a tenant, I had no objective record of humidity conditions in the flat. If a landlord ever pushed mould damage onto me as my responsibility, I'd have nothing to point to. Managing mould "by feel" felt fragile.

So I started measuring.

---

## What I built (hardware + pipeline)

This is the part I did end-to-end:

**Bathroom**
- Sonoff Zigbee temperature/humidity sensor (paired via a Zigbee USB dongle on a home Linux server)
- Zigbee2MQTT installed and configured to forward sensor messages to MQTT
- PostgreSQL set up locally to store readings; schema and ingestion logic written by me

**Kitchen**
- ESP32 with GP2Y dust sensor and DHT22 temperature/humidity sensor
- Firmware was adapted from open-source examples; I wired the circuit on a breadboard and deployed it inside a waterproof enclosure on the kitchen counter

**Living room**
- BME680 sensor (temperature, humidity, pressure, gas resistance)

**Manual labelling**
Over ~7.5 days in the bathroom and ~3 days in the kitchen/living room, I logged events by hand:
- Showers, ventilation (window open/closed)
- Cooking, air fryer, microwave usage

That gave me ~3,000 labelled events aligned with sensor timestamps.

<p align="center">
  <img src="images/image1.jpg" width="45%" />
  <img src="images/image2.jpg" width="45%" />
</p>

![Bathroom humidity analysis](images/humid_analysis.png) 
<img width="423" height="435" alt="Screenshot 2026-04-26 at 16 59 08" src="https://github.com/user-attachments/assets/0b309fa6-2c03-467d-aa64-f3d4445f7f85" /> 





---  

## What I did with the data

This is where I want to be clear about what's mine and what isn't.

**Mine**: data collection, the pipeline, the labelling, the judgment calls about what counted as a "shower event" vs "ventilation window", deciding what was worth measuring in the first place.

**Not mine**: the ML analysis in the notebooks. I'm a software engineer, not a data scientist. I gave the labelled dataset to an AI assistant and asked it to explore the data using ML approaches (classification, regression). The notebooks are exploratory — they're records of what came back, not a production model I built.

I include them here because the *exploration* taught me real things, even if I didn't write the analysis code:

- Mould-risk windows after showers are longer than I expected. Humidity stays elevated long after the shower ends — it's the recovery time that matters, not the duration of the shower itself.
- Ventilation has a much bigger effect on recovery than I'd assumed.
- The "obvious" metric (humidity %) is a lagging signal compared to temperature/dew-point spread.

These insights changed how I think about the problem. They didn't come from me writing the model.

--- 


## Where this is going

I'm now rewriting the analysis layer myself, focused on what I can actually explain and defend:

- Dew point calculation (Magnus formula) — straightforward physics, no ML needed
- Threshold-based mould risk classification — interpretable, defensible, doesn't require labelled training data I don't have
- A small mobile-facing layer to surface the current risk state

The earlier notebooks stay in the repo as honest history. They're how I figured out the problem was worth working on.

--- 


## Repo structure

```
iot-sensor-analysis/
├── README.md
├── humidity_analysis.ipynb       (AI-assisted exploration, bathroom)
├── kitchen_livingroom_analysis.ipynb  (AI-assisted exploration, kitchen + living room)
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

Full datasets available on request.



--- 
## Honest summary

I built a working IoT data pipeline for my flat, labelled the data myself, and used AI tools to explore what the data could tell me. That exploration confirmed the problem is worth solving and gave me direction. The next step — which I'm working on now — is the analysis I can actually own and explain.

