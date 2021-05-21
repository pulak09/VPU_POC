# FRAMOS Sensor Module Ecosystem - Driver Binary User Guide

---

## Version Info

Version: 1.7.0

Date: 2020-12-01


## Getting Started

Getting started with [FRAMOS Sensor Module Ecosystem](https://www.framos.com/en/fsm-startup).

This chapter contains driver-binary installation instructions for FRAMOS Sensor Module Ecosystem on Nvidia Jetson Xavier platform.

---

## Prerequisites

- Installed Ubuntu 16.04 OS on host machine
- Nvidia Jetson Xavier platform (Target system) connected via USB to the host machine
- Nvidia Jetson Xavier platform (Target system) connected to same network as the host machine
- Nvidia Jetson Xavier platform (Target system) connected to Internet

---


## Host system preparation and flashing Target system

  1. Download Nvidia JetPack 4.4.1 and get FSM_Drivers_Jetson-Xavier*.tar.gz
  2. Install Nvidia JetPack on host machine
  3. Update installed Nvidia JetPack with FSM_Drivers_Jetson-Xavier files
  4. Flash Jetson Xavier

### 1. Download Nvidia JetPack 4.4.1 and get FSM_Drivers_Jetson-Xavier*.tar.gz

- Download NVIDIA SDK Manager and install Jetpack 4.4.1 for Xavier platform. It can be downloaded from [SDK Manager](https://developer.nvidia.com/embedded/downloads#?search=SDK) with its [SDK Manager Download Instructions](https://docs.nvidia.com/sdk-manager/download-run-sdkm/index.html). Log in required.

- Extract FSM_Drivers_Jetson-Xavier*.tar.gz archive to Download directory
    ```
    cd ~/Downloads/
    sudo tar xvzfp FSM_Drivers_Jetson-Xavier*.tar.gz
    ```
    
### 2. Install the Nvidia JetPack on host machine

-  To install JetPack on your host machine please follow these instructions [SDK Manager Install Instructions](https://docs.nvidia.com/sdk-manager/install-with-sdkm-jetson/index.html)
- **Note**: Archive scripts and user guide follows default install path of JetPack
- **Note**: When  window appears "SDK Manager is about to flash your Jetson ...", press Skip.


### 3. Update default Nvidia JetPack boot system files

- **First export Linux_for_Tegra full path(installation path).** Example:
    ```
    export L4T_DIR=<jetpack>/Linux_for_Tegra
    ```
        
Automatic patch script is provided to patch kernel files:

```
cd ~/Downloads/FSM_Drivers_Jetson-Xavier/bin/
sudo -E ./patch.sh 
```

Steps taken by automatic script are next:
a) Patching binary Kernel Image

b) Patching binary Device Tree

c) Patching partition configuration file

d) Patching binary Kernel headers and Kernel modules archive



### 4. Flash Jetson Xavier

