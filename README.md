# IoT Sensor Analysis

Personal project exploring what you can do with data from cheap home sensors.

It started with a practical problem: mould forming in my bathroom. I wanted to understand why it was happening, and whether sensor data could help predict when conditions were becoming risky. Once the data pipeline was running, I expanded to the kitchen and living room.

---

## Hardware

**Bathroom** — Zigbee temperature/humidity sensor connected to a home server running Zigbee2MQTT and PostgreSQL. Data flows in automatically every few minutes.

**Kitchen** — ESP32 with a GP2Y dust sensor and DHT22 temperature/humidity sensor, deployed in a waterproof enclosure on the kitchen counter.

**Living room** — BME680 sensor (temperature, humidity, pressure, gas resistance).

<img src="images/image1.jpg" width="500"/>
<img src="images/image2.jpg" width="500"/>

---

## Notebooks

### `humidity_analysis.ipynb` — Bathroom sensor
~5,000 readings over 7.5 days.

- Calculating dew point (condensation risk) from raw sensor readings
- Identifying shower events using threshold-based labelling
- Linear regression to predict dew point — and why the near-perfect R² here is expected, not impressive

![Bathroom humidity analysis](images/humid_analysis.png)

---

### `kitchen_livingroom_analysis.ipynb` — Kitchen & living room
945 kitchen readings, 9,333 living room readings, 22 manually labelled events over 3 days.

- Loading and cleaning kitchen sensor data (Korean timestamps, ADC conversion, warmup noise)
- Converting raw GP2Y ADC values to µg/m³
- Aligning manually logged events with sensor timestamps
- Visualising cooking and ventilation effects on air quality
- Living room air quality exploration

![Kitchen dust with cooking and ventilation events](images/kitchen_dust_timeline.png)

---

## Repo structure

```
iot-sensor-analysis/
├── README.md
├── .gitignore
├── humidity_analysis.ipynb
├── kitchen_livingroom_analysis.ipynb
├── images/
│   ├── hardware_dev.jpg
│   ├── hardware_kitchen.jpg
│   ├── humid_analysis.png
│   ├── kitchen_dust_timeline.png
│   ├── cooking_vs_idle.png
│   └── livingroom.png
└── data/
    ├── sample_humid_temp.csv
    ├── sample_kitchen.csv
    ├── sample_livingroom.csv
    └── sample_events.csv
```

Full datasets available on request.

---

## What I was trying to learn

I'm a software engineer, not a data scientist. The goal wasn't to build a production ML system — it was to understand the pipeline end to end: what it takes to go from raw sensor output to something that could inform a real decision, and where the gaps and failure modes are.

The most useful things I took away:

**Labelling is harder and more important than model selection.** The analysis code took hours. Creating consistent labels — being present, recording times, deciding what counts as a cooking event — took longer and required more discipline.

**What a metric measures and what you want to measure are often different things.** Episode duration in the bathroom analysis measures how long humidity stays elevated, not how long the shower ran. Without ventilation, those can be very different numbers. Knowing that distinction matters before drawing conclusions.

**High accuracy can be misleading on imbalanced data.** Idle events make up ~90% of the dataset. A model that always predicted idle would be 90% accurate and completely useless. Macro F1 and per-class recall tell a more honest story.

**R² near 1.0 isn't always impressive.** The dew point regression achieved R² ≈ 0.99 — because dew point is mathematically derived from temperature and humidity. The model is rediscovering a known equation, not finding hidden patterns. Recognising that is the point.