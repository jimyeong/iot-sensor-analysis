# IoT Sensor Analysis — Why Mould is Returning

## The story so far

In February, I lost a towel to mould. That was the trigger to build 
this monitoring system — a Zigbee humidity sensor in the bathroom 
feeding into a backend that alerts when conditions stay elevated.

Around the same time, I had a professional clean the bathroom 
ceiling. The mould was visibly removed.

Three months later, I'm seeing early signs of regrowth — subtle 
discolouration, the same patterns starting to appear.

This notebook is the diagnostic. **Why is it coming back?**

## The hypothesis I started with — and why it was wrong

The original alert system used a simple rule:
> "If humidity exceeds 60% for sustained periods, warn the user."

It seemed reasonable. Humidity is the obvious metric. But it 
doesn't reflect the actual physics of mould growth.

Mould germinates when a surface stays at high relative humidity 
(typically ≥ 80%) for sustained periods. Surface RH isn't directly 
measured by an air sensor — but it's tightly related to **dew-point 
spread**: the gap between air temperature and dew point. When that 
gap closes, the surface is approaching condensation, and surface 
RH rises sharply.

So the real predictive features aren't:
- ❌ Air humidity %
- ❌ Peak humidity values

They are:
- ✅ Dew-point spread (T − T_dew)
- ✅ How long the spread stays within the danger zone (≤ 3°C)
- ✅ Cumulative time per day in that zone

## What I found when I re-analysed a month of data

[Image 2: Gap timeline — green spread line crossing 3°C threshold]

Over the past month, the dew-point spread crossed below the 3°C 
warning threshold far more often than I had appreciated.

[Image 1: Timeline of high mould risk events]
- Mould-risk windows (spread < 3°C) per month: dozens of events
- Average duration of each event: 26.7 minutes  
- Worst event: 69 minutes sustained
- Events lasting longer than 30 minutes: 38

[Image 4: Recovery time histogram]

The recovery distribution is bimodal — many events recover quickly 
(< 10 min) but a substantial group takes 30–50 minutes. This split 
likely reflects different ventilation conditions, but with only T/RH 
data I can't separate them.

This is enough exposure for spores to germinate repeatedly. 
**Cleaning removes existing mould, but it doesn't change the 
conditions that grew it.** The system was alerting on the wrong 
thing.

## Why the current system can't fully solve this

Two structural problems:

1. **No extractor fan in the bathroom.**
   Recovery has to rely on dehumidifier + passive ventilation 
   (window). The data shows this isn't fast enough — 38 events 
   per month exceeded 30-minute exposure, despite consistent 
   dehumidifier use.

2. **One-dimensional sensing.**
   Air T/RH alone can't separate "why" recovery was slow on a 
   given event. Was the window closed? Did the dehumidifier run? 
   Was outside air more humid than inside? Without those 
   variables, the model can't learn what intervention works.

## Next step — expanding sensor coverage

The next phase adds:
- **Window contact sensor** (Zigbee) — captures open/closed state
- **Smart plug on dehumidifier** (Zigbee) — captures runtime + power
- **Smart plug on circulation fan** (Zigbee) — adds active air 
  movement as a controlled variable
- **Surface-proximate T/RH sensor** — closer to the real surface 
  conditions where mould actually grows

With these, the next analysis can answer: *which combination of 
interventions actually moves recovery time, and by how much?*

That work continues in the next repo in this series:
[link to next repo when ready]

## Repo structure

iot-sensor-analysis/
├── README.md
├── humidity_analysis.ipynb   (this analysis)
├── data/
└── images/