1. Set Nvidia Jetson Xavier platform in force USB Recovery Mode
    - **Note**: See Nvidia [Development Guide ](https://docs.nvidia.com/jetson/l4t/index.html), chapter "Flashing and Booting the Target Device"
2. To apply all changes to Target system, flash script (full - device tree, image, c-boot, rootfs) must be executed. This operation will take approx 30min:

    ```
    cd $L4T_DIR
    sudo ./flash.sh jetson-xavier mmcblk0p1
    ```
  
- **Note**: When flash process finishes and system boots up, you will be asked to create a user with other settings as well. User name and password will be needed for transferring files from host to target.

---

## First time mounting/Replacing ISB on Target board

**CAUTION:** Not following this procedure may cause permanent damage to the image sensor module!    

Prerequisites:

- Copy the "write_eeprom.sh" file from ~/FSM_Software-Package/bin/ folder to the home folder:
    ```
    cd ~/FSM_Software-Package/bin/
    cp write_eeprom.sh ~/.
    ```   
Procedure:
1. Power off the target system and unplug DC power

2. Connect the FSM to the appropriate FSA module, the FSA to an appropriate FPA module and connect the assembly to the Jetson Xavier.

    **Note:** **If FSM does have EEPROM on the board** move DIP switch pin 1 to OFF position on FSA module, else set it to ON position (if DIP switch is available on FSA board).
    
    **Warning:** Wrong FPAs / FSAs can lead to permanent damage of the FSM! Use only compatible FPA / FSA listed in the Datasheet of the FSM!
    
3. Replug DC power and power on the Target

4. Write the Module ID to EEPROM of the FPA module. This is done by execute "write_eeprom" script with the correct parameters. A list of compatible Module ID, I2C BUS ID and I2C EEPROM Address can be found below.
  
    |FPA Module       |EEPROM Available|MUX Available|EEPROM ID        |I2C BUS ID  |I2C EEPROM address|I2C MUX address|
    |-----------------|----------------|-------------|-----------------|------------|------------------|---------------|
    |FPA-TX2-FT1      |Yes             |Yes          |framos-adapter   |2           |0x54              |0x70           |
    |FPA-TX2-FT2      |No              |No           |-                |-           |-                 |-              |
    |FPA-4.A/TXA      |Yes             |Yes          |framos-adapter   |2           |0x54              |0x70           |
    
    *Example.* 
    On target system write Module ID to FPA-4.A/TXA EEPROM:
    ```
    cd
    sudo ./write_eeprom.sh 2 0x54 framos-adapter 0x70
    ```
    
    EEPROM content of the FPA module EEPROM can be checked with:
    ```
    sudo i2cdump -y 2 0x54
    ```

- **Note**: Some FPA modules do not have an EEPROM, in which case the EEPROM write is not possible.
- **Note**: Content of EEPROM at register 0x14 must be 0xcc
- **Note**: Module ID starts at register 0x15 of EEPROM
- **Note**: The Module ID must be null terminated (0x00)
- **Note**: If available, multiplexor's I2C address must be written to register 0x59 and followed with value 1 at register 0x5a

5. Compatible sensor module ID has to be written. Sensor module ID consists of:
    - Key word: "framos"
    - Sensor ID: "imx283"
    - **(optional)** Sensor type:
        - MONO -> "m"
        - COLOR -> "c"
    - Multiplexor channel sensor lies on: 
        - if sensor is on J5 -> "0"
        - if sensor is on J6 -> "2"
    - **(optional)** CSI lane configuration, more in "Configuring MIPI CSI-2 lanes of IS on Target board" chapter 
  
- **Note**: Sensor module ID parts are separated by dash character "-"
- **Note**: Some sensors take additional parameter for MONO sensor or COLOR sensor because some features differ based on that distinction. In table below there is example of IMX296 sensor that takes the additional parameter. MONO sensors are tagged with "m", while COLOR sensors are tagged with additional parameter "c". If this parameter is left empty driver will load with its full feature version. 
- **Note**: Adding Sensor type parameter for sensors that do not take this parameter will result in empty driver probe.


Writing Sensor module ID is done by executing `write_eeprom` script with right parameters and the list of compatible Module ID and I2C BUS ID can be found below. Right I2C BUS ID on the FPA-4.A/TXA depends on the connector (J5, J6, J7 or J8) where FSA/FPA is connected. FPA-TX2-FT1 connector is associated with J5 connector of the FPA-4.A/TXA module but for transparency they are separated in the table below.


|Image Sensor Module | Driver Name | EEPROM ID                           | FPA-4.A/TXA I2C BUS ID [MUX-CH]- J5, J6, J7, J8 |
|--------------------|-------------|-------------------------------------|--------------------------------------------------|
|FSM-AR0144          |ar0144       |framos-ar0144-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-AR0521          |ar0521       |framos-ar0521-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-AR1335          |ar1335       |framos-ar1335-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-HDPY230         |hdpy230      |framos-hdpy230-[MUX-CH]              |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX264          |imx264       |framos-imx264-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX283          |imx283       |framos-imx283-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX290          |imx290       |framos-imx290-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX462          |imx290       |framos-imx462-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX296          |imx296       |framos-imx296-[MONO/COLOR]-[MUX-CH] |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX297          |imx297       |framos-imx297-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX304          |imx304       |framos-imx304-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX327          |imx327       |framos-imx327-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX334          |imx334       |framos-imx334-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX335          |imx335       |framos-imx335-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX412          |imx412       |framos-imx412-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX415          |imx415       |framos-imx415-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX464          |imx464       |framos-imx464-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX477          |imx477       |framos-imx477-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX485          |imx485       |framos-imx485-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX530          |imx530       |framos-imx530-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX565          |imx565       |framos-imx565-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |
|FSM-IMX577          |imx577       |framos-imx577-[MUX-CH]               |30 [0], 32 [2], 34 [4], 36 [6]                    |

**Different procedure must be followed if FSM does have EEPROM on the board. If FSM does have EEPROM on the board refer to procedure under 5.a) else refer to procedure under 5.b).**
    
