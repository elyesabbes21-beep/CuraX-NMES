<div align="center">

# RehabPulse
### Quadriceps NMES Rehabilitation — Engineering Prototype

**A battery-powered, STM32-controlled high-voltage pulse generator, built as the electronics base of a quadriceps rehabilitation device.**

![Status](https://img.shields.io/badge/status-engineering%20prototype-orange)
![Platform](https://img.shields.io/badge/controller-STM32F401-blue)
![Application](https://img.shields.io/badge/application-NMES%20concept-2563eb)

</div>

---

## Overview

RehabPulse is an engineering prototype of a microcontroller-controlled electrical pulse generator, designed as the basis for a neuromuscular electrical stimulation (NMES) device for quadriceps rehabilitation. NMES uses electrical pulses to stimulate motor nerves and produce muscle contractions, which can help maintain muscle activation when voluntary exercise is limited, for example after a knee injury or surgery.

This project covers the electronics: a high-voltage generator delivering about 200 V, regulated by an STM32 microcontroller through voltage feedback, and a timed pulse output whose period and width are set by the firmware.

> **Status:** working bench prototype. It is not a clinically validated or approved medical device and must not be connected to a person.

## Product concept

![RehabPulse product concept — illustrative only](assets/rehabpulse-product-concept.png)

*Illustrative design concept only. This image is not a photograph of a manufactured device, and the interface values it shows are not the settings of the bench prototype.*

## The prototype

![Prototype hardware](assets/113c69c5-9c4a-4752-a950-8aee039827fa.jpg)

![Prototype setup](assets/7877ab75-01e0-4c86-a388-e4faea0017a0.jpg)

*The bench prototype, and the DC output of the generator measured at 200.4 V with a UNI-T UT33A multimeter.*

### Architecture

| Block | Role |
|---|---|
| Battery and charge/boost module | Powers the generator and the controller |
| High-voltage generator | Produces the stimulation supply of about 200 V (measured 200.4 V DC) |
| STM32F401CCU6 (BlackPill) | Regulates the generator and generates the pulse train |
| Voltage-feedback divider (100:1) | Scales the output voltage down for the STM32's ADC |
| Power transistor | Switching element of the high-voltage stage |

### Operating principle

1. The high-voltage generator charges its output capacitor to about 200 V.
2. A 100:1 resistor divider brings this voltage down to a level the STM32 can read on its ADC (PA0).
3. The firmware compares the measured voltage to two thresholds (hysteresis) and drives a control signal (PA1) that turns the generator on or off.
4. Independently, a hardware timer produces the stimulation pulse train on PA3, with a period and a pulse width defined in the firmware.

## Firmware

The firmware runs on an STM32F401CCU6 (BlackPill). It is written with PlatformIO and the Arduino framework for STM32, and uploaded through an ST-Link V2 over SWD.

### Pin assignment

| Pin | Function |
|---|---|
| PA0 | ADC input, voltage feedback from the 100:1 divider |
| PA1 | Generator control signal (1 / 0 with hysteresis) |
| PA3 | Stimulation pulse output (TIM2 channel 4, hardware PWM) |

### Voltage feedback and regulation

The ADC runs at 12 bits (0 to 4095) with a 3.3 V reference. Each reading is the average of 16 samples, which reduces noise. Because of the 100:1 divider, the voltage at the ADC pin is one hundredth of the generator output.

The control signal uses **hysteresis** with two thresholds. It goes high when the measured voltage rises above the upper threshold and goes low only when it falls below the lower threshold. Between the two it keeps its previous state, so it does not chatter around a single value. Both thresholds are constants at the top of the code.

### Pulse generation

PA3 is connected to TIM2, a 32-bit timer configured with a 1 µs resolution. The pulse period and the pulse width are therefore programmed directly in microseconds, through two constants. Once started, the pulse train is produced entirely by hardware and is not disturbed by the measurement loop.

### Source code

`src/main.cpp`

```cpp
#include <Arduino.h>

const int ADC_PIN   = PA0;   // feedback from the 100:1 divider
const int CTRL_PIN  = PA1;   // generator control signal
const int PULSE_PIN = PA3;   // stimulation pulse output (TIM2_CH4)

const float VREF      = 3.3f;   // ADC reference (V)
const float V_HIGH    = 2.0f;  // upper threshold at the ADC pin (V)
const float V_LOW     = 1.95f;  // lower threshold at the ADC pin (V)
const bool  INVERT_OUTPUT = false;

const uint32_t PULSE_PERIOD_US = 20000;  // pulse period in microseconds
const uint32_t PULSE_WIDTH_US  = 1000;  // pulse width in microseconds

bool state = false;   // memorised state of the control signal

// Voltage at the ADC pin, averaged over 16 samples
float readVoltage() {
  uint32_t sum = 0;
  for (int i = 0; i < 16; i++) sum += analogRead(ADC_PIN);
  return (sum / 16.0f) * VREF / 4095.0f;
}

// Hardware pulse train on PA3 (the timer counts in microseconds)
void setupPulse() {
  PinName pn = digitalPinToPinName(PULSE_PIN);
  TIM_TypeDef *inst = (TIM_TypeDef *)pinmap_peripheral(pn, PinMap_PWM);
  uint32_t channel  = STM_PIN_CHANNEL(pinmap_function(pn, PinMap_PWM));

  HardwareTimer *pulseTimer = new HardwareTimer(inst);
  pulseTimer->setMode(channel, TIMER_OUTPUT_COMPARE_PWM1, PULSE_PIN);
  pulseTimer->setOverflow(PULSE_PERIOD_US, MICROSEC_FORMAT);
  pulseTimer->setCaptureCompare(channel, PULSE_WIDTH_US, MICROSEC_COMPARE_FORMAT);
  pulseTimer->resume();
}

void setup() {
  analogReadResolution(12);
  pinMode(CTRL_PIN, OUTPUT);
  digitalWrite(CTRL_PIN, INVERT_OUTPUT ? HIGH : LOW);
  setupPulse();
}

void loop() {
  float v = readVoltage();

  if (!state && v > V_HIGH) state = true;        // above the upper threshold
  else if (state && v < V_LOW) state = false;    // below the lower threshold
  // between the two thresholds the previous state is kept (hysteresis)

  digitalWrite(CTRL_PIN, (state != INVERT_OUTPUT) ? HIGH : LOW);
  delay(10);
}
```

`platformio.ini`

```ini
[env:blackpill_f401cc]
platform = ststm32
board = blackpill_f401cc
framework = arduino
upload_protocol = stlink
```

## Results

| Measurement | Result | Instrument |
|---|---|---|
| DC output voltage of the generator | 200.4 V | UNI-T UT33A multimeter |

## Safety notice

**Do not connect this prototype to a person.** A generator output of about 200 V does not establish the safety of a patient-facing stimulation output, and poorly designed or improperly applied electrical stimulation can cause serious injury.

The voltage-feedback divider is not galvanically isolated from the controller. Human use would require a qualified medical-device engineering review, a documented risk analysis, verified output current and charge limits, isolation and fault protection, and compliance with applicable medical-device regulations.

## Limitations

- Clinical effectiveness has not been established.
- The product concept image is illustrative and does not show a manufactured device.
- The prototype has not been tested on a person, and no treatment protocol or settings are provided.
- No regulatory approval or certification is claimed.
