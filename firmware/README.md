# Firmware

Board name in ArduPilot: **`GEC-H743`** (`APJ_BOARD_ID 1200`, not registered upstream - fine for a prototype).

| File | Content |
|---|---|
| `ardupilot/GEC-H743/hwdef.dat`, `hwdef-bl.dat` | board definition: pins, SPI/I2C/UART/CAN, 12 PWM/DShot outputs, sensors, DMA |
| `ardupilot/build/arducopter_with_bl.hex` | Copter binary **with bootloader** (flash at `0x08000000` via SWD/ST-Link or USB DFU) |
| `ardupilot/build/arducopter.apj` | Copter image for ArduPilot ground stations (after the bootloader is installed) |
| `ardupilot/build/AP_Bootloader.bin` | bootloader only |
| `ardupilot/BUILD_REPORT.md`, `win_patches_*.diff` | how the firmware was built on native Windows (Russian) |

Built from ArduPilot commit `dc9c4e2` (2026-11-21), Copter, `./waf configure --board GEC-H743 && ./waf copter`: 1,539,600 bytes, ~78 % of application flash.
**Not yet tested on real hardware.**

To rebuild: copy `ardupilot/GEC-H743/` to `libraries/AP_HAL_ChibiOS/hwdef/GEC-H743/` in an [ArduPilot](https://github.com/ArduPilot/ardupilot) checkout and build as usual.

ArduPilot is licensed under **GPLv3**; the binaries here are GPLv3 builds of ArduPilot with this board definition. Source: https://github.com/ArduPilot/ardupilot