a) **EEPROM on the FSM** 

Prerequisites:
- 1st pin of the DIP switch on FSA board set to OFF position. 

 **Note**: Some FSA modules do not have the DIP switch to enable/disable the on board EEPROM. There, on board EEPROM is enabled by default and care must be taken that only one EEPROM is written with Module ID and the other one is erased, or the same Module ID is written to FSA EEPROM and FSM EEPROM.

Procedure:
- execute `write_eeprom` script with right parameters

*Example.* 

FSM-IMX283 with EEPROM on board, FSA-FT12/A and FPA-4.A/TXA are connected to J5 connector of the target system.
On target system write Module ID to FSM-IMX283 EEPROM:
```
cd
sudo ./write_eeprom.sh 30 0x56 framos-imx283-0
```

- erase the EEPROM content if FSA board doesn't have DIP switch available. 

*Example.* 

If the FSA board doesn't have the DIP switch, erase its EEPROM content. FSM-IMX283 with EEPROM on board, FSA-FT12/A and FPA-4.A/TXA are connected to J5 connector of the target system.

On target system erase EEPROM content of FSA board:
```
cd
sudo ./write_eeprom.sh 30 0x55
```

EEPROM content of the FSM module EEPROM can be checked with:
```
sudo i2cdump -y 30 0x56
```
    
b) **EEPROM on the FSA**

Prerequisites:
- 1st pin of the DIP switch on FSA board set to ON position.  
 
**Note**: Some FSA modules do not have the DIP switch to enable/disable the on board EEPROM. There on board EEPROM is enabled by default.
 
Procedure:
- execute `write_eeprom` script with right parameters

*Example.* 

FSM-IMX283 module, FSA-FT12/A and FPA-4.A/TXA are connected to J5 connector of the target system.
On target system write Module ID to FSA-FT12/A EEPROM:
```
cd
sudo ./write_eeprom.sh 30 0x55 framos-imx283-0
```

EEPROM content of the FSA module EEPROM can be checked with:
```
sudo i2cdump -y 30 0x55
```

- **Note**: Jetson is using a feature of the Plugin Manager (for Device Tree) that loads drivers depending on the content of EEPROM.
- **Note**: The content of EEPROM at register 0x14 must be 0xcc
- **Note**: The Module ID starts at register 0x15 of the EEPROM
- **Note**: The Module ID must be null terminated (0x00)
    
6. Power off and Power on target board

7. Make sure that the driver is loaded. The driver will automatically load during boot up.
Check with dmesg command if the driver is loaded properly.
For example, for Sony IMX283:

    ```
    dmesg | grep "Detected imx283 sensor"
    ```
    The output message appears as shown below:
    ```
    imx283 30-001a: Detected imx283 sensor at address 0x1a.
    ```

---
## Configuring MIPI CSI-2 lanes of IS on Target board

