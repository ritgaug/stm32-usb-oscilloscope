# Design Notes

## MCU and Display Selection

**STM32L432KC chosen as the microcontroller.**
- Small package size (UFQFPN32), which fits the small form factor requirement.
- Onboard 12-bit ADC capable of up to 5 MSPS, well above the 500 kHz bandwidth requirement even with oversampling.
- Native USB support, needed for the data streaming requirement.
- Ultra low power chip, which keeps a future battery powered version feasible.
- Using the STM32Cube ecosystem, which is widely used in industry and worth learning.

**2.8 inch, 192x96 SPI graphic LCD chosen as the display.**
- SPI interface fits the limited pin count on the STM32L432KC, an 8 bit parallel interface would use too many pins.
- White backlight and resolution are enough detail to draw signal waveforms clearly.
- Uses the ST75256 controller chip. Learned that the panel (physical screen) and the controller (driver chip) are separate things and can be mixed and matched, so both datasheets matter, one for pin interface and sizing, one for the communication protocol and timing.
- Created the schematic symbol for this display manually using KiCad's symbol editor, since the distributor did not provide one.

## Power Rail Planning

Checked both components' datasheets for their voltage requirements before deciding on rails.

- STM32L432KC operates on 1.71 to 3.6 volts, with an absolute maximum of 4V.
- LCD module logic supply (VDD) typically runs at 3.3 volts, with a maximum rating of 4V for the logic side and 19V for the backlight side.

Since both components can run on 3.3 volts, only a single 3.3V power rail is needed for this design. If the two parts had required different voltages, multiple rails would have been necessary.

## Decoupling Capacitors

Learned the purpose and placement of decoupling (bypass) capacitors.

- As an integrated circuit switches on and off, it creates noise on its input voltage rail. A decoupling capacitor acts as a local power supply to stabilize that rail against voltage dips caused by switching.
- Followed ST's application note (AN4555) for the STM32, which recommends pairing a 100 nF ceramic capacitor with a 10 microfarad tantalum or ceramic capacitor in parallel on each VDD and VSS pair, placed as close to the chip as possible.
- Checked whether the LCD module needed its own decoupling capacitors. Found that the module already has capacitors on the board, confirmed by tracing the VDD pin to an existing capacitor and by checking the vendor's interfacing guide, which also does not show any external decoupling for this part. Decided not to add extra ones for now, can revisit if needed later.
