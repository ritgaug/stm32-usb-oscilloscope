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

## Decoupling Capacitor Reference

Used ST application note AN4555 (Getting started with STM32L4 Series and STM32L4+ Series hardware development) to determine decoupling capacitor placement. The note specifies that each power supply pair should be decoupled with a 100 nF ceramic capacitor and a 10 microfarad tantalum or ceramic capacitor connected in parallel, placed as close as possible to the relevant pins. Added these capacitors to the design following this recommendation and the application note's reference design, with one pair for each VDD/VSS pair on the MCU.

## STM32CubeMX Pin Configuration

Used STM32CubeMX to configure and assign pins on the STM32L432KC.

- SPI1 set to transmit only master mode, since the LCD is write only, with software controlled chip select instead of hardware NSS.
- ADC1 enabled, set to single ended, for reading the analog input channels.
- Learned that GPIO interrupt lines are shared across pins with the same number across different ports (for example PA6 and PB6 share the same EXTI line), so each interrupt pin needed a unique line number. Assigned PA4 to button channel 2, PA6 to button record, and PB1 to button channel 1.
- Grouped GPIO pins physically based on where they will sit on the PCB, buttons grouped near the SPI/LCD pins since they are intended to sit underneath the LCD on the board.
- LEDs assigned as plain GPIO outputs, buttons assigned as interrupts.
- USB enabled as a virtual COM port (CDC) for data streaming to a PC.
- Debug interface configured for SWD (serial wire debug) instead of JTAG, since SWD only needs two GPIO wires, which suits the pin constrained 32 pin package. UART1 also enabled for debug logging.

## LED and Resistor Selection

Calculated current limiting resistor values using R = (Vsupply minus Vf) / I, with Vsupply at 3.3V. Chose LEDs with a forward voltage of 2.1V, and targeted 6 mA per LED, giving a 200 ohm resistor value. Selected 200 ohm resistors with 5 percent tolerance. Checked the STM32L432's datasheet absolute maximum ratings and confirmed total combined IO current draw must stay under 100 mA. Three LEDs assigned as: power indicator, channel activity indicator, and recording/streaming indicator.

## USB Power Input Circuit

Built the power delivery path from USB to the 3.3V rail. A USB connector, wired for USB 2.0 since that is the only USB level the STM32L432 supports, feeds a linear voltage regulator with 3.3V output and 1A current capability, stepping down from the 5V USB supply. Added decoupling capacitors on the regulator's input and output (4.7 microfarad each) per its datasheet, and pulled the enable pin up to 5V so the regulator turns on as soon as USB is connected.

Wired the USB connector: VBUS to 5V input, ground to ground, DP/DN to the MCU's USB data lines. Pulled down CC1 and CC2 with 5.1k resistors, since the device is a USB peripheral. A peripheral terminates CC pins with pull downs, while a host uses pull ups. Left SBU1 and SBU2 unconnected, since those are used only for alternate mode functionality, such as DisplayPort, that this design does not need.

## LCD Wiring

Wired the LCD using the four wire SPI interface per the vendor's interfacing guide. Added two new GPIO signals not previously assigned, LCD reset and LCD A0 (data/instruction select), assigned to PA3 and PA2. Deviated slightly from the vendor's reference design by adding pull up resistors on some LCD signals that the guide did not include, as a precaution, since the module itself did not appear to have them onboard. Unused, static LCD signal lines share pull up resistors with each other, which is acceptable since they do not change state.

## Push Button Wiring

Wired each push button as a switch to ground, with a 10k pull up resistor to 3.3V on the signal line. This means the signal reads high (3.3V) when the button is not pressed, and reads low (0V, ground) when pressed. Since these pins are configured as interrupts, a button press triggers an edge interrupt in firmware.

## Signal Input Stage

Added two female BNC connectors, one per channel, for probing input signals. The connector's internal pin carries the signal, wired to the ADC input, and the external shielding is wired to ground.

## Debug and Programmer Header

Added a 14 pin header to interface with an ST-Link V3 Mini programmer over SWD. Wired VTREF to the 3.3V rail, ground pins to ground, RST to the MCU's NRST pin, UART TX/RX to the debug UART, and SWDIO/SWCLK to the corresponding MCU pins. Added a 100 ohm resistor on the header's presence detection pin per the ST-Link spec. Connected the MCU's boot pin to ground, since this design does not need alternate boot configurations.

With this, the schematic is functionally complete.