Some image sensors have the option of choosing different numbers of CSI-2 lanes and this is available through extended sensor EEPROM ID configuration.

FPA module can support maximum of 4 CSI-2 lanes on J5 connector, 4 CSI-2 lanes on J6 connector, 2 CSI-2 lanes on J7 connector and 2 CSI-2 lanes on J8 connector. 

Below is a table of supported CSI-2 lanes configuration for every sensor.

|Image Sensor Module   |Supported CSI-2 lanes |
|----------------------|--------------------|
|FSM-AR0144            |         2          |
|FSM-AR0521            |       2 & 4        |
|FSM-AR1335            |       2 & 4        |
|FSM-HDPY230           |       2 & 4        |
|FSM-IMX264            |         4          |
|FSM-IMX283            |         4          |
|FSM-IMX290            |       2 & 4        |
|FSM-IMX462            |       2 & 4        |
|FSM-IMX296            |         1          |
|FSM-IMX297            |         1          |
|FSM-IMX304            |         4          |
|FSM-IMX327            |       2 & 4        |
|FSM-IMX334            |         4          |
|FSM-IMX335            |       2 & 4        |
|FSM-IMX412            |       2 & 4        | 
|FSM-IMX415            |       2 & 4        |
|FSM-IMX464            |       2 & 4        |
|FSM-IMX477            |       2 & 4        |
|FSM-IMX485            |       2 & 4        |
|FSM-IMX530            |         4          |
|FSM-IMX565            |       2 & 4        |
|FSM-IMX577            |       2 & 4        |
    
There are several possibilities on how to configure every connector:

    1. Connector has available 4 CSI-2 lanes
    2. Connector has available 2 CSI-2 lanes
    
To configure CSI-2 lanes, description is added on existing sensor EEPROM ID ("framos-r"). If CSI-2 configuration is left empty maximum available CSI-2 lanes are set. Below are examples on how to configure CSI-2 lanes.

    
Configuration is done by executing `write_eeprom` script with right parameters. Configuration of imx577 on connector J5 with 2 available CSI-2 lanes is shown in the example. Care must be taken to write full EEPROM ID to FSA/FSM EEPROM depending on the HW configuration (explained in First time mounting/Replacing ISB on Target board chapter)


a) **EEPROM on the FSM**

    ```
    cd
    sudo ./write_eeprom.sh 30 0x56 framos-imx577-0-2
    ```
    
    EEPROM content of the FSA module EEPROM can be checked with:
    ```
    sudo i2cdump -y 30 0x56
    ```

b) **EEPROM on the FSA**

    ```
    cd
    sudo ./write_eeprom.sh 30 0x55 framos-imx577-0-2
    ```

    EEPROM content of the FSA module EEPROM can be checked with:
    ```
    sudo i2cdump -y 30 0x55
    ```

---


## Working with LVDS/SLVS sensor

This chapter will describe possibilities and procedures that must be followed to get the stream from the LVDS/SLVS interface sensors.

Working with LVDS/SLVS sensors on Nvidia Jetson Xavier reveals a problem of interface incompatibility. Solution to the problem is converting the interfaces which is realized with Lattice IP LVDS/SLVS to MIPI CSI-2 bridge on CrossLink FPGA. 

### CrossLink configuration

Prerequisites:
- "firmware" folder copied to target system
    - Copy "firmware" folder to home folder on target system (using ssh or your preferred method):
    
        ```
        scp -r ~/Downloads/FSM_Drivers_Jetson-Xavier/firmware nvidia@10.32.66.6:.
        ```
        
- **Note**: Use firmware from the package itself, not from an earlier or later version of the package. The drivers and firmware are only compatible if they come from the same version of the package!

Firmwares are grouped per supported LVDS/SLVS channel configuration and pixel format. Inside that group are firmwares with specified data rate output. In the table bellow all supported LVDS/SLVS channel configurations on specific pixel format and supported output data rate for every supported Image sensor are listed.

