# WiFi Link - ESP8266 firmware for Arduino

This is a fork of Arduino.org WiFi Link firmware. The goal of this fork is make improvements and provide documentation.

The WiFi Link firmware is an ESP8266 arduino sketch developed by Arduino.org in Arduino IDE using Arduino esp8266 core. It was developed for the Arduino Star Otto, Arduino Primo and [Uno WiFi Developer Edition](https://github.com/jandrassy/UnoWiFiDevEdSerial1).

With changes in this fork, it can be used with generic esp8266 in combination with an Arduino MCU.

The corresponding Arduino library is [WiFi Link library](https://github.com/jandrassy/arduino-library-wifilink).

## Building from source code

The WiFi Link firmware is an Arduino sketch so you can build it in Arduino IDE and upload it to ESP from Arduino IDE.

Building WiFi Link firmware from source files gives you possibility to build the newest version, build a branch version, build some fork version or change something in source code you need.

You need the esp8266 core for Arduino IDE. To install it using boards manager, follow the [instructions](https://github.com/esp8266/Arduino#installing-with-boards-manager).

Additionally, install the [Arduino ESP8266 filesystem uploader IDE plugin](https://github.com/esp8266/arduino-esp8266fs-plugin#arduino-esp8266-filesystem-uploader-)

### Download the source code

Use the source code from this fork. Every GitHub repository has a green "Clone or download" button which opens a small menu. Choose "Download ZIP".

Open or extract the downloaded zip and copy the folder ArduinoFirmwareEsp from zip to your sketches folder.

Start Arduino IDE and open the ArduinoFirmwareEsp.ino sketch. It opens additional files as tabs in IDE.

Go to config.h tab and uncomment the `#define` for your board or GENERIC_ESP8266.

### Board selection and Verify

In tools menu select board options and choose your board/shield/module or Generic ESP8266 from the ESP8266 section of the Boards menu. 

Now verify the sketch with the Verify button. The first compilation after changing the board will take time.

From now on always check the selected board in the right bottom corner of the IDE window. 

### Upload

[Setup](https://github.com/jandrassy/arduino-firmware-wifilink/wiki/Test-Setup) your esp board/shield/module for flashing. Connect it to USB of the computer and if it doesn't support DTR, put it into programming mode with dedicated button.

Use the Upload button in IDE to upload the WiFi Link firmware.

### SPIFFS
 
SPIFFS is the file system for ESP8266. 
 
In subfolder `data` of the source codes of the WiFi Link firmware are the static web files (html, css, js) for the Web Panel. You can add your own files and they will be accessible on expected url.
 
We installed a plugin tool in chapter "Install esp8266 packages". This plugin creates a "ESP8266 Sketch Data upload" command in Tools menu.
 
The tool builds the SPIFFS binary and uploads it to selected port. With serial port you must put your board/shield/module into flashing mode in it doesn't support reset on DTR. Then choose "ESP8266 Sketch Data upload" from Tools menu.

### Web Panel

Before connecting to WebPanel after serial flashing of firmware, power cycle the module/board/shield. 

Web Panel network configuration uses the configuration Access Point (AP). User connects to the WiFi network created by a device, goes to fixed IP address URL and configures the network access in a Web Panel. The device connects in STA mode to the selected WiFi network and is accessible at IP address assigned by a DHCP or a static address set in Web Panel.

The AP network name is set as SSIDNAME in config.h. The WiFi Link fixed address in AP mode is http://192.168.240.1/.

### OTA upload 

The WiFi Link firmware supports OTA upload of new version of the firmware binary. OTA upload will only work if some version of WiFi Link is working in the ESP8266 and is configured to STA or STA+AP mode.

After configuring WiFi Link to WiFi STA or STA+AP mode, the IDE will detect it on network using mdns. The network 'port' will be accessible in Port submenu of Tools menu. Choose the network port for the OTA upload and use the Upload button in IDE.

*Notes: Do not put the board in bootloader mode. Do not search for some special programmer in Tools menu.*

The upload of the ArduinoFirmwareEsp.ino will overwrite the firmware binary and leave the SPIFFS part of the flash unchanged.

OTA upload works with SPIFS too and is fast.

### config.json

WiFi Link firmware writes WiFi settings into SPIFFS file config.json. SPIFFS upload overrides the SPIFFS content and the setting are lost. After restart WiFi Link firmware goes to AP mode and you must once again connect to this AP, choose the WiFi network, enter the password and connect back to your WiFi. 

If you often upload the SPIFFS, add your config.json file into data subfolder of the WiFi Link firmware source codes. The basic content is `{"ssid":"yourwifi","password":"yourpassword"}.` 

### Atmega sketch OTA upload support

The WiFi Link firmware build without `#define MCU_OTA` (config.h) doesn't support ATmega sketch OTA upload. To build from the source codes with `MCU_OTA` you need a library called dfu.

For OTA with generic esp8266 module/board/shield, the reset pin of the ATmega must be [connected](https://github.com/jandrassy/arduino-firmware-wifilink/wiki/Test-Setup) to an ESP GPIO pin. Default in dfu library is GPIO5. You can change it to GPIO0, as it is on pinout header on most ESP modules. Star Otto and Uno WiFi have special setting hardcoded.

Only settings for the ATmega328p with Uno bootloader (Optiboot) exist in the dfu library. 

#### The dfu library

The dfu library is not available in Library manager in IDE. The source code repository is on [GitHub](https://github.com/ciminaghi/libdfu/tree/arduino-debug). And it is not an arduino library. First running a script from arduino subfolder builds the [arduino version](https://github.com/jandrassy/arduino-firmware-wifilink/wiki/lib/dfu.zip). Install it to your libraries folder.

#### Sketch OTA upload tool

Tool for OTA uploading the Atmega sketch is a python script available in arduino.org [GitHub repository](https://github.com/arduino-org/arduino-tool-mcu-ota). To run it, you need Python 2.7, 3.x doesn't work. With python installed you can package the script as exe. Instructions are in the GitHub repository.

A way to integrate the tool arduino-tool-mcu-ota into IDE is patching the platform.txt file in `hardware/arduino/avr` IDE installation folder of IDE.

The platform.txt values to change are 
```
tools.avrdude.network_cmd={runtime.tools.arduinoOTA.path}/bin/arduino_mcuota
tools.avrdude.upload.network_pattern="{network_cmd}" -i {serial.port} -p 80 -f "{build.path}/{build.project_name}.hex"
```

Put the tool executable to a location evaluated by tools.avrdude.network_cmd. In IDE 1.8.5 it is `hardware/tools/avr/bin` in installation folder of IDE.
