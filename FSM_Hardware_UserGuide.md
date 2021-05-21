# FRAMOS Sensor Module Ecosystem - Hardware User Guide

---

## Version Info

Version: 1.1.1

Date: 2020-12-01

---
## Getting Started

Getting started with [FRAMOS Sensor Module Ecosystem](https://www.framos.com/en/fsm-startup) and FSM_Jetson-*_UserGuide.

This document contains hardware functionality description for FRAMOS Sensor Module Ecosystem.

---

## FRAMOS Processor Adapter Boards

 This chapter contains hardware functionality description of FPA boards for FRAMOS Sensor Module Ecosystem. FPA boards are platform specific and supported platforms will be stated for each FPA.


### FPA-4.A/TXA

Supported platforms: Jetson AGX, Jetson TX2

---

FPA-4.A/TXA is processor adapter board for connecting processing board with FSA/FSM modules through four connectors marked J5, J6, J7 and J8. 

Processor board clock signals CAM0_MCLK and CAM1_MCLK are routed through four clock drivers to connectors for better integrity of the clock signal. CAM0_PWDN and CAM0_RST signals are routed parallel to all connectors. CAM_I2C_SCL and CAM_I2C_SDA are routed parallel to EEPROM whose functionality is primarily to utilize functionality of L4T Plugin manager and I2C eight channel multiplexer, whose pairs of channels are routed to specific connector (MUX-ch0/ch1 are connected to J5, MUX-ch2/ch3 are connected to J6, MUX-ch4/ch5 are connected to J7, MUX-ch6/ch7 are connected to J8). 

Additional functionality of FPA is available through its three switches. SW1 have the option of interconnecting XVS and XHS triggering signals. SW2 have the option of interconnecting XTRIG signals and forwarding CAM2_MCLK04 and SYS_PW_EN processor board signals. SW3 controls write protect pin and power for the on board EEPROM with its default 0x54 address.

Descriptions of the switches when switched to ON state, with its recommended state in parenthesis:
- SW1
    - pin 1 (OFF) - connects XVS signal from J8 to interconnecting XVS line
    - pin 2 (OFF) - connects XHS signal from J8 to interconnecting XHS line
    - pin 3 (OFF) - connects XVS signal from J7 to interconnecting XVS line
    - pin 4 (OFF) - connects XHS signal from J7 to interconnecting XHS line
    - pin 5 (OFF) - connects XVS signal from J5 to interconnecting XVS line
    - pin 6 (OFF) - connects XHS signal from J5 to interconnecting XHS line
    - pin 7 (OFF) - connects XVS signal from J6 to interconnecting XVS line
    - pin 8 (OFF) - connects XHS signal from J6 to interconnecting XHS line
    
Test point for the interconnecting XVS line is marked as TP3

Test point for the interconnecting XHS line is marked as TP4

- SW2
    - pin 1 (OFF) - connects CAM2_MCLK04 signal from processor board to MCLK2 pin on all connectors
    - pin 2 (OFF) - connects SYS_PW_EN signal from processor board to SYS_PW_EN pin on all connectors
    - pin 3 (OFF) - NOT IN USE
    - pin 4 (OFF) - NOT IN USE
    - pin 5 (OFF) - connects XTRIG signal from J7 to interconnecting XTRIG line
    - pin 6 (OFF) - connects XTRIG signal from J8 to interconnecting XTRIG line
    - pin 7 (OFF) - connects XTRIG signal from J6 to interconnecting XTRIG line
    - pin 8 (OFF) - connects XTRIG signal from J5 to interconnecting XTRIG line
    
Test point for the interconnecting XTRIG line is marked as TP5

- SW3
    - pin 1 (ON) - supplies power to on board EEPROM [ON/OFF -> HI/LOW]
    - pin 2 (OFF) - write protect for on board EEPROM [ON/OFF -> HI/LOW]

Description of main test points on FPA module:

- TP 66 - auxiliary voltage supply, routed parallel to all connectors
- TP 67 - 1.2V processors board voltage supply (controllable)
- TP 68 - 1.8V processors board voltage supply (controllable)
- TP 69 - 2.8V processors board voltage supply (controllable)
- TP 70 - 1.8V generated voltage supply
- TP 71 - 3.8V generated voltage supply
- TP 72 - 5.0V / 3.3V processors board voltage supply (always on)
- GND - GND

### FPA-2.A/96B

Supported platforms: FPA is compliant with 96Boards specification

---

FPA-2.A/96B is processor adapter board designed per 96Boards specification for connecting processing board with FSA/FSM modules through two connectors marked J3 and J4.

Main function of processor adapter is generating specific power supply for connected FSA/FSM boards. In addition, processor adapter board offers additional functionality, available through 8 pin DIP switch. Clock signal for image sensors are expected from sensor adapter board. Reason for this is that DragonBoard 410c board uses all two available programmable clock interfaces for CSI0_MCLK and CSI1_MCLK interface. Each connector has its own I2C buses connected to High speed connector and separate reset and power enable pins for each connector.

Tables below will give you pinout description of required signals for sensor to work. 


|J3 signal                  |DragonBoard 410c - 96 board signal         |
|---------------------------|-------------------------------------------|
|CAM0_RST_0                 |LS_EXP_GPIO_I (APQ GPIO_35) (CSI0_RST)     |
|CAM0_PW_EN_0               |LS_EXP_GPIO_J (APQ GPIO_34) (CSI0_PWDN)    |
|CAM0_I2C_0_SCL(SPI_SCK)    |I2C2_SCL (APQ GPIO_30)                     |
|CAM0_I2C_0_SDA(SPI_MOSI)   |I2C2_SDA (APQ GPIO_29)                     |   
|CAM0_GPIO0(XMASTER0)       |96B_GPIO_A                                 |

|J4 signal                  |DragonBoard 410c - 96 board signal         |
|---------------------------|-------------------------------------------|
|CAM1_RST_0                 |LS_EXP_GPIO_K (APQ GPIO_28) (CSI1_RST)     |
|CAM1_PW_EN_0               |LS_EXP_GPIO_L (APQ GPIO_33) (CSI1_PWDN)    |
|CAM1_I2C_0_SCL(SPI_SCK)    |I2C3_SCL (APQ GPIO_15)                     |
|CAM1_I2C_0_SDA(SPI_MOSI)   |I2C3_SDA (APQ GPIO_14)                     |   
|CAM1_GPIO0(XMASTER0)       |96B_GPIO_C                                 |
    
    

Additional functionality of FPA is available through its 8 pin DIP switch. From processor board side input pins are connected to GPIO pins where there is a possibility to trigger sensors XVS/XHS/XTRIG/GPIO pins accordingly.

Descriptions of the switches when switched to ON state, with its recommended state in parenthesis:
- DIP Switch:
    - pin 1 (OFF) - connects CAM0 XVS signal to processors board LS_EXP_GPIO_B (APQ GPIO_12) (TS_RST_N) signal
    - pin 2 (OFF) - connects CAM0 XHS signal to processors board LS_EXP_GPIO_E (APQ GPIO_115) (GYRO_ACCL_INT_N) signal
    - pin 3 (OFF) - connects CAM0 XTRIG signal to processors board LS_EXP_GPIO_F (PM_MPP_4) (DSI_BLCTRL) signal
    - pin 4 (OFF) - connects CAM0 GPIO signal to processors board LS_EXP_GPIO_D (APQ GPIO_69) (MAG_INT) signal
    - pin 5 (OFF) - connects CAM1 XVS signal to processors board LS_EXP_GPIO_B (APQ GPIO_12) (TS_RST_N) signal
    - pin 6 (OFF) - connects CAM1 XHS signal to processors board LS_EXP_GPIO_E (APQ GPIO_115) (GYRO_ACCL_INT_N) signal
    - pin 7 (OFF) - connects CAM1 XTRIG signal to processors board LS_EXP_GPIO_F (PM_MPP_4) (DSI_BLCTRL) signal
    - pin 8 (OFF) - connects CAM1 GPIO signal to processors board LS_EXP_GPIO_D (APQ GPIO_69) (MAG_INT) signal
    

Description of main test points on FPA module:

- TP 49 - 1.8V generated voltage supply
- TP 50 - 3.8V generated voltage supply
- TP 51 - 5.0V processors board voltage supply 
- TP 52, TP 53 - GND

### FPA-A/NVN

Supported platforms: Jetson Nano Developer Kit, Jetson Xavier-NX Developer Kit

---

FPA-A/NVN is processor adapter board for connecting processing board with FSA/FSM modules through available connectors. Main functionality of FPA-A/NVN is to bridge interfaces of camera connector on Jetson Nano Developer Kit or Jetson Xavier-NX Developer Kit to FRAMOS 60-pin camera interface standard.

Processor board clock signals MCLK0 and GPIO CAM_PWDN are routed through clock driver to connector for better integrity of the signal. I2C_SCL_IN and I2C_SDA_IN are routed parallel to EEPROM that enables user to store board specific information.

Description of main test points on FPA module makred J2:

- pin 1 (marked with white dot) - 1.8V generated voltage supply
- pin 2 - GND
- pin 3 - CAM0_GPIO1(XVS0)
- pin 4 - CAM0_GPIO2(XHS0)
- pin 5 - CAM0_GPIO11(FSTROBE)
- pin 6 - CAM0_PW_EN_1
- pin 7 - CAM0_PW_EN_0

## FRAMOS Sensor Adapter Boards

 This chapter contains hardware functionality description of FSA boards for FRAMOS Sensor Module Ecosystem. All FSA module interface is compliant with FRAMOS standard.

---

### FSA-FTxx/A

FSA-FTxx/A is function board designated to adjust power supplies, power-up sequence, clock and control signals for image sensor to work in application conditions as described in sensors datasheet.

FSA-FTxx/A have on board EEPROM with 0x55 as its default address, two switches (SW1 – Rotary switch, SW2 – DIP switch), internal oscillator generating clock, linear voltage regulators and power sequencer for specific image sensor. Linear voltage regulators generate analog, interface and digital power supplies specified for specific image sensor and the sequence is controlled with power sequencer.

FSA-FTxx/A is dependent on FPA-4.A/TXA adapters power supplies for generating necessary power supplies with appropriate power-up sequence, clock and control signals for FSM.

Additional functionality of FSA is available through its two switches. One is DIP switch with four positions controlling the on board EEPROM power supply, image sensor slave address and XMASTER pin if available. Other is rotary switch with three positions controlling the clock source for the image sensors input clock.

Descriptions of the switches state, with its recommended state in round parenthesis:
- DIP Switch 
    - pin 1 (ON) - supplies power to on board EEPROM
    - pin 2 (OFF) - controlling the logic level on image sensor SLAMODE1 pin [ON/OFF -> HI/LOW]
    - pin 3 (OFF) - controlling the logic level on image sensor SLAMODE2 pin [ON/OFF -> HI/LOW]
    - pin 4 (OFF) - controlling the logic level on FSA male connector XMASTER pin [ON/OFF -> HI/LOW] 
    
- Rotary Switch
    - state 1 - connects on board oscillator clock (recommended)
    - state 2 - connects CAM0_MCLK platform board clock
    - state 3 - connects CAM1_MCLK platform board clock

Description of main test points on FSA module:

- TP 3 - image sensors input clock signal
- TP 4 - image sensors reset signal
- TP 5 - image sensors slave mode 1 select signal
- TP 6 - image sensors slave mode 2 select signal
- TP 7 - image sensors XMASTER signal, slave/master mode select
- TP 9 - 3.3V input voltage supply
- TP 10 - 1.8V input voltage supply
- TP 11 - image sensors analog voltage supply
- TP 12 - image sensors interface voltage supply
- TP 13 - image sensors digital voltage supply
- TP 8,17 - GND

### FSA-FTxx/A-00G

FSA-FTxx/A-00G sensor adapter board functionality is basically the same as FSA-FTxx/A, adjusting power supplies, power-up sequence, clock and control signals for image sensor to work in application conditions as described in sensors Datasheet but the main function that is added to the sensor adapter board is to bridge LVDS/SLVS to MIPI CSI-2 interface.

FSA-FTxx/A-00G have two switches (SW1 – DIP switch, SW2 – Rotary switch), internal oscillator generating clock, linear voltage regulators, I2C bus voltage-level translator, 8-pin header connector, flash memory and Lattice CrossLink FPGA chip. On the board there are two groups of Linear voltage regulators. One group of regulators is designated for generating power supplies for CrossLink while the other group generates analog, interface and digital power supplies specified for specific image sensor where the sequence is controlled within the CrossLink logic.

FSA-FTxx/A-00G is dependent on FPA-4.A/TXA adapters power supplies for generating necessary power supplies, clock and control signals for CrossLink and FSM.

Additional functionality of FSA is available through its two switches. One is DIP switch with four positions controlling the flash/boot options for CrossLink FPGA. Other is rotary switch with three positions controlling the clock source for the CrossLink input clock.


Descriptions of the switches state, with its recommended state in round parenthesis, and 8-pin header pinout:


- DIP Switch 
    - pin 1 (ON) - connects I2C SDA line to CrossLink J1 pin(PB53/SPI_SCK/MCK/SDA)
    - pin 2 (OFF) - connects SPI EE_SCK line to CrossLink J1 pin(PB53/SPI_SCK/MCK/SDA)
    - pin 3 (ON) - connects I2C SCL line to CrossLink H1 pin(PB52/SPI_SS/CNS/SCL)
    - pin 4 (OFF) - connects SPI EE_SS line to CrossLink H1 pin(PB52/SPI_SS/CNS/SCL)
    
There are several options available for flash/boot of CrossLink FPGA and they are dependent on DIP switch positions. 

1. I2C CrossLink SRAM configuration (DIP pins 1 & 3 are ON, 2 & 4 are OFF)

To configure CrossLink SRAM configuration over I2C, DIP pins 1 & 3 must be in ON position and pins 2 & 4 in OFF position. This type of configuration is on-board available without any others HW components. 

2. SPI CrossLink SRAM configuration (DIP pins 2 & 4 are ON, 1 & 3 are OFF)

To configure CrossLink SRAM configuration over SPI, DIP pins 2 & 4 must be in ON position and pins 1 & 3 in OFF position. This type of configuration is available with external programmer through FSA 8-pin header.

3. Boot from on board Flash memory over SPI (DIP pins 2 & 4 are ON, 1 & 3 are OFF)

To boot CrossLink over SPI, (DIP pins 2 & 4 must be in ON position and pins 1 & 3 in OFF position). To boot from flash memory, bit file must be flashed to flash memory over Lattice Diamond Programmer. Flash programming is available with external HW programmer through FSA 8-pin header or over EE_* pins on 60pin connector.

 **Note**: DIP pins 1 & 2 and 3 & 4 must be set opposite!

 **Note**: When booting from flash memory, programmer must be disconnected or the CrossLink may not boot.
    
    
- Rotary Switch
    - state 1 - connects on board oscillators clock  (recommended)
    - state 2 - connects CAM0_MCLK platform board clock
    - state 3 - connects CAM1_MCLK platform board clock

- 8-pin header pinout

    - pin 1 - VCCIO_0  (marked with number 1 on the board)
    - pin 2 - CDONE
    - pin 3 - EE_SCK
    - pin 4 - EE_MISO
    - pin 5 - EE_SS
    - pin 6 - EE_MOSI
    - pin 7 - CRESET_B
    - pin 8 - GND
    
---


## Revision History

| Version | Date           | Description                                                      |
| ------- | -------------- | -----------------------------------------------------------------|
| 1.1.1   | 2020-12-01     | Minor bug fixes and improvements                                 |
| 1.1.0   | 2020-07-17     | Added new FRAMOS Processor Adapter Board: FPA-A/NVN              |
| 1.0.0   | 2020-03-19     | Initial release                                                  |

---