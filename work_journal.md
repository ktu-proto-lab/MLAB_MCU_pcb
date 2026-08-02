# Work journal for the MLAB_MCU test PCB

## TODO

- Use README.md to choose components and create the schematic

## 2026-07-19

- Create schematic + footprint for the ordered socket
- Start working on the schematic

Found socket for IC. TODO: Verify the test IC pin ordering when we receive the physical socket.

Chose TPS7A2012 and TPS7A2033 LDOs. Will put LDOs in parallel not efficient but for a bringup board it is fine. More simple design easier to customize for testing.

Chose DSC1101DL5-020.0000 20MHz oscillator - cheapest in mouser and good availability. Can be supplied from 3.3V.

Put a header and a screw terminal for input power to LDOs.

# 2026-07-28
CMOS Clock oscillator:
https://eu.mouser.com/en/ProductDetail/Microchip-Technology/DSC1101DL5-020.0000?qs=Gd3Cm49KlLOJ7fVrdqTCCA%3D%3D


# 2026-08-02
## TODO
- Add switch for rst
- Add EEPROM socket (DIP-8)