| Image Sensor Module  |Supported LVDS/SLVS channels|Pixel format | Suported data rate |
|----------------------|-----------------------------|-------------|--------------------|
|FSM-IMX264            |4                            |RAW12        |594                 |
|FSM-IMX304            |8                            |RAW10, RAW12 |594                 |
|FSM-IMX530            |8                            |RAW10, RAW12 |594                 |

For example if working with FSM-IMX264 only one firmware is supported and it is lvds2mipi_4ch_RAW12_594_fw*.

In order to start the stream, CrossLink must be configured. There are few options on how to configure CrossLink, one being volatile configuration where the firmware is stored on Crosslinks SRAM, other being non-volatile where the configuration is stored on flash memory on the FSA board. 

I2C bus CrossLink configuration is convenient way for developing process and fits the need of quickly testing different options available by sensor. If one firmware fulfills all application requests, programming on board Flash memory is recommended. Procedure for every possibility is explained next working with FSM-IMX264. 

### 1. I2C bus SRAM configuration 
Configuration of CrossLink SRAM over I2C bus (if step 2 is done, skip to step 5): 
1. Power-off the Jetson
2. On FSA module put the pins 1 & 3 on DIP switch to ON position, and pins 2 & 4 to OFF position
3. Connect the FSM->FSA->CAB->FPA
4. Power-on the Jetson
5. Flash the firmware over I2C bus using the user space application with appropriate parameters

    Parameters needed to run the application:
        
    - 1st - CrossLink I2C bus ID (30 for the connector J5, for more information check the "Replacing ISB on Target board" chapter)
    - 2nd - CRESET_B, CrossLink reset GPIO number (445, CAM1_PWDN signal on Jetson Camera Connector)
    - 3rd - Algorithm or Data file name location
    
    ```
    cd ~/firmware/bin/
    sudo ./crosslink_configurator 30 445 ../lvds2mipi_4ch_RAW12/lvds2mipi_4ch_RAW12_594_fw_v1.2.1.0.ied
    ```
    
- **Note**: DIP pins 1 & 2 must be set opposite and pins 3 & 4 must be set opposite!
 
 
#### SRAM configuration of multiple CrossLink devices over I2C bus

With current hardware, user can't configure SRAM of multiple devices one by one because CRESET_B pin on every FSA board is connected to the same GPIO pin on the FPA board. Configuring SRAM on CrossLink devices one by one will result in good configuration of the last configured CrossLink and all previous CrossLink devices will end up in empty state, without any configuration in SRAM.

At the time, I2C bus SRAM multiple configuration can be done configuring SRAM on CrossLink devices simultaneously using the broadcasting feature of the multiplexer. To do so, CrossLink I2C bus ID must be set to bus 37 which is configured as broadcasting channel on multiplexer.

```
cd ~/firmware/bin/
sudo ./crosslink_configurator 37 445 ../lvds2mipi_4ch_RAW12/lvds2mipi_4ch_RAW12_594_fw_v1.2.1.0.ied
```

- **Note**: Only one firmware configuration can be used for multiple configuration.

### 2. SPI SRAM configuration
Configuration of CrossLink SRAM over SPI (if step 2 is done, skip to step 5): 
1. Power-off the Jetson
2. On FSA module put the pins 2 & 4 on DIP switch to ON position, and pins 1 & 3 to OFF position
3. Connect the FSM->FSA->CAB->FPA
4. Connect the Lattice programmer (HW-USBN-2B) to 8-pin header connector on the FSA board (check the header pinout in Hardware functionality description/FSA-FTxx/A-00G chapter)
5. Program the bitstream to CrossLink over Lattice Diamond Programmer with SSPI SRAM Programming Access mode 

 - **Note**: DIP pins 1 & 2 must be set opposite and pins 3 & 4 must be set opposite!
 
