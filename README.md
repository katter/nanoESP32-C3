nanoESP32-C3
-----------
[中文](./README_cn.md)
* [nanoESP32-C3 Introduce](#nanoESP32-C3-Introduce)
* [Module Specifications](#Module-Specifications)
* [ESPLink](#ESPLink)
* [Demo](#Demo)
* [Product Link](#Product-Link)
* [Reference](#Reference)


# nanoESP32-C3 Introduce
nanoESP32-C3 is ESP32-C3 dev board made by MuseLab, base on ESP32-C3 Modules manufactured by Expressif, with on-board usb-to-serial, TYPE-C, RGB LED,  the on-board programmer ESPLink (based on DAPLink) support usb-to-serial (compatible with traditional use), drag-n-drop program, and jtag debug, made it convenient for development and test.

<div align=center>
<img src="https://github.com/wuxx/nanoESP32-C3/blob/master/doc/nanoESP32-C3-1.jpg" width = "500" alt="" align=center />
</div>

# Module Specifications
nanoESP32-C3 use the ESP32-C3-MINI-1 module manufactured by Expressif, here are key features
Component|Detail |
----|----|
MCU         | 32bit RISC-V ESP32-C3FN4 up to 160MHz |
ROM         | 384KB |
SRAM        | 400KB(16KB for cache) |
Flash       | 4MB |
WiFi        | IEEE 802.11bgn |
Bluetooth   | BLE 5.0 & mesh |
GPIO        | 22 |
SPI         | 3 |
UART        | 2 |
I2C         | 1 |
RMT         | 2T+2R |
DMA         | 3T+3R |
LED-PWMC    | 6 channel |
TWAI        | 1 |
ADC         | 2 x 12bit, 6 channel |
Temp sensor | 1 |
Timer       | 6 |
Security    | OTP/AES/SHA/RSA/RNG/HMAC |

# ESPLink
nanoESP32-C3 has an on-board programmer called ESPLink, which has some useful features, here are the detailed explanation
## USB-to-Serial
compatible with the traditional use, can used to program with esptool.py or monitor the serial output message. examples for reference:
```
$idf.py -p /dev/ttyACM0 flash monitor
$esptool.py --chip esp32c3 \
           -p /dev/ttyACM0 \
           -b 115200 \
           --before=default_reset \
           --after=hard_reset \
           --no-stub \
           write_flash \
           --flash_mode dio \
           --flash_freq 80m \
           --flash_size 2MB \
           0x0     esp32c3/bootloader.bin \
           0x8000  esp32c3/partition-table.bin \
           0x10000 esp32c3/blink_100.bin
```

## Drag-and-Drop Program
the ESPLink support drag-n-drop program, after power on the board, a virtual USB Disk named `ESPLink` will appear, just drag the flash image into the ESPLink, wait for some seconds, then the ESPLink will automatically complete the program work. it's very convenient since it's no need to rely on external tools and work well on any platform (Win/Linux/Mac .etc), some typical usage scenarios: quickly verification, compile on cloud server and program on any PC, firmware upgrade when used on commercial products .etc
note: the flash image is a splicing of three files (bootloader.bin/partition-table.bin/app.bin), just expand the `bootloader.bin` to size 0x8000, expand the `partition-table.bin` to size 0x10000, and concatenate these three files, use the script `esppad.sh` under tools directory to make it, example for reference:
```
$./tools/esppad.sh bootloader.bin partition-table.bin app.bin flash_image.bin
```


## JTAG Debug
ESPLink support jtag interface to debug the ESP32-C3, it's useful for those hobbyist who are interested with RISC-V assembly language, or for product developer to fix the bug when system crash, here are the instructions

### Openocd Install

The pre-compiled package distributed by the openocd-esp32 repository lacks support for ESPLink, so it needs to be compiled manually:

```
$ git clone https://github.com/espressif/openocd-esp32.git
$ cd openocd-esp32
$ git checkout v0.11.0-esp32-20220706
```

openocd-esp32 is not quite stable, so it is recommended to checkout a specified version like `v0.11.0-esp32-20220706`, be careful not to checkout `v0.11.0`, it lacks support for esp32 targets.

```
$ ./bootstrap
```

If an AC_INIT error occurs, change line 2 of configure.ac from `AC_INIT([openocd], [ ]` to `AC_INIT([openocd], [ "" ]` (add the quotes where the version would go).

```
$ ./configure --enable-cmsis-dap --disable-werror
$ make -j
$ sudo make install
```

If you have multiple custom versions of openocd on your system, it is recommended **not** to run `sudo make install`, just run the above compiled `openocd-esp32/src/openocd` when needed, e.g.

```
./src/openocd -f interface/cmsis-dap.cfg -f target/esp32c3.cfg -s "tcl" -c 'adapter_khz 10000'
```

### Burn the efuse
the efuse JTAG_SEL_ENABLE should be burned to enable the jtag function.
```
$espefuse.py -p /dev/ttyACM0 burn_efuse JTAG_SEL_ENABLE
```

### Attach to ESP32-C3
set GPIO10 to 0 for choose the GPIO function to JTAG, then power on the board and execute the attach script
```
$sudo openocd -f tcl/interface/cmsis-dap.cfg -f tcl/target/esp32c3.cfg -c 'adapter_khz 10000'
Open On-Chip Debugger  v0.10.0-esp32-20201202-30-gddf07692 (2021-03-22-16:48)
Licensed under GNU GPL v2
For bug reports, read
        http://openocd.org/doc/doxygen/bugs.html
adapter speed: 10000 kHz

force hard breakpoints
Info : Listening on port 6666 for tcl connections
Info : Listening on port 4444 for telnet connections
Info : CMSIS-DAP: SWD  Supported
Info : CMSIS-DAP: JTAG Supported
Info : CMSIS-DAP: FW Version = 0255
Info : CMSIS-DAP: Serial# = 0800000100430028430000014e504332a5a5a5a597969908
Info : CMSIS-DAP: Interface Initialised (JTAG)
Info : SWCLK/TCK = 1 SWDIO/TMS = 1 TDI = 1 TDO = 1 nTRST = 0 nRESET = 1
Info : CMSIS-DAP: Interface ready
Info : High speed (adapter_khz 10000) may be limited by adapter firmware.
Info : clock speed 10000 kHz
Info : cmsis-dap JTAG TLR_RESET
Info : cmsis-dap JTAG TLR_RESET
Info : JTAG tap: esp32c3.cpu tap/device found: 0x00005c25 (mfg: 0x612 (Espressif Systems), part: 0x0005, ver: 0x0)
Info : datacount=2 progbufsize=16
Info : Examined RISC-V core; found 1 harts
Info :  hart 0: XLEN=32, misa=0x40101104
Info : Listening on port 3333 for gdb connections
```

### Debug
when attach success, open another ternimal to debug, you can use gdb or telnet, explained separately as follows
#### Debug with Gdb
```
$riscv32-esp-elf-gdb -ex 'target remote 127.0.0.1:3333' ./build/blink.elf
(gdb) info reg
(gdb) s
(gdb) continue
```

#### Debug with telnet
```
$telnet localhost 4444
>reset
>halt
>reg
>step
>reg pc
>resume
```

# Demo
The preset factory test firmware is located in the demo directory. expected that the RGB LED should start to flash after power-on, how to compile the source code show here for reference
```
$git clone https://github.com/espressif/esp-idf.git
$cd esp-idf && ./install.sh && . ./export.sh
$cd examples/get-started/blink/
$idf.py set-target esp32c3
$idf.py -p /dev/ttyACM0 flash monitor
```
# Product Link
[nanoESP32-C3 Board](https://www.aliexpress.com/item/1005003081928629.html?spm=a2g0o.productlist.0.0.4a666866ubSEVc&algo_pvid=f0a84073-dd83-42a1-bf2b-a53530a6cb62&algo_exp_id=f0a84073-dd83-42a1-bf2b-a53530a6cb62-0)

# Reference
### esp-idf
https://github.com/espressif/esp-idf
### esp32-c3 get-started
https://docs.espressif.com/projects/esp-idf/en/latest/esp32c3/get-started/
### esp32-c3
https://www.espressif.com/zh-hans/products/socs/esp32-c3
