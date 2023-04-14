# Arancino Pico core
![Release](https://img.shields.io/github/v/release/smartmeio/arancino-core-rp2040?style=plastic)

This core is a fork of the [earlephilhower Arduino-Pico](https://github.com/earlephilhower/arduino-pico) core modified to support the functionality of the Arancino architecture.
It uses the bare Raspberry Pi Pico SDK and a custom GCC 10.3/Newlib 4.0 toolchain.

# Supported Boards
* Arancino Pico

## Installation on Arduino IDE

This core is available as a package in the Arduino IDE cores manager. If you want to install it:

  1. Open the **Preferences** of the Arduino IDE.
  2. Add this URL `https://raw.githubusercontent.com/smartmeio/arancino-boards/master/package_smartmeio_index.json` in the **Additional Boards Manager URLs** field, and click OK.
  3. Open the **Boards Manager** (menu `Tools` -> `Board` -> `Board Manager...`)
  4. Install **Arancino RP2040 Boards**
  5. Select one of the boards under **Arancino RP2040 Boards** in `Tools` -> `Board` menu

## Installation on PlatformIO and Visual Studio Code
To create a project with Visual Studio Code and PlatformIO it is necessary to initially create a project for Raspberry Pico and then modify the `platformio.ini` file in order to point to the Arancino packages. The `platformio.ini` file must be modififed in order to contains this configuration:

```
[env:arancinopico]
platform = https://github.com/smartmeio/platform-raspberrypi.git#1.7.3-arancino
board = arancinopico
framework = arduino
platform_packages =
    smartmeio/framework-arduino-rp2040-arancino@https://github.com/smartmeio/arancino-core-rp2040.git
    toolchain-pico@https://github.com/earlephilhower/pico-quick-toolchain/releases/download/1.4.0-c/x86_64-linux-gnu.arm-none-eabi-0196c06.220714.tar.gz
lib_deps = https://github.com/smartmeio/arancino-library
upload_port = ...
```
Any other used library must be included as dependency through `lib_deps` (as for the Arancino library) or saved under the `lib` folder; in the latter case the project structure should look like:
```
include
lib
+-- your_library
     +-- examples
     +-- include
     +-- keywords.txt
     +-- library.json
     +-- src
src
+-- main.cpp
test
```

Alternatively, you can set an extra directory in which to search for libraries. For example, if you want to include all the libraries installed in the Arduino IDE, you can specify it adding this lines to the `platformio.ini` configuration file:
```
lib_extra_dirs = ~/Arduino/libraries
```


# Uploading Sketches
To upload your first sketch, you will need to hold the BOOTSEL button down while plugging in the Pico to your computer.
Then hit the upload button and the sketch should be transferred and start to run.

After the first upload, this should not be necessary as the `arancino-pico` core has auto-reset support.
### Arduino IDE
Select the appropriate serial port shown in the Arduino `Tools` -> `Port` -> `Serial Port` menu once (this setting will stick and does not need to be
touched for multiple uploads). This selection allows the auto-reset tool to identify the proper device to reset.

### PlatformIO
Set the appropriate serial port into the `platformio.ini` file, e.g.:
```
upload_port = /dev/ttyACM1
```
Then hit the upload button and your sketch should upload and run.

In some cases the Pico will encounter a hard hang and its USB port will not respond to the auto-reset request.  Should this happen, just
follow the initial procedure of holding the BOOTSEL button down while plugging in the Pico to enter the ROM bootloader.

# Uploading Sketches with Picoprobe
If you have built a Raspberry Pi Picoprobe, you can use OpenOCD to handle your sketch uploads and for debugging with GDB.

Under Windows a local admin user should be able to access the Picoprobe port automatically, but under Linux `udev` must be told about the device and to allow normal users access.

To set up user-level access to Picoprobes on Ubuntu (and other OSes which use `udev`):
````
echo 'SUBSYSTEMS=="usb", ATTRS{idVendor}=="2e8a", ATTRS{idProduct}=="0004", GROUP="users", MODE="0666"' | sudo tee -a /etc/udev/rules.d/98-PicoProbe.rules
sudo udevadm control --reload
````

The first line creates a file with the USB vendor and ID of the Picoprobe and tells UDEV to give users full access to it.  The second causes `udev` to load this new rule.  Note that you will need to unplug and re-plug in your device the first time you create this file, to allow udev to make the device node properly.

Once Picoprobe permissions are set up properly, then select the board "Raspberry Pi Pico (Picoprobe)" in the Tools menu and upload as normal.

# Uploading Sketches with pico-debug
[pico-debug](https://github.com/majbthrd/pico-debug/) differs from Picoprobe in that pico-debug is a virtual debug pod that runs side-by-side on the same RP2040 that you run your code on; so, you only need one RP2040 board instead of two.  pico-debug also differs from Picoprobe in that pico-debug is standards-based; it uses the CMSIS-DAP protocol, which means even software not specially written for the Raspberry Pi Pico can support it.  pico-debug uses OpenOCD to handle your sketch uploads, and debugging can be accomplished with CMSIS-DAP capable debuggers including GDB.

Under Windows and macOS, any user should be able to access pico-debug automatically, but under Linux `udev` must be told about the device and to allow normal users access.

To set up user-level access to all CMSIS-DAP adapters on Ubuntu (and other OSes which use `udev`):
````
echo 'ATTRS{product}=="*CMSIS-DAP*", MODE="664", GROUP="plugdev"' | sudo tee -a /etc/udev/rules.d/98-CMSIS-DAP.rules
sudo udevadm control --reload
````

The first line creates a file that recognizes all CMSIS-DAP adapters and tells UDEV to give users full access to it.  The second causes `udev` to load this new rule.  Note that you will need to unplug and re-plug in your device the first time you create this file, to allow udev to make the device node properly.

Once CMSIS-DAP permissions are set up properly, then select the board "Raspberry Pi Pico (pico-debug)" in the Tools menu.

When first connecting the USB port to your PC, you must copy [pico-debug-gimmecache.uf2](https://github.com/majbthrd/pico-debug/releases/) to the Pi Pico to load pico-debug into RAM; after this, upload as normal.

# Debugging with Picoprobe/pico-debug, OpenOCD, and GDB

### Arduino IDE
The installed tools include a version of OpenOCD (in the pqt-openocd directory) and GDB (in the pqt-gcc directory).  These may be used to run GDB in an interactive window as documented in the Pico Getting Started manuals from the Raspberry Pi Foundation.  For [pico-debug](https://github.com/majbthrd/pico-debug/), replace the raspberrypi-swd and picoprobe example OpenOCD arguments of "-f interface/raspberrypi-swd.cfg -f target/rp2040.cfg" or "-f interface/picoprobe.cfg -f target/rp2040.cfg" respectively in the Pico Getting Started manual with "-f board/pico-debug.cfg".

### PlatformIO (picoprobe)
This additional debugging information must be added to the `platformio.ini` file
```
upload_protocol = picoprobe
debug_tool = picoprobe
build_type = debug
lib_archive = no
```

# Enabling FreeRTOS support
This core supports FreeRTOS: to include and enable it just include the `Arancino.h` library and, depending on the used IDE, enable it. 

### Arduino IDE
Through the menu `Tools` -> `Using FreeRTOS` -> `Yes`.

### PlatformIO
Add an extra flag to the `platformio.ini` configuration file:
```
build_flags = -DUSEFREERTOS
```


# Features
* Adafruit TinyUSB Arduino (USB mouse, keyboard, flash drive, generic HID, CDC Serial, MIDI, WebUSB, others)
* Generic Arduino USB Serial, Keyboard, and Mouse emulation
* Filesystems (LittleFS and SD/SDFS)
* Multicore support (setup1() and loop1())
* FreeRTOS SMP support
* Overclocking and underclocking from the menus
* digitalWrite/Read, shiftIn/Out, tone, analogWrite(PWM)/Read, temperature
* Peripherals:  SPI master, Wire(I2C) master/slave, dual UART, emulated EEPROM, I2S audio input, I2S audio output, Servo
* printf (i.e. debug) output over USB serial

The RP2040 PIO state machines (SMs) are used to generate jitter-free:
* Servos
* Tones
* I2S Input
* I2S Output
* Software UARTs (Serial ports)

# Licensing and Credits
* The [Arduino Pico core](https://github.com/earlephilhower/arduino-pico) is developed and maintained by [Earle F. Philhower, III](mailto:earlephilhower@yahoo.com).
* The [Arduino IDE and ArduinoCore-API](https://arduino.cc) are developed and maintained by the Arduino team. The IDE is licensed under GPL.
* The [RP2040 GCC-based toolchain](https://github.com/earlephilhower/pico-quick-toolchain) is licensed under under the GPL.
* The [Pico-SDK](https://github.com/raspberrypi/pico-sdk) is by Raspberry Pi (Trading) Ltd and licensed under the BSD 3-Clause license.
* [Arduino-Pico](https://github.com/earlephilhower/arduino-pico) core files are licensed under the LGPL.
* [LittleFS](https://github.com/ARMmbed/littlefs) library written by ARM Limited and released under the [BSD 3-clause license](https://github.com/ARMmbed/littlefs/blob/master/LICENSE.md).
* [UF2CONV.PY](https://github.com/microsoft/uf2) is by Microsoft Corporation and licensed under the MIT license.
* Some filesystem code taken from the [ESP8266 Arduino Core](https://github.com/esp8266/Arduino) and licensed under the LGPL.
* [FreeRTOS](https://freertos.org) is Copyright Amazon.com, Inc. or its affiliates, and distributed under the MIT license.



# Full Arduino Pico Documentation by earlephilhower
See https://arduino-pico.readthedocs.io/en/latest/ along with the examples for more detailed usage information.