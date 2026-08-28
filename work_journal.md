# Work journal for the MLAB_MCU test PCB

## TODO

- Add links for each component for automatic BOM generation

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

# 2026-08-03
## Ernesto pastebejimai
1. Gnd kontaktu maitinimo saltiniui is lab bench
2. sroves stiprintuvas suntams
3. Clock apdaryt 3 pos header kad pasirinkt tarp sma ir CMOS osc ir dar papildoma 2pos kad prijungti fpga
4. perkelti gpio jungima i fpga kad butu uz nuoseklio varzos
5. 

# Klausimai
1. Ar ant clk net šalia socket'o reikia dėti pF kondensatorių?
2. Kiek vietos palikt aplink SMA jungteles?


# BOM
- Socket: https://lt.farnell.com/3m/232-5205-01/test-socket-qfn-32pos-0-5mm-th/dp/2668401?cfm=true
- CMOS Clock oscillator:
https://eu.mouser.com/en/ProductDetail/Microchip-Technology/DSC1101DL5-020.0000?qs=Gd3Cm49KlLOJ7fVrdqTCCA%3D%3D
- LDO 3V3: https://www.mouser.lt/en/ProductDetail/Texas-Instruments/TPS7A2033PDBVR?qs=hd1VzrDQEGjk%2FOBnRfKB4A%3D%3D
- LDO 1V2: https://www.mouser.lt/en/ProductDetail/Texas-Instruments/TPS7A2012PDBVRG4?qs=vOcB1WHNNXISX%252BdA%2FhaD3A%3D%3D
- EEPROM: https://www.mouser.lt/en/ProductDetail/Microchip-Technology/24LC512-I-P?qs=JmwSjbzn2OL8zYUOM6epRw%3D%3D
- Current-sense amp (4 channel):
    - 20 V/V: https://www.mouser.lt/en/ProductDetail/Texas-Instruments/INA4180A1IPWR?qs=f9yNj16SXrIc8%252Bkjpx1s2w%3D%3D
    - 50 V/V: https://www.mouser.lt/en/ProductDetail/Texas-Instruments/INA4180A2IPWR?qs=chTDxNqvsykNfIKY08v%252BVA%3D%3D
    - 100 V/V: https://www.mouser.lt/en/ProductDetail/Texas-Instruments/INA4180A3IPWR?qs=chTDxNqvsynaXF8xKMZ2OQ%3D%3D
    - SS34
    - SMAJ6.0A
    - 