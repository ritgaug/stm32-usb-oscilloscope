# STM32 USB Oscilloscope

A two channel, USB streaming oscilloscope built from scratch, schematic, PCB, and firmware, as a way to actually learn embedded systems design rather than just read about it.

**Status:** In progress, currently at schematic stage.

## Why this project

I'm a computer engineering student heading into embedded systems roles, and I wanted a project that forced me through the entire pipeline, not just writing code, but making real hardware decisions: reading datasheets, sizing power rails, choosing peripherals, and figuring out why a design works instead of just copying one that does. An oscilloscope felt like a good fit: it touches analog input, ADC/DMA, a display, USB communication, and real time firmware concerns all in one device.

This repo is my running log of that process, the decisions, the mistakes, and eventually the finished thing.

## What it does

- Captures 2 channels of analog input, 0 to 3.3V range
- Accurate up to 500 kHz, sampled well above Nyquist (about 10x oversampling) for a clean signal on screen
- Displays live signal data on a 2.8 inch 192x96 SPI graphic LCD
- Streams captured ADC data to a PC over USB for live plotting and logging
- Physical buttons for channel enable/select and start/stop recording
- LED indicators for power, channel activity, and recording status
- Small form factor design

## Repo structure

- `hardware/` for KiCad schematic and PCB design files
- `firmware/` for embedded C source (STM32)
- `docs/` for requirements, BOM, design notes, and progress photos

## Design highlights

- Chose an STM32L432KC for its small package size, onboard 12 bit ADC (up to 5 MSPS), low power consumption, and native USB support
- Chose a 2.8 inch 192x96 SPI graphic LCD (ST75256 controller) since the SPI interface fits the STM32's limited pin count, and manually created its schematic symbol in KiCad since none was provided
- Single 3.3V power rail, since both the MCU and LCD run on 3.3V after checking each part's datasheet
- Followed ST's application note for decoupling capacitor placement on the MCU, and confirmed the LCD module already has its own onboard decoupling before deciding not to add extra ones
- Firmware will be built around a hybrid interrupt/DMA/main loop architecture: DMA handles ADC sampling directly, lightweight interrupts flag button and timer events, and the main loop handles slower work like display rendering and data transmission, so nothing blocks or gets missed

## Tools and stack

STM32, C, KiCad, SPI, ADC/DMA, USB CDC, Logic analyzer for debugging

## Progress log

See [`docs/`](./docs) for the full requirements list, BOM, and design notes as the project develops.

---

More detail and a demo video coming as this gets built out, check back or see the commit history for progress.