### 3. SPI on board Flash programming
This chapter explains how to flash the bitstream configuration to on board Flash memory. After power-on the CrossLink will boot with the current configuration stored in Flash memory (DIP pins 2 & 4 must be in ON position, to boot from Flash).
Flash programming over SPI (if step 2 is done, skip to step 5): 
1. Power-off the Jetson
2. On FSA module put the pins 2 & 4 on DIP switch to ON position, and pins 1 & 3 to OFF position
3. Connect the FSM->FSA->CAB->FPA
4. Connect the Lattice programmer (HW-USBN-2B) to 8-pin header connector on the FSA board (check the header pinout in Hardware functionality description/FSA-FTxx/A-00G chapter)
5. Program the bitstream to Flash memory over Lattice Diamond Programmer with SPI Flash Programming Access mode 

 - **Note**: DIP pins 1 & 2 must be set opposite and pins 3 & 4 must be set opposite!
 - **Note**: When booting from flash memory, programmer must be disconnected or the CrossLink may not boot.
 
### Streaming with LVDS/SLVS sensor

To start the stream there is prerequisite to configure CrossLink with the right firmware. The current configuration has its limits, first one and the most obvious to the user is that dynamical adjustment to different pixel formats and data rates isn't available. User must reconfigure CrossLink (as explained above) when changing the pixel format and data rate, available through Image sensor driver. 
    
- ***Note*** - When using Main platform device tree with LVDS/SLVS sensors, driver is probed even if sensor is not physically connected. Then, on stream, care must be taken to choose device which is physically connected.
- ***Note*** - In order to change the pixel format, this procedure must be repeated.
- ***Note*** - Multistream is also supported.
    
---

## Boost CPU  
Image processing and displaying consumes most of CPUs power. To increase the performance of applications and overall performance, CPU boost is a viable option.

