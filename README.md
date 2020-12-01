# esptool-ftdi.py
This version correct a bug which prevent serial from closing because close is undefined


## Overview

This `esptool.py` wrapper lets you use RTS/CTS for ESP8266/ESP32
chip resets instead of the usual RTS/DTR.  It only works with
FTDI-based USB to serial adapters.

Normally, CTS is an input.  On FTDI chips, CTS can be reconfigured as
an output by entering bitbang mode.  This wrapper replaces all of the
internal `esptool.py` serial routines with direct calls into
`libftdi1` and `libusb-1.0` to dynamically switch between normal and
bitbang modes as needed to make the typical reset-into-bootloader
sequence work.

## Requirements

* [libftdi1 1.4](https://www.intra2net.com/en/developer/libftdi/index.php)
* [libusb 1.0](https://libusb.info/)

This was tested on Linux only.  Some of the `ctypes` library loading
probably needs to be tweaked for other OSes.

## 2 install options 
## #1 Integrating with ESP-IDF

In your Makefile, after the line

    include $(IDF_PATH)/make/project.mk

add

    ESPTOOL_FTDI := /path/to/esptool-ftdi.py
    ESPTOOLPY_SERIAL := $(PYTHON) $(ESPTOOL_FTDI) $(ESPTOOLPY_SRC) --chip esp32 --port $(ESPPORT) --baud $(ESPBAUD) --before $(CONFIG_ESPTOOLPY_BEFORE) --after $(CONFIG_ESPTOOLPY_AFTER)

This could be simplified with some changes in ESP-IDF.

## #2 Esptool replacement

Locate your esptool.py in $IDF_PATH/components/esptool_py/esptool.py
Rename it esptool.py.back
Put esptool-ftdi.py in place and rename it esptool.py

## Ubuntu 19.04
If you have unmet dependency : 

    sudo apt install python3-libusb1 python3-ftdi1

If you can't use without sudo follow those steps : 

    sudo usermod -aG tty $USER
    sudo usermod -aG dialout $USER
    echo 'SUBSYSTEM=="usb", ATTRS{idVendor}=="0403", GROUP="dialout"' | sudo tee -a /etc/udev/rules.d/60-libftdi.rules >         /dev/null
    reboot 
