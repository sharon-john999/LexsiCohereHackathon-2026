---
license: cc-by-nc-sa-4.0
task_categories:
  - text-generation
  - conversational
language:
  - en
tags:
  - aviation
  - ATC
  - air-traffic-control
  - pilot-training
  - general-aviation
  - gulf-coast
  - FAA
  - phraseology
  - VFR
  - flight-training
size_categories:
  - 1K<n<10K
pretty_name: Gulf Coast GA ATC Communications
---

# Gulf Coast General Aviation ATC Communications Dataset

A conversational dataset of pilot–ATC radio exchanges designed for **student pilots training in the Gulf Coast region** (Texas, Louisiana, Mississippi, Alabama, Florida panhandle). Built for fine-tuning language models as study aids for aviation radio communications.

## Dataset Overview

| Metric | Value |
|--------|-------|
| **Total conversations** | 5,401 |
| **Train split** | 4,860 |
| **Test split** | 541 |
| **Gulf Coast synthetic scenarios** | 3,400 |
| **Real ATC recordings (transcribed)** | 2,001 |
| **Scenario types** | 11+ |
| **Gulf Coast airports covered** | 26 |

## Two Configurations

### `sft` — For model fine-tuning
Standard conversational format with `messages` column (system/user/assistant roles). Ready to use with TRL's `SFTTrainer`.

```python
from datasets import load_dataset
ds = load_dataset("starlineventures/gulf-coast-ga-atc-communications", "sft")
```

### `study` — For human study & analysis
Detailed metadata with scenario type, airport, aircraft, callsign, region, full transcript, and structured messages JSON.

```python
from datasets import load_dataset
ds = load_dataset("starlineventures/gulf-coast-ga-atc-communications", "study")
```

## Scenario Types

| Scenario | Count | Description |
|----------|-------|-------------|
| **VFR Departure (Towered)** | 400 | Ground contact → taxi → tower → takeoff → departure handoff |
| **VFR Arrival (Towered)** | 400 | Approach contact → pattern entry → landing clearance → ground |
| **Pattern Work / Touch-and-Go** | 500 | Closed traffic, multiple patterns, student pilot training |
| **CTAF Non-Towered** | 400 | Self-announce at uncontrolled fields (common Gulf Coast GA) |
| **VFR Flight Following** | 350 | En-route radar services, traffic advisories |
| **Emergency Communications** | 200 | Engine failure, lost pilot, low fuel, electrical, medical |
| **Class B Transition** | 250 | Houston/New Orleans Bravo airspace transit requests |
| **Weather Diversion** | 250 | Thunderstorm avoidance (very common Gulf Coast) |
| **Go-Around** | 250 | Missed approach, re-sequence into pattern |
| **Progressive Taxi** | 200 | Step-by-step taxi at unfamiliar airports |
| **PIREPs** | 200 | Pilot weather reports (turbulence, icing, visibility) |
| **Real ATC (UWB-ATCC)** | 1,818 | Real pilot/controller exchanges from Prague (IFR reference) |
| **Real ATC (ATCO2)** | 183 | Real exchanges from Czech airports (phraseology reference) |

## Gulf Coast Airports Covered

### Class B
- **KHOU** — Houston Hobby
- **KMSY** — Louis Armstrong New Orleans International
- **KIAH** — George Bush Intercontinental

### Class C
- KPNS (Pensacola), KBPT (Beaumont), KGPT (Gulfport-Biloxi), KMOB (Mobile), KBTR (Baton Rouge), KLFT (Lafayette)

### Class D
- KDWH (David Wayne Hooks), KEFD (Ellington Field), KLCH (Lake Charles), KNEW (Lakefront NOLA), KSGR (Sugar Land), KCXO (Conroe)

### Non-Towered (Class G)
- KAXH (Houston Southwest), KLVJ (Pearland), KBYY (Bay City), KRKP (Rockport), KPQL (Pascagoula), KASD (Slidell), T78 (Liberty), 6R7 (Old Jezebel), and more

## Data Sources

1. **Synthetic Gulf Coast GA scenarios** — Generated using FAA Aeronautical Information Manual (AIM) Chapter 4 standard phraseology, correct frequencies, runway numbers, and procedures for real Gulf Coast airports.

2. **[Jzuluaga/uwb_atcc](https://huggingface.co/datasets/Jzuluaga/uwb_atcc)** — 20 hours of real ATC speech from Prague airport with pilot (`_PI`) and controller (`_AT`) role labels. CC BY-NC-SA 4.0.

3. **[Jzuluaga/atco2_corpus_1h](https://huggingface.co/datasets/Jzuluaga/atco2_corpus_1h)** — 1 hour of real ATC speech from Czech airports (ATCO2 project). Speaker roles encoded in recording IDs.

## Example Conversation

**Scenario: VFR Departure from Sugar Land Regional (KSGR), Class D**

```
✈️ [PILOT]: sugar land regional ground skyhawk november four two seven three alpha
   at the ramp with information golf request vfr departure to the southeast

📡 [ATC_GROUND]: skyhawk november four two seven three alpha sugar land ground
   taxi to runway one seven via bravo

✈️ [PILOT]: taxi to runway one seven via bravo seven three alpha

✈️ [PILOT]: sugar land tower seven three alpha holding short runway one seven
   ready for departure

📡 [ATC_TOWER]: seven three alpha runway one seven cleared for takeoff
   wind one eight zero at eight

✈️ [PILOT]: cleared for takeoff runway one seven seven three alpha

📡 [ATC_TOWER]: seven three alpha frequency change approved squawk vfr
   have a good flight

✈️ [PILOT]: squawk vfr seven three alpha thank you
```

## Use Cases

- **Fine-tune an LLM** as an interactive ATC radio practice partner
- **Flashcard / quiz generation** for pilot training
- **Phraseology pattern mining** — study correct readback/hearback procedures
- **Emergency procedure drilling** — practice mayday/pan-pan calls
- **Airport familiarization** — learn procedures at specific Gulf Coast airports

## Limitations

- Synthetic data uses randomized but realistic parameters; not every combination mirrors a real-world scenario
- Real ATC data is from European (Czech) airports — phraseology conventions differ slightly from FAA standards
- Audio is not included (text transcripts only)
- No ATIS broadcast text is included (only references to information codes)
- Gulf Coast weather patterns (sea breeze, tropical systems) are referenced but not modeled in detail

## Citation

If you use this dataset, please cite the source research corpora:

```bibtex
@article{zuluaga2022atco2,
  title={ATCO2 corpus: A Large-Scale Dataset for Research on Automatic Speech Recognition and Natural Language Understanding of Air Traffic Control Communications},
  author={Zuluaga-Gomez, Juan and others},
  journal={arXiv preprint arXiv:2211.04054},
  year={2022}
}
```
