# Design-Your-Arduino-Nano-33-IoT
## Introduction

[Arduino Nano 33 IoT](https://store-usa.arduino.cc/products/arduino-nano-33-iot?selectedStore=us) is a great board with BLE, WiFi, IMU. It can be used as a mainboard of projects, and using breadboard to connect more peripherals. It is a simple and quick way for research. In addition, we can design a PCB with the desired peripheral functionality by just leaving pad locations for the Arduino Nono 33 IoT and soldering it on. In this project, based on the official open source ([Schematics](https://content.arduino.cc/assets/NANO33IoTV2.0_sch.pdf) and [Eagle files](https://content.arduino.cc/assets/Nano33IoT.zip)), I designed a new Arduino 33 Nano IoT and added the TB6612 and UWB module as peripherals (As shown below).

<img src="/img/pcb_3d.png" width="300"> <img src="/img/pcb_real.png" width="300">


## PCB design

I will not introduce this part too much. The main schematic refers to the [official documents](https://content.arduino.cc/assets/NANO33IoTV2.0_sch.pdf), and then add some peripherals needed for your project, such as sensor interfaces, motor drivers, etc.

## Bootloader

The [article](https://support.arduino.cc/hc/en-us/articles/8991429732124-Burn-the-bootloader-on-Arduino-Nano-33-IoT) about how to burn the bootloader on Arduino Nano 33 IoT can be found on the Arduino official website. There are two methods, I chose to use [Arduino MKR Zero](https://store-usa.arduino.cc/products/arduino-mkr-zero-i2s-bus-sd-for-sound-music-digital-audio-data?selectedStore=us) as a programmer to do this task, and a SanDisk Ultra 128G SD card. Then we can follow the instructions, but I found the codes need to be tuned a little.

1. Download the bootloader binary, rename it to `fw.bin`, and move this file to SD card.
2. Insert the SD card into Arduino MKR Zero, connect the board to computer using USB cable.
3. Open Arduino IDE, install **Adafruit DAP library**.
4. Open [`flash_from_SD_nkrzero.ino`](/flash_from_SD_mkrzero/flash_from_SD_mkrzero.ino) file in this repository using Arduino IDE, and upload to Arduino MKR Zero. We will see below messages in serial monitor.

![programmer](/img/programmer.png)

> Note: In the official steps, we select **File > Examples > Adafruit DAP library > samd21 > flash_from_SD** from the Arduino IDE's menus. However, this sketch has some issues when we use Arduino MKR Zero as programmer.
> - [GCC 4.4 warnings](#gcc-44-warnings)
> - [Card failed, or not present](#card-failed-or-not-present)

5. Connect the programmer Arduino board to the target Arduino board as follows:

| Programmer board    | Target board (Nano 33 IoT) |
| -------- | ------- |
| VCC  | +3.3V    |
| 10 | SWDIO     |
| 9    | SWCLK    |
| GND | GND     |
| 11    | RST    |

Here I use a [**PIN PROBE CLIP**](https://www.digikey.sg/en/products/detail/adafruit-industries-llc/5433/16584075), which can connect two boards together easily.

<img src="/img/connect.png" width="500">

6. Press the reset button on the Arduino MKR Zero. Following messages should be shown in serial monitor.

![result](/img/result.png)

## Firmware

After burn the bootloader, we also need to install the firmware, otherwise some modules such as BlueTooth can't work well.

1. Plug the USB cable of the new Arduino Nano 33 IoT board into your computer.

<img src="/img/firmware_usb.png" width="500">

2. Select the target board.

<img src="/img/target_board.png" width="300">

3. **Tools > Firmware Updater > select board > check updates > install**

<img src="/img/firmware_success.png" width="500">

> - [Firmware: Installation failed. Please try again.](#firmware-installation-failed-please-try-again)

After finishing above steps, a new Arduino Nano 33 IoT is made.

## Issues

### GCC 4.4 warnings

The [**Adafruit DAP library**](https://github.com/adafruit/Adafruit_DAP) is used. The following warnings maybe show:
```
\Documents\Arduino\libraries\Adafruit_DAP_library\Adafruit_DAP_SAM.h:166:40: note: offset of packed bit-field 'Adafruit_DAP_SAMx5::<unnamed union>::<unnamed struct>::<anonymous>' has changed in GCC 4.4
```
To solve these warnings, we can open **Adafruit_DAP_SAM.h** in the following directories:
```
\Documents\Arduino\libraries\Adafruit_DAP_library\Adafruit_DAP_SAM.h
```
Replace lines 165-190 of the code with the following (Change all **uint8_t -> uint16_t**):
```
  typedef union {
    struct __attribute__((__packed__)) {
      uint16_t BOD33_Disable : 1;
      uint16_t BOD33_Level : 8;
      uint16_t BOD33_Action : 2;
      uint16_t BOD33_Hysteresis : 4;
      uint16_t : 8;
      uint16_t : 3;
      uint16_t NVM_BOOT : 4;
      uint16_t : 2;
      uint16_t SEESBLK : 4;
      uint16_t SEEPSZ : 3;
      uint16_t RAM_ECCDIS : 1;
      uint16_t : 8;
      uint16_t WDT_Enable : 1;
      uint16_t WDT_Always_On : 1;
      uint16_t WDT_Period : 4;
      uint16_t WDT_Window : 4;
      uint16_t WDT_EWOFFSET : 4;
      uint16_t WDT_WEN : 1;
      uint16_t : 1;
      uint16_t NVM_LOCKS : 32; // As per SAM D5x/E5x Family datasheet page 53
      uint32_t User_Page : 32;
    } bit;
    uint8_t reg[USER_ROW_SIZE];
  } USER_ROW;
```
### Card failed, or not present

If we follow the official instructions, we will perhaps find function **SD.begin()** cannot work, and the following text on the serial monitor.
```
Card failed, or not present
```
We can make the following modefication for the example codes.
- **Step 1**

The pin correspondence is as follows, so we need to change pins define.

| Programmer board    | Target board (Nano 33 IoT) |
| -------- | ------- |
| VCC  | +3.3V    |
| 10 | SWDIO     |
| 9    | SWCLK    |
| GND | GND     |
| 11    | RST    |

Replace lines 6-8:
```
#define SWDIO 10
#define SWCLK 9
#define SWRST 11
```

- **Step 2**

Below line 11, add the following codes at lines 12-28:
```
#ifndef SDCARD_SS_PIN
const uint8_t SD_CS_PIN = SS;
#else  // SDCARD_SS_PIN
// Assume built-in SD is used.
const uint8_t SD_CS_PIN = SDCARD_SS_PIN;
#endif  // SDCARD_SS_PIN

// Try max SPI clock for an SD. Reduce SPI_CLOCK if errors occur.
#define SPI_CLOCK SD_SCK_MHZ(50)

#if HAS_SDIO_CLASS
#define SD_CONFIG SdioConfig(FIFO_SDIO)
#elif  ENABLE_DEDICATED_SPI
#define SD_CONFIG SdSpiConfig(SD_CS_PIN, DEDICATED_SPI, SPI_CLOCK)
#else  // HAS_SDIO_CLASS
#define SD_CONFIG SdSpiConfig(SD_CS_PIN, SHARED_SPI, SPI_CLOCK)
#endif  // HAS_SDIO_CLASS

```

Then replace lines 55-57:
```
  if (!SD.begin(SD_CONFIG)) {
    error("Card failed, or not present");
  }
```

- **Step 3**

We should check the SD card format first, here I use SanDisk Ultra 128G, so some codes need to be tuned.

Replace line 36 from:
```
SdFat SD;
```
to
```
SdExFat SD;
```

Replace line 60 from:
```
 File32 dataFile = SD.open(FILENAME);
```
to
```
 ExFile dataFile = SD.open(FILENAME);
```

### Firmware: Installation failed. Please try again.

When we first install firmware, sometimes will meet this error. 

<img src="/img/firmware_failed.png" width="500">

At this time, we just seltect the board and install firmware again, usually the problem will be solved.