Power modes and efficiency of the mode is described in [Development Guide ](https://docs.nvidia.com/jetson/l4t/index.html), chapter "Supported Modes and Power Efficiency" 

---

## Driver advanced functionality
This chapter describes driver advanced functionality that user can exploit.

### Synchronized streaming

Drivers are supporting configuring image sensors in master or slave operation mode. As most of the sensors support outputting triggering signals when in master mode, synchronization without external hardware is also available. To be able for master to trigger slave sensors, triggering signals must be interconnected on FPA module over SW1 DIP switch (refer to FSM_Hardware_UserGuide document).


Main settings in driver are Streaming mode, Operation mode and Synchronizing function (if sensor supports it). 

Streaming mode defines stream, that could be:
1. Standalone 
    - is basic stream, where no additional checks are performed. User must be careful with the settings of the image sensor regarding outputting the trigger signals. 
    - it is recommended that Standalone stream is run with SW1 on FPA module configured as all pins in OFF state.
2. Sync mode
    - is stream mode intended for synchronize stream and additional checks are performed to unable the user to set two sensors as trigger driving sensors.
    - one and only one sensor must be put as master, and others as slave. This is not the rule on sensors with Synchronizing function, they could be all masters but one and only one could be configured as master with Internal synchronizing function.
3. Ex HW mode
    - is stream mode intended for synchronize stream and additional checks are performed to unable the user to set sensor as trigger driving sensors.
    - all sensors must be put as slave, as is expected to have external hardware as trigger signals generator
    
All probed sensors MUST be configured with the same streaming mode!

Operation mode is configuring master/slave operation mode on sensor with option of register setting operation mode. Some sensors have only HW capability of selecting operation mode (refer to FSM_Hardware_UserGuide document). HW setting and SW setting must be the same, or the sensor may not work properly.
1. Master
    - if sensor is put to master mode and is in Sync Streaming mode it will output triggering signals
2. Slave
    - if sensor is put to slave mode it expects to be triggered 
    
Synchronizing function is feature of a master operation mode
1. External sync
    - feature of a sensor where sensor is in master mode but can be synchronized with the signal on XVS trigger pin
    - if put as external, sensor isn't outputting triggering signals
2. Internal sync
    - feature of a sensor where sensor is in master mode but it outputs triggering signals so the other sensor could be synchronized or triggered
    - Triggering pins are outputted in Standalone streaming mode too!
    
    
- **Note**: In slave mode, only synchronous stream is supported.

### Broadcasting 

Some image sensors support broadcasting feature over its second slave address. This feature is implemented with modified i2c-multiplexer driver. Basically, writing to last channel of multiplexer will broadcast message over all channels. Current implementation of drivers with available broadcasting feature have control "I2C Broadcast controls" available. Changing that controls property from "Unicast" to "Broadcast" configures all available sensors with their second slave address and first sensor with "Broadcast" property with ACK on the second slave address. Even without generating I2C ACK bit, sensors are capable of accepting information that are coming to that address. 

- **Note**: Only gain, integration time and exposure time controls are broadcasted.


### Configured firmware on CrossLink

Validation of configured firmware on CrossLink is implemented because IP core is statically adjust to one image frame property. For user to validate if the right firmware configuration is configured, there is "Check Firmware Compatibility" control. Validation is performed within the start of stream process, but the user have the ability to check the configuration on "Check" property. Control will return "OK" or "Error" property depending on configuration.

### Slave mode triggering and syncronization (LVDS/SLVS sensors)
LVDS/SLVS image sensors can be set to work in master or slave mode. Master mode is set by default but there is a possibility to control driving image sensor through specific CrossLink registers in slave mode. When slave mode is enabled, CrossLink will generate slave signals (XHS, XVS or XTRIG) according to values in registers. For that purpose Timing Generator module was developed inside firmware design. 

Also, there is a possibility to synchronize slave signals (XHS, XVS or XTRIG) to external input synchronization signal generated by an external source or some other image sensor.

There are few controls that must be considered when setting image sensor in slave mode:
- **Exposure**
- **Frame Rate**
- **Operation Mode**
- **Global Shutter Mode**
- **Timing Generator Mode**
- **Expanded exposure**
- **Delay frame**

More details about Timing Generator modes and registers can be found in the Timing Generator User Guide.

---

## Known issues

In this chapter issues that are known will be listed. If the specific issue has a workaround  this also will be described.

### Driver issues
1. Streaming with slave/master configuration, libArgus stops stream starting from sensor on connector J5 to J8. If the master is placed on connector before slave, it will be set to software standby and slave sensor will lost trigger signals and won't output frames. That will cause argus to crash and reboot will be necessary.

    **Place slave sensors before master on FPA connectors.**
    
2. Frame rate and Exposure control ranges are updated after start stream. Driver is calculating one-line time reading current Image sensor registers settings. Those registers are modified by driver depending on various Image sensor configuration and set on start stream. After modifying those registers, one-line time is recalculated, and ranges of Frame rate and Exposure controls are updated.
3. If the driver is supporting different pixel formats on certain resolution, control ranges will be taken from the first mode (first supported pixel resolution) and it could be that before first stream those are not the maximum supported ranges by sensor for that resolution and pixel format. Control ranges will be correctly updated after the start stream.



---
## Revision History

| Version | Date           | Description                                                      |
| ------- | -------------- | -----------------------------------------------------------------|
| 1.7.0   | 2020-12-01     | added preserve environment flag to patch script, minor bug fixes and improvements                     |
| 1.6.2   | 2020-09-23     | Removed Note for system restriction                              |
| 1.6.1   | 2020-07-24     | Fixed Nvidia links                                               |
| 1.6.0   | 2020-07-17     | Removed paragraph: XMASTER pin control (LVDS/SLVS sensors); Added new paragraph: Slave mode triggering and syncronization (LVDS/SLVS sensors)
| 1.5.1   | 2020-03-19     | Modified sensor tables; Removed instruction for Hardware; Removed instruction for Software package |
| 1.5.0   | 2019-12-20     | Modified sensor tables                                           |
| 1.4.1   | 2019-12-19     | Minor bug fixes and improvements                                 |
| 1.4.0   | 2019-11-21     | Initial release                                                  |

---
