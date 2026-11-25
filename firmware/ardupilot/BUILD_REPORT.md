# Проверка сборки ArduPilot для GEC-H743 (22.11.2026)

## Итог
- hwdef.dat и hwdef-bl.dat проходят `chibios_hwdef.py` без единой правки пинов и синтаксиса (предупреждений нет).
- Загрузчик (bootloader) для GEC-H743 собран полностью: AP_Bootloader.bin 40 084 байта (apj 40 057 байт).
- **`waf copter` собран целиком (1333/1333, 0 ошибок компиляции и линковки).** arducopter.bin = 1 539 600 Б (text 1 537 364 + data 2 228;
  BSS 424 412), arducopter.apj = 1 376 774 Б, arducopter_with_bl.hex = 4 699 220 Б. Занято ~78 % из 1920 КБ (2048 минус 128 КБ загрузчика).
  Артефакты скопированы в `ardupilot_hwdef/build_out/`. Единственный сбой: шаг waf `make_intel_hex.py` (запуск .py как exe на Windows,
  WinError 193); hex сделан вручную из `build/GEC-H743`: `py -3.12 ../../Tools/scripts/make_intel_hex.py bin/arducopter.bin ../../Tools/bootloaders/GEC-H743_bl.bin 128`.
  На железе не проверялось.
- Админ-права не требовались ни для одного шага. Установочный скрипт ArduPilot (Cygwin, C:\cygwin64, MAVProxy) НЕ запускался.

## Шаг 1-2. hwdef-процессор
Копия hwdef лежит в `libraries/AP_HAL_ChibiOS/hwdef/GEC-H743/`. `hwdef-bl.dat` = MatekH743/hwdef-bl.dat, изменена только строка
`APJ_BOARD_ID 1200` (файл также сохранён в `ardupilot_hwdef/GEC-H743/hwdef-bl.dat`). Остальное в bl-файле осталось от Matek
(пины LED PE3, USB PA11/PA12 совпадают с нашим hwdef; PD10 = PINIO1 OUTPUT LOW, а у нас PD10 = SAFETY_IN, CS-пины PB12/PC13/PD4/PE2 —
от Matek). На работу bl это не влияет, но перед выпуском стоит привести к нашей плате: убрать PINIO1 на PD10, оставить CS только
для IMU1_CS(PC15), IMU2_CS(PE11), BARO_CS(PB12).

Команды (из корня репозитория, Python 3.12):
```
py -3.12 -m pip install --user pexpect future lxml empy==3.3.4 pymavlink intelhex dronecan pyserial setuptools
py -3.12 libraries/AP_HAL_ChibiOS/hwdef/scripts/chibios_hwdef.py -D out_hw --params None libraries/AP_HAL_ChibiOS/hwdef/GEC-H743/hwdef.dat
py -3.12 libraries/AP_HAL_ChibiOS/hwdef/scripts/chibios_hwdef.py -D out_bl --bootloader libraries/AP_HAL_ChibiOS/hwdef/GEC-H743/hwdef-bl.dat
```
Нюансы: без `-I` (скрипт импортирует соседние модули); для не-bl без параметра `--params` скрипт падает (TypeError на None),
нужен любой несуществующий путь, например `None` (waf передаёт то же).
Результат hwdef.dat: `hwdef.h`, `hw.dat`, DMA-карта записаны; единственное сообщение — «Bootloader GEC-H743_bl.bin does not exist
and AP_BOOTLOADER_FLASHING_ENABLED» (штатно, пока не собран bl). Строки `PC4 VDD_5V_SENS ADC1 SCALE(3)`, `PD11 LED_SAFETY OUTPUT LOW`,
`SPIDEV bmi270`, `COMPASS IST8310 I2C:1:0x0E false ROTATION_NONE`, `define HAL_I2C_INTERNAL_MASK 2` приняты без замечаний.
Правок нашего hwdef: **нет**.

## Шаг 3. Полная сборка на нативной Windows
Установочный скрипт не использован (ставит Cygwin в C:\cygwin64 и MAVProxy, плюс интерактивный ReadKey). Вместо него:
- gcc-arm-none-eabi 10-2020-q4-major (zip с developer.arm.com, 200 МБ) распакован в рабочую папку;
- GNU make 4.4.1 (xpack windows-build-tools 4.4.1-3, zip 2.7 МБ, без установки) — в PATH на сессию, нужен и `sh.exe` из него/Git;
- сабмодули (depth 1): waf, mavlink(+pymavlink), ChibiOS, DroneCAN/{DSDL,pydronecan,dronecan_dsdlc,libcanard}, lwip, littlefs, picolibc.
  (CrashDebug в этой версии ArduPilot не сабмодуль.)
- **Сборка идёт из `C:\ap175\src`** (копия), а не из папки с кириллицей и пробелами: gcc 10 mingw не понимает кириллицу в путях
  (первый заход упал на picolibc из-за этого + регистра пути).

Платформенные патчи (только в рабочей копии, не в нашем hwdef; diff-ы: `win_patches_ardupilot.diff`, `win_patches_waf.diff`, окружение `win_env.sh`):
1. `modules/waf/waflib/Tools/c_config.py`: `open('/dev/null')` -> `open(os.devnull)`.
2. `modules/waf/waflib/Context.py` (exec_command): строковые rule на win32 запускать через `sh -c` со слэшами вместо обратных
   (иначе `BUILDDIR=x make` не работает в cmd.exe).
3. `Tools/ardupilotwaf/picolibc.py`: фильтр `_without_toolchain_libc` сравнивал пути с `/arm-none-eabi/include`, на Windows пути с `\`
   -> подтягивались заголовки newlib и picolibc не компилировался.
4. `Tools/ardupilotwaf/toolchain.py`: убрать `-Wl,--enable-auto-import` из LINKFLAGS (ld arm-none-eabi его не знает).
5. Опции waf: `--no-submodule-update` (git submodule status на Windows-путях падает).

Порядок команд (Git Bash, `. win_env.sh` задаёт PATH к gcc/make):
```
cd /c/ap175/src
py -3.12 waf configure --board GEC-H743 --bootloader --no-submodule-update
py -3.12 waf bootloader -j12
cp build/GEC-H743/bin/AP_Bootloader.bin Tools/bootloaders/GEC-H743_bl.bin
py -3.12 waf configure --board GEC-H743 --no-submodule-update
py -3.12 waf copter -j12
```
Результат: configure (обе конфигурации) OK; bootloader OK (Text 39 976 Б); copter OK (см. выше).
Сборка очень медленная на этом ПК (при -j6 ~3-12 задач/мин, общее время copter ~3 ч, CPU 2-9 %, узкое место — запуск процессов).
Рекомендуется `-j12`..`-j16`.

## Повторить/пересобрать
`cd /c/ap175/src; . ../env.sh; py -3.12 waf copter -j12`. Для прошивки в плату: arducopter.apj (через загрузчик/Mission Planner) или
arducopter_with_bl.hex (SWD/DFU, вместе с загрузчиком GEC-H743_bl.bin). Загрузчик -- AP_Bootloader.bin/.apj. Board ID 1200 (не зарегистрирован).

## Диск
`C:\ap175` ~2.3 ГБ (+ build растёт), `G:\...\fw\ardupilot` ~2.4 ГБ (первичный клон, gcc, make). Можно удалить G-копию после
завершения (кроме нужных diff-ов, лежащих в ardupilot_hwdef).

## Команды с правами администратора
Не потребовались. (Установочный скрипт ArduPilot для Windows их бы запросил — пропущен.)
