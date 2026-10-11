# RehabPulse — Quadriceps NMES Rehabilitation Prototype

> A microcontroller-controlled electrical pulse generation prototype exploring a potential platform for quadriceps neuromuscular electrical stimulation (NMES).

![RehabPulse product concept](assets/rehabpulse-product-concept.png)

## Overview

RehabPulse is an engineering prototype concept intended to explore controlled electrical pulse generation for potential quadriceps rehabilitation applications. The long-term idea is to support muscle activation during periods when voluntary exercise is limited, such as during selected stages of knee rehabilitation.

**Current status:** electronics prototype. This repository must not be interpreted as evidence of clinical effectiveness or approval for human use.

## Problem

Reduced activity during recovery from certain knee injuries or surgeries can contribute to quadriceps weakness and muscle loss. NMES is a rehabilitation technique in which electrical pulses stimulate motor nerves to produce muscle contractions.

## Prototype architecture

The current prototype is described as containing:

- **High-voltage generator:** nominal output around 200 V.
- **STM32 microcontroller:** controls the generator and pulse-switching logic.
- **Voltage feedback:** a resistor divider scales the generator output for monitoring and regulation.
- **Switching stage:** a transistor-controlled stage gates pulse output.
- **Firmware:** regulates the generator output and controls pulse timing.

These descriptions should be updated to match the actual circuit and firmware in the repository.

## How it works

1. The high-voltage generator produces the target output.
2. A voltage divider scales the output for the STM32 feedback input.
3. The STM32 adjusts generator control to maintain the target voltage.
4. A separate control signal drives the switching stage to create timed pulses.
5. Output behavior is verified using suitable laboratory equipment and a test load.

## Validation evidence to add

- [ ] Actual circuit schematic with component values and test points
- [ ] Clear photos of the assembled hardware
- [ ] STM32 firmware and build instructions
- [ ] Multimeter readings with test conditions recorded
- [ ] Oscilloscope captures taken from the real hardware
- [ ] Output measurements under a documented test load
- [ ] Startup, stability, and fault-condition test results
- [ ] Bill of materials and known limitations

Do not label generated or illustrative graphics as real measurements. Record the instrument, probe, load, supply voltage, and test conditions for each measurement.

## Product concept

The proposed product direction is a compact, portable NMES platform initially intended for evaluation with rehabilitation clinics and healthcare professionals. A future version could potentially support supervised home rehabilitation, subject to clinical evidence, risk controls, and applicable regulatory requirements.

The concept image is illustrative only. It does not represent a validated or manufactured medical device, and displayed specifications are not verified product specifications.

## Safety and limitations

**This prototype is not intended for connection to a person.** A nominal 200 V generator output does not establish the safety of the patient-facing output. Electrical stimulation hardware can cause serious injury if improperly designed or used.

Before any human use, the design requires qualified medical-device and electrical-safety review, risk analysis, verified current and charge limits, appropriate isolation and fault protection, suitable output monitoring, clinical oversight, and compliance with the applicable medical-device regulations. Do not test on a person based on this repository.

## Potential business model

1. **Initial customers:** rehabilitation clinics and hospitals for supervised evaluation.
2. **Validation:** collaborate with qualified clinicians to define requirements and evaluate safety and usability.
3. **Future market:** consider clinician-supported home use only after appropriate validation and regulatory clearance.

## License

Choose a license before public release. If you intend to protect commercial intellectual property, review licensing and disclosure choices before publishing detailed circuit files.
