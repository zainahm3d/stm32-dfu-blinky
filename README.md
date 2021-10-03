Ensure [dfu-util](http://dfu-util.sourceforge.net) is installed

Command to flash over USB DFU (assert the BOOT0 pin while resetting the micro to enter ST bootloader with DFU):
```
dfu-util -a 0 -D build/dfu_blinky.bin -s 0x8000000:leave
```
Command breakdown:
- We want to program the main flash of the micro. This is "alt" 0. We can see which alt is the main flash by running `dfu-util --list` while a DFU device is connected.
```sh
$ dfu-util --list
Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2021 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Found DFU: [0483:df11] ver=2200, devnum=1, cfg=1, intf=0, path="1-1", alt=3, name="@Device Feature/0xFFFF0000/01*004 e", serial="359A365D3433"
Found DFU: [0483:df11] ver=2200, devnum=1, cfg=1, intf=0, path="1-1", alt=2, name="@OTP Memory /0x1FFF7800/01*512 e,01*016 e", serial="359A365D3433"
Found DFU: [0483:df11] ver=2200, devnum=1, cfg=1, intf=0, path="1-1", alt=1, name="@Option Bytes  /0x1FFFC000/01*016 e", serial="359A365D3433"
Found DFU: [0483:df11] ver=2200, devnum=1, cfg=1, intf=0, path="1-1", alt=0, name="@Internal Flash  /0x08000000/04*016Kg,01*064Kg,01*128Kg", serial="359A365D3433"
```
- The `-D` option runs the "download" command, then we follow it with the path of the binary file (our application).
- The `-s` option is the target address of the device to start flashing at. On most STM32 devices, this will be `0x08000000`. See the datasheet for details.
- Finally, following the address with `:leave` will "leave" DFU mode after flashing, and fire up the application.

This will flash the application and reboot the micro. Open a serial console at any baud rate to view the current systick / 1000 (uptime in seconds). Send the char `r` to reboot the device. This is done using the native USB-CDC driver, so no UART adapter is needed, just use the onboard USB port.

Typical output:
```
$ python3 -m serial.tools.miniterm
uptime: 6 seconds
uptime: 7 seconds
uptime: 8 seconds
uptime: 9 seconds
uptime: 10 seconds
uptime: 11 seconds
uptime: 12 seconds
uptime: 13 seconds
$ r
Going down for reset
```

Device: STM32F401CCU6 "Black Pill"

Moderately interesting code is in `Core/src/main.c:main()` and `USB_DEVICE/App/usbd_cdc_if.c:CDC_Receive_FS()`

TODO: Maybe one day I will add a command to jump to the bootloader. This can be done by writing a byte to NVM or RTC memory, and checking it as the first step of bootup (before initializing clocks!). If the "go to bootloader" byte is set, then we can jump to the bootloader. This is required since the bootloader expects a freshly booted micro, with all peripherals in their fresh boot state.