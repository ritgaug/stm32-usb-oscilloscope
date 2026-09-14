# Requirements

Initial requirements, first draft. Expected to evolve as the design progresses (iterative design).

Split into **Hardware** and **Firmware** requirements, since some features need both.

## Hardware Requirements

- **2 channels**: two physical connectors for probing signals
- **Graphic display**: for showing captured signals
- **0–3.3V input range**: ADC input is limited to this range (no negative voltage or high-voltage handling in this version)
- **Accurate to 500 kHz**: ADC must support a sample rate well above the Nyquist rate for 500 kHz to get a clean, high-resolution signal on screen (~10x oversampling target, i.e. ~5 MSPS)
- **USB connector**: for data streaming/logging to a PC
- **Small form factor**: keep the physical size as compact as reasonably possible
- **Buttons**: physical push buttons for channel activation
- **Button**: start/stop control for recording
- **LED indicators**: for power, channel activity, and recording status

## Firmware Requirements

- Display shows **1 channel at a time** on the onboard LCD
- **Data streaming and logging**: stream captured data to a PC in real time (also allows viewing the second channel on the PC, since the LCD can only show one)
- Buttons control **channel enable/select**
- Button controls **start/stop of recording**
- LED behavior implemented per indicator (power, channel activity, recording)

## Requirements marked as Hardware + Firmware

- **USB data streaming**: needs the physical USB connector (hardware) and the streaming/logging protocol implementation (firmware)

---

*Last updated: [2026-09-14]. Update this doc as requirements change during the design process.*
