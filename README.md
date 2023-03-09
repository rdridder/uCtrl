# uCtrl

uCtrl is the base library of [uMODULAR](https://github.com/midilab/uMODULAR) hardware project. It provides driver layer for all uMODULAR modules and user interface layer to easly create, extend and share more advanced Arduino applications.

This library also enables realtime-like functionality inside Arduino ecosystem by using hardware timer interruption to create time predictable task management.

uCtrl is a choice for robust, portable and fast-making musical instruments, sequencers, audio/video controllers and other related machines for Arduino platform.

## Supported microcontrollers

The following microcontrollers and boards are supported and were tested:

**AVRs:** ATmega168, ATmega328, ATmega16u4, ATmega32u4 and ATmega2560.  
**ARMs:** ESP32, SAMD21 and All Teensy ARM microcontrollers.  

**Boards:** All AVR Arduino boards, All Teensy AVRs and ARMs, ESP32 based boards and Seedstudio XIAO M0.

## Modules avaliable

[PAGE](#page): Modular User Interface programming using pages and components.  
[AIN](#ain): Wire up to 64 potentiometers.  
[DIN](#din): Wire up to 64 push buttons or rotary encoders.  
[DOUT](#dout): Wire up to 64 leds or any other digital output.  
[TOUCH](#touch): Wire up to 32 capcitive touch buttons(teensy arms only for now).  
[MIDI](#midi): Agregate and control MIDI interfaces.  
[OLED](#oled): Connect a OLED screen.  
[STORAGE](#storage): Make use of epprom and/or connect a sdcard.  
[RAM](#ram): Make use of external SRAM.  

## How to use?

In case you are using git for your Arduino application we strongly suggest to setup uCtrl library for your project as a [submodule](#submodule).

If you dont need versionng control for you Arduino application the fast way to setup you uCtrl library for your project is via [download](#download).

### Submodule

This allows you to manage uCtrl version related to your project version.

```console
$ cd YourSketch/
$ mkdir src/
$ cd src/
$ git submodule add https://github.com/midilab/uCtrl.git
$ git add uCtrl/
$ git commit -m "uCtrl library added to project"
$ git push
```

after each new clone of your application repository you'll need to init subproject uCtrl only once:

```console
$ git clone https://github.com/mister_dev/YourSketch.git
$ cd YourSketch/
$ git submodule update --init --recursive
```

### Download

In case you want to download uCtrl instead of use it as a submodule.  

Create a src/ directory inside your sketch, then clone or [download](https://github.com/midilab/uCtrl/archive/refs/heads/main.zip) this repository.

unzip **main.zip** and move the unziped folder named **uCtrl-main/** to **YourSketch/src/uCtrl/**

### Setting your sketch

Create a new file on IDE tab for your sketch named **modules.h**

Configure **modules.h** accordly to the needed modules for your application.(see Modules for MACRO setup)

### modules.h

You should have a guard macro for this file:

```c
#ifndef __U_CTRL_MODULES_H__
#define __U_CTRL_MODULES_H__

// your uCtrl modules MACRO setup

#endif
```

### Initial Project structure 

Your initial project structure should look like:  
/YourSketch  
/YourSketch/YourSketch.ino  
/YourSketch/modules.h  
/YourSketch/src/uCtrl/  

# Modules

A introduction to modules, base schematics and **modules.h** MACRO setup.

## AIN

Wire up to 64 potentiometers.  

Use this module to create midi controllers or any other user interface with potentiometers.

This module can handle single ADC ports on your microcontroller or multiplexed ADC port via 4051 or 4067 CI.

**modules.h**
```c
#ifndef __U_CTRL_MODULES_H__
#define __U_CTRL_MODULES_H__

// enables the driver
#define USE_AIN

// for multiplexed support uncomment the needed driver
//#define USE_AIN_4051_DRIVER
//#define USE_AIN_4067_DRIVER

#endif
```

YourSketch.ino
```c++
#include <Arduino.h>
#include "src/uCtrl/uCtrl.h"

typedef enum {
  POT_1,
  POT_2,
  POT_3,
  POT_4,
  //...,
  //...,
};

// get change values from connected potentiometers
void ainInput(uint8_t port, uint16_t value, uint8_t interrupted)
{
    switch (port) {
        case POT_1:
            // do something with port 1 value(0 ~ 1023)
            break;
        case POT_2:
            // do something with port 2 value(0 ~ 1023)
            break;
        case POT_3:
            // do something with port 3 value(0 ~ 1023)
            break;
        case POT_4:
            // do something with port 4 value(0 ~ 1023)
            break;
        //...
        //...
    }
}

// this plugs 4x 4051 making 32 potentiometers avaliable
void setAinMultiplexed4051()
{
    // initAin(uint8_t MUX_CTRL_PIN_A, uint8_t MUX_CTRL_PIN_B, uint8_t MUX_CTRL_PIN_C)
    uCtrl.initAin(D3, D4, D5);
    // plug(uint8_t MUX_ANALOG_PORT_PIN_X)
    uCtrl.ain->plug(A0);
    uCtrl.ain->plug(A1);
    uCtrl.ain->plug(A2);
    uCtrl.ain->plug(A3);
}

// this plugs 2x 4067 making 32 potentiometers avaliable
void setAinMultiplexed4067()
{
    // initAin(uint8_t MUX_CTRL_PIN_A, uint8_t MUX_CTRL_PIN_B, uint8_t MUX_CTRL_PIN_C, uint8_t MUX_CTRL_PIN_D)
    uCtrl.initAin(D3, D4, D5, D6);
    // plug(uint8_t MUX_ANALOG_PORT_PIN_X)
    uCtrl.ain->plug(A0);
    uCtrl.ain->plug(A1);
}

// this plugs 4 microntroller ADC ports making 4 potentiometers avaliable
void setAinSingle()
{
    uCtrl.initAin();
    // plug(uint8_t MUX_ANALOG_PORT_PIN_X)
    uCtrl.ain->plug(A1);
    uCtrl.ain->plug(A0);
    uCtrl.ain->plug(A2);
    uCtrl.ain->plug(A3);
}

void setup() 
{
    setAinSingle();
    //setAinMultiplexed4051();
    //setAinMultiplexed4067();
    uCtrl.ain->setCallback(ainInput);
    // most arduinos max is 1024.
    // you can lower or raiser this value for your needs
    uCtrl.ain->setMaxAdcValue(128);

    // only init uCtrl after all modules setup
    uCtrl.init();
}

void loop()
{
  uCtrl.run();
}
```

Make use of 4051 or 4067 for multiplexed analog inputs  
*link or image to the ain options schematic

## DIN

Wire up to 64 push buttons or rotary encoders.

This module can handle single Digital ports on your microcontroller or multiplexed Digital port via 165 CI.

**modules.h**
```c
#ifndef __U_CTRL_MODULES_H__
#define __U_CTRL_MODULES_H__

// enables the driver
#define USE_DIN

//
// for direct usage of microcontroller Digital port pin
// all pins are setup with internal pullup resistor
// so wire you button between microcontroller pin and GND(no need for resistor)
// USE_DIN_MAX_PORTS is default to 16 if you dont set it
//
#define USE_DIN_PORT_PIN
//#define USE_DIN_MAX_PORTS   8

// 2 driver options for multiplexed button input: SPI and BITBANG.
// use only one driver option!

//
// using SPI hardware wich is the recommended way in case you have 
// a Free or shared SPI device.
//
//#define USE_DIN_SPI_DRIVER
//#define DIN_LATCH_PIN   D4

//
// using bitbang in case you dont have a free SPI. 
// this is slower and uses the CPU to process the data transfer.
// this options requires you to define the latch, data and clock pins of 165.
//
//#define USE_DIN_BITBANG_DRIVER
//#define DIN_LATCH_PIN   D4
//#define DIN_DATA_PIN    D5
//#define DIN_CLOCK_PIN   D6

#endif
```

YourSketch.ino
```c++
#include <Arduino.h>
#include "src/uCtrl/uCtrl.h"

typedef enum {
  BUTTON_1,
  BUTTON_2,
  ENCODER_DEC,
  ENCODER_INC,
  //...,
  //...,
};

// get change values from connected push buttons or encoders
void dinInput(uint8_t port, uint16_t value, uint8_t interrupted)
{
    switch (port) {
        case BUTTON_1:
            // do something with BUTTON_1 value(HIGH | LOW)
            break;
        case BUTTON_2:
            // do something with BUTTON_2 value(HIGH | LOW)
            break;
        case ENCODER_DEC:
            // do something with ENCODER_DEC value(HIGH)
            break;
        case ENCODER_INC:
            // do something with ENCODER_INC value(HIGH)
            break;
        //...
        //...
    }
}

// this plugs 1x 165 using SPI device, making 8 push buttons avaliable
void setDinMultiplexedSpi()
{
    // initDin(spi device)
    uCtrl.initDin(&SPI);
    // plugSR(uint8_t number of 165's to plug)
    uCtrl.din->plugSR(1);
}

// this plugs 2x 165 using bitbang driver, making 16 push buttons avaliable
void setDinMultiplexedBitbang()
{
    uCtrl.initDin();
    // plugSR(uint8_t number of 165's to plug)
    uCtrl.din->plugSR(2);
}

// this plugs 4 microntroller Digital ports making 4 push buttons avaliable
void setDinSingle()
{
    uCtrl.initDin();
    // plug(uint8_t DIGITAL_PORT_PIN_X)
    uCtrl.din->plug(D2);
    uCtrl.din->plug(D3);
    uCtrl.din->plug(D4);
    uCtrl.din->plug(D5);
}

void setup() 
{
    setDinSingle();
    //setDinMultiplexedSpi();
    //setDinMultiplexedBitbang();
    uCtrl.din->setCallback(dinInput);

    // encoders setup?
    // in pair and sequential pins always! 
    // pairs starting with even ids: 0 and 1, 2 and 3, 4 and 5
    uCtrl.din->encoder(ENCODER_DEC, ENCODER_INC);

    // only init uCtrl after all modules setup
    uCtrl.init();
}

void loop()
{
  uCtrl.run();
}
```

Make use of 165 shift register to expand buttons and/or encoders.  
*link or image to the din options schematic

## DOUT

Wire up to 64 leds or any other digital output.

This module can handle single Digital output ports on your microcontroller or multiplexed Digital output port via 595 CI.

**modules.h**
```c
#ifndef __U_CTRL_MODULES_H__
#define __U_CTRL_MODULES_H__

// enables the driver
#define USE_DOUT

//
// for direct usage of microcontroller Digital output port pin
// USE_DOUT_MAX_PORTS is default to 8 if you dont set it
//
#define USE_DOUT_PORT_PIN
//#define USE_DOUT_MAX_PORTS   8

// 2 driver options for multiplexed output: SPI and BITBANG.
// use only one driver option!

//
// using SPI hardware wich is the recommended way in case you have 
// a Free or shared SPI device.
//
//#define USE_DOUT_SPI_DRIVER
//#define DOUT_LATCH_PIN   D7

//
// using bitbang in case you dont have a free SPI. 
// this is slower and uses the CPU to process the data transfer.
// this options requires you to define the latch, data and clock pins of 595.
//
//#define USE_DOUT_BITBANG_DRIVER
//#define DOUT_LATCH_PIN   D7
//#define DOUT_DATA_PIN    D8
//#define DOUT_CLOCK_PIN   D9

#endif
```

YourSketch.ino
```c++
#include <Arduino.h>
#include "src/uCtrl/uCtrl.h"

typedef enum {
  LED_1,
  LED_2,
  LED_3,
  LED_4,
  //...,
  //...,
};

// this plugs 1x 595 using SPI device, making 8 digital outputs avaliable
void setDoutMultiplexedSpi()
{
    // initDout(spi device)
    uCtrl.initDout(&SPI);
    // plugSR(uint8_t number of 595's to plug)
    uCtrl.dout->plugSR(1);
}

// this plugs 2x 595 using bitbang driver, making 16 digital outputs avaliable
void setDoutMultiplexedBitbang()
{
    uCtrl.initDout();
    // plugSR(uint8_t number of 165's to plug)
    uCtrl.dout->plugSR(2);
}

// this plugs 4 microntroller Digital ports making 4 digital outputs avaliable
void setDoutSingle()
{
    uCtrl.initDout();
    // plug(uint8_t DIGITAL_PORT_PIN_X)
    uCtrl.dout->plug(D2);
    uCtrl.dout->plug(D3);
    uCtrl.dout->plug(D4);
    uCtrl.dout->plug(D5);
}

void setup() 
{
    setDoutSingle();
    //setDoutMultiplexedSpi();
    //setDoutMultiplexedBitbang();

    // only init uCtrl after all modules setup
    uCtrl.init();

    // now you can use the uCtrl interface to set states on the outputs
    // set all leds off
    uCtrl.dout->writeAll(LOW); 
    // set all leds on
    //uCtrl.dout->writeAll(HIGH); 
    // set LED_1 on
    uCtrl.dout->write(LED_1, HIGH); 
}

void loop()
{
  uCtrl.run();
}
```

Make use of 595 shiftregister to expand digital output like leds.  
*link or image to the dout options schematic

## Touch

Wire up to 32 capacitive touch buttons using common ADC port.

Make use of 4067 to multiplex capacitive buttons.  
*link or image to the touch options schematic

## MIDI

Agregate and control MIDI interfaces.
Agregator for [Francois Best's Arduino MIDI library](https://github.com/FortySevenEffects/arduino_midi_library) that can handle realtime operations inside uCtrl.  

Make use of MIDI circuit.
*link or image to the midi options schematic

## Oled

Connect a OLED screen.

Display helper class for [U8g2 oled library](https://github.com/olikraus/U8g2_Arduino). 


Make use of i2c or spi devices.
*link or image to the oled options schematic  

## Storage

Make use of epprom and/or sdcard.

Storage abstration for for Epprom and Sdcard usage. Sdcard library helper using [greiman's SdFat library](https://github.com/greiman/SdFat/)
*link or image to the sdcard options schematic

## Ram

(23LC1024 only for now)

## Page

Environment programming to interface your app using the ecosystem of uCtrl, can be used with components for interface
