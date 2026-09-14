# STM32 USB Oscilloscope

A two-channel, USB-streaming oscilloscope built from scratch, schematic, PCB, and firmware, as a way to actually learn embedded systems design rather than just read about it.

**Status:** 🚧 In progress, currently at schematic / breadboard stage.

## Why this project

I'm a computer engineering student heading into embedded systems roles, and I wanted a project that forced me through the *entire* pipeline, not just writing code, but making real hardware decisions: reading datasheets, sizing power rails, choosing peripherals, and figuring out why a design works instead of just copying one that does. An oscilloscope felt like a good fit: it touches analog input, ADC/DMA, a display, USB communication, and real-time firmware concerns all in one device.

This repo is my running log of that process, the decisions, the mistakes, and (eventually) the finished thing.

## What it does

- Captures 2 channels of analog input, 0–3.3V range
- Accurate up to 500 kHz, sampled well above Nyquist (~10x oversampling) for a clean signal on screen
- Displays live signal data on a 2.8" 192x96 SPI graphic LCD
- Streams captured ADC data to a PC over USB for live plotting/logging
- Physical buttons for channel enable/select and start/stop recording
- LED indicators for power, channel activity, and recording status
- Small form factor design

## Repo structure

- `hardware/` → KiCad schematic and PCB design files
- `firmware/` → Embedded C source (STM32)
- `docs/` → Requirements, BOM, design notes, and progress photos

## Design highlights

- Chose an STM32L432KC for its small package size, onboard 12-bit ADC (up to 5 MSPS), low power consumption, and native USB support
- Single 3.3V power rail, regulated down from USB 5V, since both the MCU and LCD run at 3.3V
- Firmware built around a hybrid interrupt/DMA/main-loop architecture: DMA handles ADC sampling directly, lightweight interrupts flag button/timer events, and the main loop handles slower work like display rendering and data transmission — so nothing blocks or gets missed
- Verified SPI communication with a logic analyzer against the LCD controller's datasheet timing

## Tools & stack

STM32 · C · KiCad · SPI · ADC/DMA · USB CDC · Logic analyzer for debugging

## Progress log

See [`docs/`](./docs) for the full requirements list, BOM, and design notes as the project develops.

---

More detail and a demo video coming as this gets built out, check back or see the commit history for progress.
