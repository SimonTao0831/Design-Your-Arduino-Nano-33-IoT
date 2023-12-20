# Arduino-Bootloader-Issue
## 
Recently, I designed a PCB, and its main part is based on the Arduino Nano 33 IoT. Before using it, a bootloader should burn. However, there are some issues I met so I recorded them and their solutions here.

The [article](https://support.arduino.cc/hc/en-us/articles/8991429732124-Burn-the-bootloader-on-Arduino-Nano-33-IoT) about how to burn the bootloader on Arduino Nano 33 IoT can be found on the Arduino official website. There are two methods, I chose to use Arduino MKR Zero as a programmer to do this task. Then we can follow the instructions, but I found the codes need to be tuned a little.

##
The [**Adafruit DAP library**](https://www.arduino.cc/reference/en/libraries/adafruit-dap-library/) will be used. The following warnings maybe show:
```
.\Documents\Arduino\libraries\Adafruit_DAP_library\Adafruit_DAP_SAM.h:166:40: note: offset of packed bit-field 'Adafruit_DAP_SAMx5::<unnamed union>::<unnamed struct>::<anonymous>' has changed in GCC 4.4
```
To solve these warnings, we can open **Adafruit_DAP_SAM.h** in the following directories:
```
.\Documents\Arduino\libraries\Adafruit_DAP_library\Adafruit_DAP_SAM.h
```
Replace lines 165-192 of the code with the following (Change all uint8_t -> uint16_t):
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
  USER_ROW _USER_ROW;
};
```
##
If we follow the official instructions, we will find function **SD.begin()** cannot work.
```
Card failed, or not present
```
We can make the following modefication for origin example codes.
- 1
Replace lines 5-8:
```
#define SD_CS 4
#define SWDIO 10
#define SWCLK 9
#define SWRST 11
```
The pin correspondence is as follows:

| Programmer board    | Target board (Nano 33 IoT) |
| -------- | ------- |
| VCC  | +3.3V    |
| 10 | SWDIO     |
| 9    | SWCLK    |
| GND | GND     |
| 11    | RST    |

- 2
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

- 3
Replace lines 55-57:
```
  if (!SD.begin(SD_CONFIG)) {
    error("Card failed, or not present");
  }
```

- Test
![Result](./result.md)
