# Плата FC-STM32 — полётный контроллер на STM32H743VIT6 под ArduPilot. Описание платы (25.11.2026)

Проект EasyEDA Pro `Quad_01_FC-STM32`, hwdef ArduPilot `GEC-H743`. Описание составлено по файлам папки платы (`README.md`, `ТЗ_FC-STM32.md`, `Pinmap_и_hwdef_FC-STM32.md`, `BOM.md`, `firmware_ArduPilot\GEC-H743\hwdef.dat`, `hwdef-bl.dat`, `firmware_ArduPilot\BUILD_REPORT.md`), по описанию схемы `Claude\easyeda\designs\fc_stm32.py` + `common.py` (каждый пин ↔ цепь — это то, что реально нарисовано в EasyEDA), по экспорту `EasyEDA\Quad_01_FC-STM32_Netlist.json` / `_BOM.xlsx` (23.11.2026) и по системным документам `00 - ТЗ и система\02_ТЗ_система.md`, `04_План_сборки_коптера.md`, `05_Проверки_прошивок.md`, `06_BOM_все_платы.md`. Там, где в источниках данных нет, так и написано: «не задано в документах».

---

## 1. Что это за плата и место в системе

FC-STM32 — основной полётный контроллер квадрокоптера проекта 175 (класс Pixhawk 2.4.8 / Matek H743) под прошивку **ArduPilot Copter**. На плате: MCU STM32H743VIT6 (480 МГц, 2 МБ flash, 1 МБ ОЗУ), два независимых IMU (IIM-42652 по SPI1 и BMI270 по SPI4), барометр MS5611 (SPI2), встроенный компас IST8310 (I2C1), microSD (SDMMC1, 4 бита) для журналов, USB-C для настройки и прошивки, CAN (DroneCAN) через TJA1051T/3, 12 выходов PWM/DShot + WS2812, зуммер, safety switch, два порта MAVLink-телеметрии. Плата заменяет покупной Pixhawk 2.4.8: по системному ТЗ она должна дать ту же функциональность (стабилизация, GPS-навигация Loiter/RTL/Auto, геозоны, failsafe, логи на microSD, измерение V/I батареи, управление подвесом, два порта телеметрии). Решение в пользу H743 (а не Nucleo-F446RE) принято потому, что F446 (512 КБ flash) не поддерживается ни одной полётной прошивкой; Nucleo остаётся стендовой платой.

С другими платами и узлами проекта FC-STM32 соединяется кабелями JST-GH по Pixhawk Connector Standard: **PWR** (модуль питания, плата 03) → разъём POWER (5 В + аналоговые V/I) и разъём I2C (INA226, адрес 0x40, BATT_MONITOR=21); **TLM-AIR** (телеметрия 433 МГц, плата 04) → TELEM1 (UART7, MAVLink 2); **SiYI HM30** (видеолинк, покупной) → TELEM2 (USART2); **GPS M10N с внешним компасом** → GPS1 (USART1 + I2C2 + safety switch + зуммер); **приёмник DBR4 (CRSF)** → RC (USART6); **ESC ×4** → гребёнка M1–M8 (DShot/PWM) + ESC-телеметрия (UART4); **подвес камеры** (плата 08 GIMBAL или имеющийся 2-осевой) → PWM AUX11/AUX12 либо UART3 (USART3); **DroneCAN-устройства** → CAN1. Экспериментальная плата FC-ESP32 (02) — отдельная реализация, на рабочий коптер не ставится.

## 2. Картинки и файлы

Все пути — относительно папки платы `01 - FC-STM32 (полётный контроллер H743)\`.

| Файл | Что это | Дата файла |
|---|---|---|
| `EasyEDA\Quad_01_FC-STM32_Schematic.png` | схема (растр) | 25.11.2026 11:05 |
| `EasyEDA\Quad_01_FC-STM32_Schematic.pdf` | схема (PDF) | 23.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_PCB_2D.png` | 2D-вид платы с разводкой (итерация трассировки 24.11.2026 15:55) | 24.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_PCB.pdf` | PCB в PDF — **устарел**, экспорт 23.11 до трассировки (экспорт разведённой платы через мост не укладывается в 240 с; обновить из GUI: File → Export → PDF) | 23.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_3D_top.png` | 3D-вид платы сверху | 25.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_3D_bottom.png` | 3D-вид платы снизу | 25.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_3D.step` | 3D-модель платы с компонентами, STEP (45.8 МБ) | 25.11.2026 11:07 |
| `EasyEDA\Quad_01_FC-STM32.epro2` | проект EasyEDA Pro (схема + плата, актуальная разводка) | 24.11.2026 16:01 |
| `EasyEDA\Quad_01_FC-STM32_BOM.xlsx` | BOM из EasyEDA (42 строки, 96 компонентов, LCSC-номера) | 23.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_CPL.xlsx` | координаты компонентов (pick-and-place) | 23.11.2026 |
| `EasyEDA\Quad_01_FC-STM32_Netlist.json` | нетлист: 96 компонентов, 102 цепи, правила по цепям | 23.11.2026 |
| `README.md` | статус, расхождения, состояние EasyEDA | 24.11.2026 |
| `ТЗ_FC-STM32.md` | требования, выбор компонентов, распределение периферии | 21.11.2026 (v0.1) |
| `Pinmap_и_hwdef_FC-STM32.md` | пин-карта, распиновки разъёмов, черновик hwdef | 21.11.2026 (v0.1) |
| `BOM.md` | черновик BOM | — |
| `firmware_ArduPilot\GEC-H743\hwdef.dat`, `hwdef-bl.dat` | определение платы для ArduPilot (прошивка и загрузчик) | 21–22.11.2026 |
| `firmware_ArduPilot\build_out\` | собранные `AP_Bootloader.bin/.apj`, `arducopter.bin/.apj`, `arducopter_with_bl.hex`, `build_copter.log` | 22.11.2026 |
| `firmware_ArduPilot\BUILD_REPORT.md`, `win_env.sh`, `win_patches_*.diff` | как собиралось на Windows, патчи инструментов | 22.11.2026 |

Схема:

![Схема FC-STM32](EasyEDA/Quad_01_FC-STM32_Schematic.png)

2D-вид платы (разводка 24.11.2026; ось Y на картинке направлена вверх, поэтому гребёнки J12/J14/J15, которые в описании расстановки названы «верхней кромкой» (y ≈ 1.5–6.6 мм), на картинке внизу, а разъёмы J5/J3/J8 (y = 46.5) — вверху):

![2D-вид платы](EasyEDA/Quad_01_FC-STM32_PCB_2D.png)

3D-виды:

![3D сверху](EasyEDA/Quad_01_FC-STM32_3D_top.png)

![3D снизу](EasyEDA/Quad_01_FC-STM32_3D_bottom.png)

## 3. Состав и принцип работы по узлам

Обозначения и цепи — по `designs/fc_stm32.py` и нетлисту; имена цепей совпадают с сигналами hwdef.

**MCU и тактирование (U1, Y1, C1–C15, R1, SW1, J13).** STM32H743VIT6 в LQFP-100 (LCSC C5271084). Кварц Y1 8 МГц 3225 (C42448903, YSX320GA, CL = 12 пФ) на PH0/PH1 (OSC_IN/OSC_OUT) с нагрузочными C13/C14 18 пФ (0402); в hwdef `OSCILLATOR_HZ 8000000`. Кварца 32.768 кГц нет: PC15 занят под IMU1_CS, RTC работает от LSI (Pinmap §1). Питание ядра: VCAP1 (вывод 48) и VCAP2 (вывод 73) → C9, C10 по 2.2 мкФ на GND (обязательно — урок платы проекта 160, где VCAP остался неподключённым). Все VDD/VDDA/VREF+/VBAT — на +3V3 напрямую, без бусины (заметка README: при необходимости добавить L/FB перед заказом). Развязки: C1–C6 и C12 100 нФ 0402, C7/C8 4.7 мкФ, C11 1 мкФ. BOOT0: R1 10 кОм на GND, кнопка SW1 (тактовая, C49023761) замыкает BOOT0 на +3V3 — вход в системный загрузчик ST (DFU по USB). NRST: C15 100 нФ на GND; на разъём NRST не выведен, тест-точка не задана (в Pinmap — «NRST тест-точка», в схеме её нет). SWD: J13 (PH2.54-1×4, C42431808): 1 GND, 2 +3V3, 3 SWDIO (PA13), 4 SWCLK (PA14).

**IMU1 — IIM-42652 (U2, SPI1).** Индустриальная версия ICM-42688 (C2988404, LGA-14 3×2.5), поддержана драйвером Invensensev3 ArduPilot (`INV3_ID_IIM42652 0x6f`, проверено в исходниках master 21.11.2026). SPI1: SCK PA5, MISO PA6 (AP_SDO), MOSI PD7 (AP_SDA/SDI), CS PC15 (IMU1_CS), INT1 → PB2 (IMU1_DRDY). В hwdef `SPIDEV iim42652 SPI1 DEVID1 IMU1_CS MODE3 2*MHZ 16*MHZ`, `DMA_NOSHARE SPI1* SPI4*`. Питание VDD и VDDIO от датчикового LDO +3V3S, развязки C16 100 нФ + C17 2.2 мкФ. Ротация `ROTATION_NONE` — задать после размещения (открытый вопрос README). При появлении ICM-42688-P на LCSC — замена один в один (тот же footprint и драйвер).

**IMU2 — BMI270 (U3, SPI4).** Резервный IMU (C2836813, LGA-14), драйвер `AP_InertialSensor_BMI270`. Отдельная шина SPI4, чтобы шум и частота опроса не делились с IMU1: SCK PE12 (SCX), MISO PE13 (SDO), MOSI PE14 (SDX), CS PE11 (CSB → IMU2_CS), INT1 → PC13 (IMU2_DRDY; у Matek PC13 = IMU3_CS, третьего IMU у нас нет). hwdef: `SPIDEV bmi270 SPI4 DEVID1 IMU2_CS MODE3 1*MHZ 10*MHZ`. Питание VDD/VDDIO от +3V3S, развязки C18, C19 100 нФ.

**Барометр — MS5611-01BA03 (U4, SPI2).** Классический барометр Pixhawk (C15639, QFN-8). SPI2: SCK PB13, MISO PB14 (SDO), MOSI PB15 (SDI/SDA), CS PB12 (BARO_CS). Вывод PS → GND (режим SPI). hwdef: `SPIDEV ms5611 SPI2 DEVID2 BARO_CS MODE3 10*MHZ 10*MHZ`, `BARO MS5611 SPI:ms5611`. У Matek H743 SPI2 занят OSD — у нас OSD нет. Питание +3V3S, C20 100 нФ. По ТЗ баро накрывать поролоном от потоков воздуха и света.

**Компас — IST8310 (U5, I2C1).** Встроенный магнитометр (C2683055, LGA-16), I2C1: SCL PB6, SDA PB7, подтяжки R2/R3 2.2 кОм к +3V3 (к шине MCU, не к +3V3S). Адрес 0x0E задан выводами CAD1 → +3V3S, CAD0 → GND; RSTN → +3V3S; вывод C1 → C22 220 нФ на GND (README: проверить по даташиту). Питание AVDD/DVDD от +3V3S, C21 100 нФ. В hwdef `I2C_ORDER I2C2 I2C1` (внешняя шина = индекс 0, внутренняя = 1), `COMPASS IST8310 I2C:1:0x0E false ROTATION_NONE`, `HAL_I2C_INTERNAL_MASK 2`, `AP_COMPASS_PROBING_ENABLED 1` — внешний компас GPS на I2C2 ArduPilot находит сам и использует как основной.

**Внешняя шина I2C2 (R14, R15, D5, J7, J6).** SCL PB10, SDA PB11, подтяжки R14/R15 2.2 кОм к +3V3, ESD-матрица D5 PRTR5V0U2X (C330092, VCC = +3V3). Выведена на разъём I2C J7 и на разъём GPS1 J6 (компас GPS 0x0D/0x0E, INA226 модуля PWR 0x40). Адреса не конфликтуют; по Pinmap кабель до PWR — ≤ 20 см или шина 100 кГц.

**microSD (J1, R8–R12, C28).** Держатель TF-01A push-push (C91145), SDMMC1 4 бита: D0 PC8, D1 PC9, D2 PC10, D3 PC11, CK PC12, CMD PD2. Подтяжки 10 кОм 0402 к +3V3: R8 (CMD), R9 (D0), R10 (D1), R11 (D2), R12 (D3); C28 10 мкФ у VDD карты (+3V3). hwdef: `HAL_OS_FATFS_IO 1`, загрузчик умеет прошиваться с карты (`AP_BOOTLOADER_FLASH_FROM_SD_ENABLED 1`). J1 и R8–R11 стоят на нижней стороне платы, карта выходит через нижнюю кромку (под J5/J3).

**USB (J2, R6, R7, D1, R4, R5).** Разъём USB-C 16p TYPE-C-31-M-12 (C165948), только USB 2.0 FS. CC1/CC2 → R6/R7 5.1 кОм на GND (устройство-потребитель). ESD-матрица D1 USBLC6-2SC6 (C7519): выводы 1/6 и 3/4 — сквозные пары линий, 5 — VBUS, 2 — GND. Со стороны разъёма: DP1/DP2 → USB_DP_C → D1.3, DN1/DN2 → USB_DN_C → D1.1. Со стороны MCU: D1.4 → USB_DP → PA12 (OTG_FS_DP); D1.6 → цепь **USB_DN**, а PA11 (OTG_FS_DM) — цепь **USB_DM**: по нетлисту обе цепи однопиновые, т.е. линия D− от MCU к разъёму **не соединена** (см. §9). VBUS → делитель R4/R5 10 кОм/10 кОм → PE15 (VBUS_SENSE, в hwdef `PE15 VBUS INPUT OPENDRAIN`). Корпус разъёма (EH) → GND.

**CAN (U6, C29, R13, J8).** Трансивер TJA1051T/3/1J (C38695, SO-8): TXD ← PD1 (CAN1_TX), RXD → PD0 (CAN1_RX), S (silent) ← PD3 (CAN1_SILENT, hwdef `GPIO_CAN1_SILENT … GPIO(70)`), VCC = +5V (C29 100 нФ), VIO = +3V3 (уровни 3.3 В). CAN_H/CAN_L → разъём J8 CAN1. Терминатор R13 120 Ом впаян постоянно (перемычки в библиотеке нет) — для не-крайнего узла R13 не ставить. В hwdef `HAL_NUM_CAN_IFACES 1` (DroneCAN).

**Питание 5 В → 3.3 В (D2, D3, D4, C23, C24, U7, U8, C25–C27).** Подробно в §4. Две линии 3.3 В: +3V3 (U7, MCU и цифровая периферия) и +3V3S (U8, только датчики U2–U5).

**Разъёмы.** JST-GH 1.25 мм: J3 TELEM1, J4 TELEM2, J5 POWER (SM06B-GHS-TB, C133065), J6 GPS1 (SM10B-GHS-TB, C2683602), J7 I2C, J8 CAN1, J9 RC, J10 UART3, J11 UART8 (SM04B-GHS-TB, C189895). Штыревые 2.54 мм: J12A/B/C — три ряда по 8 (M1–M8: сигнал / +5V_SERVO / GND; в BOM EasyEDA — HDR1X8 C9900015386, «排母», т.е. гнездовая гребёнка), J14A/B/C — три ряда по 4 (AUX9–12), J15 — ESC-телеметрия, J13 — SWD (PH2.54-1×4 C42431808). Распиновки — §5.

**Зуммер и LED (Q1, R18, LED1, LED2, R16, R17, U9, C30, R19).** Зуммер: PA15 (BUZZER, hwdef `TIM2_CH1 TIM2 GPIO(32) ALARM`) → затвор Q1 2N7002 (R18 10 кОм на GND — подтяжка затвора), сток → цепь BUZZ_N → GPS1 J6 pin 9 (BUZZER−), исток GND; «+» зуммера берётся с J6 pin 8 (+3V3) — по стандарту Pixhawk 4 GPS MODULE. Отдельного разъёма BUZZ 2p (как в Pinmap §2) в схеме нет. Светодиоды статуса: LED1 синий (K → PE3 LED0, A ← R16 1 кОм ← +3V3), LED2 зелёный (K → PE4 LED1, A ← R17 1 кОм ← +3V3) — светят при низком уровне; в hwdef `HAL_GPIO_A_LED_PIN 91` (зелёный), `HAL_GPIO_B_LED_PIN 90` (синий). RGB: U9 WS2812B-2020 (C965555), VDD = +5V, DI ← PA8 (WS2812_DI, hwdef `TIM1_CH1 PWM(13)`, `HAL_DEFAULT_NEOPIXEL_COUNT 1`), C30 100 нФ. Safety switch: PD10 (SAFETY_IN, `INPUT PULLUP`) ← J6 pin 6; LED кнопки: PD11 (SAFETY_LED, `LED_SAFETY OUTPUT LOW`) → R19 330 Ом → SAFETY_LED_OUT → J6 pin 7; `HAL_HAVE_SAFETY_SWITCH 1`.

**RC-вход (J9, R20, R21, U10).** Разъём RC J9: 1 +5V, 2 RC_RX, 3 RC_TX, 4 GND. RC_RX → R20 100 Ом → USART6_RX (PC7); RC_TX → R21 100 Ом → USART6_TX (PC6). В hwdef PC7 объявлен двояко: `TIM3_CH2 TIM3 RCININT PULLDOWN LOW` (захват PPM) и `USART6_RX USART6 NODMA ALT(1)` (CRSF/SBUS по UART), PC6 — `USART6_TX NODMA`. CRSF от DBR4 идёт напрямую. Инвертор SBUS U10 SN74LVC1G14DBVR (C7835, VCC = +3V3): вход A = RC_RX, выход Y = цепь RC_RX_INV — **никуда не подключена** (однопиновая цепь в нетлисте); для SBUS нужна перемычка RC_RX_INV → PC7 (решить до заказа). ESD-защиты на RC-разъёме в схеме нет (ТЗ F17 просило).

**Аналоговые входы (R22, R23, C31, C32, R24, R25).** От POWER J5: BATT_V_IN (pin 4) → R22 1 кОм → BATT_VOLTAGE_SENS (PC0), C31 100 нФ на GND; BATT_I_IN (pin 3) → R23 1 кОм → BATT_CURRENT_SENS (PC1), C32 100 нФ. Делители на самом модуле PWR (HAL_BATT_VOLT_SCALE 11.0). Серво-рейка: +5V_SERVO → R24 20 кОм / R25 10 кОм → VDD_5V_SENS (PC4, hwdef `SCALE(3)`) — измерение напряжения BEC от ESC вместо PRESSURE_SENS у Matek. PA4/PA7 (BATT2_VOLTAGE/CURRENT_SENS) и PC5 (RSSI_ADC) объявлены в hwdef, но в схеме подключены только к MCU — на разъёмы не выведены.

## 4. Питание

| Цепь | Откуда | Куда | Элементы |
|---|---|---|---|
| +5V_PWR | разъём POWER J5 pin 1, 2 (BEC 5.3 В 3 А модуля PWR на TPS54560) | D2 SS34 (анод) | дорожка 0.8 мм |
| VBUS | USB-C J2 (A4/B9, B4/A9) | D3 SS34 (анод), D1.5 (ESD), делитель R4/R5 → PE15 | дорожка 0.5 мм |
| +5V | катоды D2 и D3 (диодное «или» PWR / USB на SS34, C8678) | U7, U8, U6 VCC (CAN), U9 WS2812, pin 1 разъёмов J3, J4, J6, J7, J8, J9, J10, J11, pin 4 J15 | TVS D4 SMAJ5.0A (C132575) +5V–GND, C23 10 мкФ, C24 100 нФ; дорожка 0.6 мм |
| +3V3 | U7 XC6220B331MR-G (C86534; по `06_BOM` — LDO 3.3 В 1 А), VIN = CE = +5V | MCU (VDD, VDDA, VREF+, VBAT, вывод 99), microSD, CAN VIO, подтяжки I2C1/I2C2/SD, LED1/LED2, U10, D5, J6 pin 8, J13 pin 2, SW1 | C25 10 мкФ на выходе; дорожка 0.4 мм; заливка на внутреннем слое |
| +3V3S | U8 XC6220B331MR-G, VIN = CE = +5V | только датчики: U2 IIM-42652, U3 BMI270, U4 MS5611, U5 IST8310 (включая CAD1, RSTN) | C26 10 мкФ, C27 100 нФ; дорожка 0.4 мм |
| +5V_SERVO | внешний BEC ESC через ряды «+» гребёнок J12B / J14B | только делитель R24/R25 → PC4 | с +5V платы **не связана**; дорожка 1.0 мм |
| VCAP1 / VCAP2 | внутренний регулятор ядра MCU | — | C9, C10 2.2 мкФ |

Принцип: плата питается 5 В от модуля PWR (основной путь) или от USB (настройка на столе) через два Шоттки SS34; выбирается источник с большим напряжением. От +5V через два отдельных LDO получаются +3V3 для MCU и +3V3S для датчиков (как в Pixhawk — датчики на своём LDO, чтобы цифровой шум не попадал в IMU). Внешние модули (GPS, приёмник, телеметрия, CAN-узлы, подвес по UART) получают +5V с платы через pin 1 своих разъёмов — то есть в том числе и от USB, если PWR не подключён. Серво-рейка ESC-гребёнок питается только от BEC регуляторов и на плату не заходит, FC лишь измеряет её напряжение (PC4). По ТЗ F16 там же должны стоять защитный диод/стабилитрон и TVS — в схеме их нет (только делитель).

Токи (ТЗ §4, оценка): MCU ~150 мА, датчики ~30 мА, microSD до 100 мА, внешние GPS/компас ~150 мА, приёмник ~100 мА, телеметрия до 300 мА пиково → BEC 5 В ≥ 2 А (PWR даёт 3 А). Падение на SS34 и тепловой режим LDO в документах не оценены. Защита: TVS SMAJ5.0A на +5V, USBLC6 на USB, PRTR5V0U2X на I2C2; предохранителя на плате нет (предохранитель 5 А стоит на выходе BEC платы PWR). Защита от переполюсовки — конструктивно ключами JST-GH.

## 5. Разъёмы и подключение

Нумерация пинов JST-GH — по Pixhawk Connector Standard (pin 1 = VCC). Столбец «MCU» — вывод STM32 и сигнал hwdef.

**J5 — POWER, JST-GH 6p (SM06B-GHS-TB, C133065)** → кабель к плате PWR (разъём POWER на PWR).

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V_PWR | → D2 → +5V | BEC 5 В PWR |
| 2 | +5V_PWR | → D2 → +5V | BEC 5 В PWR |
| 3 | BATT_I_IN | R23 → PC1 BATT_CURRENT_SENS (ADC pin 11) | ток 0–3.3 В с INA180A2 на PWR |
| 4 | BATT_V_IN | R22 → PC0 BATT_VOLTAGE_SENS (ADC pin 10) | напряжение, делитель 11:1 на PWR |
| 5 | GND | GND | |
| 6 | GND | GND | |

**J3 — TELEM1, JST-GH 6p (C133065)** → плата TLM-AIR (LoRa 433, mLRS), SERIAL1, MAVLink 2.

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | питание TLM-AIR |
| 2 | UART7_TX | PE8 | → RX модуля |
| 3 | UART7_RX | PE7 | ← TX модуля |
| 4 | UART7_CTS | PE10 | у TLM-AIR не заведён (BRD_SER1_RTSCTS = 0) |
| 5 | UART7_RTS | PE9 | то же |
| 6 | GND | GND | |

**J4 — TELEM2, JST-GH 6p (C133065)** → SiYI HM30 / компаньон-компьютер, SERIAL3 (USART2), до 921600.

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | |
| 2 | USART2_TX | PD5 | → RX HM30 |
| 3 | USART2_RX | PD6 | ← TX HM30 |
| 4 | — | не подключён (CTS) | |
| 5 | — | не подключён (RTS) | |
| 6 | GND | GND | |

**J6 — GPS1, JST-GH 10p (SM10B-GHS-TB, C2683602)** → GPS-модуль M10N с компасом, safety switch и зуммером (как Pixhawk 4 GPS MODULE), SERIAL2 (USART1, UBX 230400).

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | питание GPS |
| 2 | USART1_TX | PA9 | → RX GPS |
| 3 | USART1_RX | PA10 | ← TX GPS |
| 4 | I2C2_SCL | PB10 (подтяжка R14, ESD D5) | компас GPS (0x0D/0x0E) |
| 5 | I2C2_SDA | PB11 (подтяжка R15, ESD D5) | компас GPS |
| 6 | SAFETY_IN | PD10 (INPUT PULLUP) | кнопка safety switch |
| 7 | SAFETY_LED_OUT | R19 330 Ом ← PD11 SAFETY_LED | LED кнопки |
| 8 | +3V3 | LDO U7 | питание LED кнопки / зуммера «+» |
| 9 | BUZZ_N | сток Q1 ← PA15 BUZZER | зуммер «−» |
| 10 | GND | GND | |

**J7 — I2C, JST-GH 4p (SM04B-GHS-TB, C189895)** → разъём I2C платы PWR (INA226, 0x40), дальномер и др. (I2C2 = внешняя шина, индекс 0).

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | |
| 2 | I2C2_SCL | PB10 | |
| 3 | I2C2_SDA | PB11 | |
| 4 | GND | GND | |

**J8 — CAN1, JST-GH 4p (C189895)** → DroneCAN-устройства.

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | |
| 2 | CAN_H | U6 CANH (TXD ← PD1, RXD → PD0) | |
| 3 | CAN_L | U6 CANL | терминатор R13 120 Ом между 2 и 3 — впаян всегда |
| 4 | GND | GND | |

**J9 — RC, JST-GH 4p (C189895)** → приёмник DBR4 (CRSF, 420 кбод), SERIAL7 (USART6).

| Pin | Цепь | MCU | Снаружи |
|---|---|---|---|
| 1 | +5V | | питание приёмника |
| 2 | RC_RX | R20 100 Ом → PC7 USART6_RX / TIM3_CH2; также вход U10 (инвертор, выход свободен) | ← TX приёмника (CRSF) |
| 3 | RC_TX | R21 100 Ом → PC6 USART6_TX | → RX приёмника (телеметрия CRSF) |
| 4 | GND | GND | |

**J10 — UART3, JST-GH 4p (C189895)** → подвес камеры (UART или SBUS-out через инвертор — инвертора на этом порту в схеме нет), SERIAL4 (USART3).

| Pin | Цепь | MCU |
|---|---|---|
| 1 | +5V | |
| 2 | USART3_TX | PD8 |
| 3 | USART3_RX | PD9 |
| 4 | GND | GND |

**J11 — UART8, JST-GH 4p (C189895)** → резерв: второй GPS / дальномер, SERIAL5 (UART8).

| Pin | Цепь | MCU |
|---|---|---|
| 1 | +5V | |
| 2 | UART8_TX | PE1 |
| 3 | UART8_RX | PE0 |
| 4 | GND | GND |

**J12A / J12B / J12C — ESC M1–M8, три ряда 1×8 шаг 2.54 мм (C9900015386)** → сигнальные провода ESC ×4 (M1–M4 для квадрокоптера; M5–M8 резерв).

| Pin ряда | J12A (ряд S) | MCU / hwdef | J12B (ряд +) | J12C (ряд −) |
|---|---|---|---|---|
| 1 | PWM1 | PB0 TIM8_CH2N PWM(1) | +5V_SERVO | GND |
| 2 | PWM2 | PB1 TIM8_CH3N PWM(2) | +5V_SERVO | GND |
| 3 | PWM3 | PA0 TIM5_CH1 PWM(3) | +5V_SERVO | GND |
| 4 | PWM4 | PA1 TIM5_CH2 PWM(4) | +5V_SERVO | GND |
| 5 | PWM5 | PA2 TIM5_CH3 PWM(5) | +5V_SERVO | GND |
| 6 | PWM6 | PA3 TIM5_CH4 PWM(6) | +5V_SERVO | GND |
| 7 | PWM7 | PD12 TIM4_CH1 PWM(7) | +5V_SERVO | GND |
| 8 | PWM8 | PD13 TIM4_CH2 PWM(8) | +5V_SERVO | GND |

**J14A / J14B / J14C — AUX 9–12, три ряда 1×4 шаг 2.54 мм (PH2.54-1×4, C42431808)** → подвес pan/tilt (AUX11/12 по пин-карте), прочие серво.

| Pin ряда | J14A (S) | MCU / hwdef | J14B (+) | J14C (−) |
|---|---|---|---|---|
| 1 | PWM9 | PD14 TIM4_CH3 PWM(9) | +5V_SERVO | GND |
| 2 | PWM10 | PD15 TIM4_CH4 PWM(10) | +5V_SERVO | GND |
| 3 | PWM11 | PE5 TIM15_CH1 PWM(11) | +5V_SERVO | GND |
| 4 | PWM12 | PE6 TIM15_CH2 PWM(12) | +5V_SERVO | GND |

**J15 — ESC TLM, 1×4 шаг 2.54 мм (C42431808)** → провод телеметрии ESC (BLHeli32/AM32), SERIAL6 (UART4).

| Pin | Цепь | MCU |
|---|---|---|
| 1 | GND | GND |
| 2 | UART4_RX | PB8 |
| 3 | UART4_TX | PB9 |
| 4 | +5V | |

**J13 — SWD, 1×4 шаг 2.54 мм (C42431808)** → ST-Link.

| Pin | Цепь | MCU |
|---|---|---|
| 1 | GND | GND |
| 2 | +3V3 | (опорное для ST-Link) |
| 3 | SWDIO | PA13 |
| 4 | SWCLK | PA14 |

NRST на разъём не выведен.

**J2 — USB-C 16p (TYPE-C-31-M-12, C165948)** → ПК (Mission Planner, DFU).

| Вывод | Цепь | Куда |
|---|---|---|
| A4/B9, B4/A9 | VBUS | D3 → +5V; R4/R5 → PE15 VBUS_SENSE; D1.5 |
| A5 | CC1 | R6 5.1 кОм → GND |
| B5 | CC2 | R7 5.1 кОм → GND |
| DP1, DP2 | USB_DP_C | D1.3 ↔ D1.4 → USB_DP → PA12 |
| DN1, DN2 | USB_DN_C | D1.1 ↔ D1.6 → USB_DN (цепь к PA11 USB_DM **не доведена**, §9) |
| GND, EH (корпус) | GND | |

**J1 — microSD TF-01A (C91145), нижняя сторона.** DAT2 → SD_D2 (PC10), CD/DAT3 → SD_D3 (PC11), CMD → SD_CMD (PD2), VDD → +3V3, CLK → SD_CK (PC12), VSS/GND → GND, DAT0 → SD_D0 (PC8), DAT1 → SD_D1 (PC9).

### Порядок подключения всей системы

По `02_ТЗ_система.md` §2, `04_План_сборки_коптера.md` и пин-карте:

1. **PWR → FC**: батарея (Q450: 4S HRB 9500) через XT60 в PWR; с PWR кабель JST-GH 6p в **J5 POWER** (5 В + аналоговые V/I) и кабель JST-GH 4p с I2C-разъёма PWR в **J7 I2C** (INA226 — основной монитор батареи, BATT_MONITOR = 21; аналог — резерв). Кабель I2C ≤ 20 см.
2. **ESC → FC**: сигнальные провода четырёх ESC в **J12A** pin 1–4 (M1–M4), земли — J12C; если ESC имеют BEC — их 5 В приходит на ряд J12B (серво-рейка, на плату не заходит, только измеряется PC4). Провод телеметрии ESC — в **J15** pin 2 (UART4_RX).
3. **GPS M10N + компас + safety switch + зуммер** — одним кабелем 10p в **J6 GPS1**. Кнопка safety switch — pin 6/7/8, зуммер — pin 8 (+3.3 В) / pin 9 (BUZZ−).
4. **Приёмник DBR4 (CRSF)** — в **J9 RC** (pin 2 = TX приёмника, pin 3 = RX приёмника).
5. **TLM-AIR (LoRa 433)** — в **J3 TELEM1** (SERIAL1); flow control не используется.
6. **SiYI HM30** — в **J4 TELEM2** (SERIAL3).
7. **Подвес** — PWM pan/tilt на **J14A** pin 3–4 (AUX11/12) или по UART на **J10 UART3** (SERIAL4).
8. **CAN-устройства** — в **J8** (если узел не крайний на шине — не ставить R13).
9. **Настройка/прошивка** — USB-C **J2** (Mission Planner), отладка — ST-Link на **J13**.
10. **Резерв** — **J11 UART8** (второй GPS/дальномер).

Порядок bring-up (04 §2, этап 5): питания → SWD и загрузчик → USB → IMU/баро/компас по одному в ArduPilot (журнал в Mission Planner) → SD-логи → CRSF → PWM на осциллограф → «Pre-arm OK» на столе; затем стенд с PWR, ESC без винтов, DBR4, GPS, TLM; потом рама Q450.

## 6. Полная пин-карта MCU

Источник: словарь `H743` в `designs/fc_stm32.py` (имя вывода символа LCSC C5271084 → цепь) + номера выводов LQFP-100 из `Quad_01_FC-STM32_Netlist.json` + функция из `hwdef.dat`. SERIAL-номера — по `SERIAL_ORDER OTG1 UART7 USART1 USART2 USART3 UART8 UART4 USART6 OTG2`.

| Вывод STM32 | № LQFP | Цепь | Функция (hwdef) / куда идёт |
|---|---|---|---|
| PA0 | 22 | PWM3 | TIM5_CH1 PWM(3) GPIO(52) → J12A.3 (M3) |
| PA1 | 23 | PWM4 | TIM5_CH2 PWM(4) GPIO(53) → J12A.4 (M4) |
| PA2 | 24 | PWM5 | TIM5_CH3 PWM(5) GPIO(54) → J12A.5 (M5) |
| PA3 | 25 | PWM6 | TIM5_CH4 PWM(6) GPIO(55) → J12A.6 (M6) |
| PA4 | 28 | BATT2_VOLTAGE_SENS | ADC1 pin 18, `HAL_BATT2_VOLT_PIN 18` — только на MCU, разъёма нет |
| PA5 | 29 | SPI1_SCK | SPI1 → U2 IIM-42652 |
| PA6 | 30 | SPI1_MISO | SPI1 ← U2 SDO |
| PA7 | 31 | BATT2_CURRENT_SENS | ADC1 pin 7, `HAL_BATT2_CURR_PIN 7` — только на MCU |
| PA8 | 67 | WS2812_DI | TIM1_CH1 PWM(13) GPIO(62) → U9 WS2812B |
| PA9 | 68 | USART1_TX | SERIAL2 GPS1 → J6.2 |
| PA10 | 69 | USART1_RX | SERIAL2 GPS1 ← J6.3 |
| PA11 | 70 | USB_DM | OTG_FS_DM OTG1 (SERIAL0) — цепь не доведена до D1/J2 (§9) |
| PA12 | 71 | USB_DP | OTG_FS_DP OTG1 → D1.4 → J2 |
| PA13 | 72 | SWDIO | JTMS-SWDIO → J13.3 |
| PA14 | 76 | SWCLK | JTCK-SWCLK → J13.4 |
| PA15 | 77 | BUZZER | TIM2_CH1 GPIO(32) ALARM → затвор Q1 |
| PB0 | 34 | PWM1 | TIM8_CH2N PWM(1) GPIO(50) → J12A.1 (M1) |
| PB1 | 35 | PWM2 | TIM8_CH3N PWM(2) GPIO(51) → J12A.2 (M2) |
| PB2 | 36 | IMU1_DRDY | INPUT (EXTI) ← U2 INT1 |
| PB3 | 89 | — | свободен |
| PB4 | 90 | — | свободен |
| PB5 | 91 | — | свободен |
| PB6 | 92 | I2C1_SCL | I2C1 (внутренняя, индекс 1) → U5 IST8310, подтяжка R2 |
| PB7 | 93 | I2C1_SDA | I2C1 → U5, подтяжка R3 |
| PB8 | 95 | UART4_RX | SERIAL6 ESC-телеметрия ← J15.2 |
| PB9 | 96 | UART4_TX | SERIAL6 → J15.3 |
| PB10 | 46 | I2C2_SCL | I2C2 (внешняя, индекс 0) → J6.4, J7.2; R14, D5 |
| PB11 | 47 | I2C2_SDA | I2C2 → J6.5, J7.3; R15, D5 |
| PB12 | 51 | BARO_CS | CS → U4 MS5611 CSB |
| PB13 | 52 | SPI2_SCK | SPI2 → U4 |
| PB14 | 53 | SPI2_MISO | SPI2 ← U4 SDO |
| PB15 | 54 | SPI2_MOSI | SPI2 → U4 SDI |
| PC0 | 15 | BATT_VOLTAGE_SENS | ADC1 pin 10, SCALE(1), `HAL_BATT_VOLT_PIN 10` ← R22 ← J5.4 |
| PC1 | 16 | BATT_CURRENT_SENS | ADC1 pin 11, `HAL_BATT_CURR_PIN 11` ← R23 ← J5.3 |
| (вывод 17) | 17 | — | не подключён (в описании схемы не задан) |
| (вывод 18) | 18 | — | не подключён (в описании схемы не задан) |
| PC4 | 32 | VDD_5V_SENS | ADC1 SCALE(3) ← R24/R25 ← +5V_SERVO |
| PC5 | 33 | RSSI_ADC | ADC1, `BOARD_RSSI_ANA_PIN 8` — только на MCU, разъёма нет |
| PC6 | 63 | USART6_TX | SERIAL7 RC, NODMA → R21 → J9.3 |
| PC7 | 64 | USART6_RX | SERIAL7 RC, NODMA ALT(1); также TIM3_CH2 RCININT PULLDOWN ← R20 ← J9.2 |
| PC8 | 65 | SD_D0 | SDMMC1_D0 → J1, подтяжка R9 |
| PC9 | 66 | SD_D1 | SDMMC1_D1 → J1, R10 |
| PC10 | 78 | SD_D2 | SDMMC1_D2 → J1, R11 |
| PC11 | 79 | SD_D3 | SDMMC1_D3 → J1, R12 |
| PC12 | 80 | SD_CK | SDMMC1_CK → J1 |
| PC13 | 7 | IMU2_DRDY | INPUT ← U3 INT1 |
| PC14 | 8 | — | свободен (LSE не ставится) |
| PC15 | 9 | IMU1_CS | CS → U2 AP_CS |
| PD0 | 81 | CAN1_RX | CAN1 ← U6 RXD |
| PD1 | 82 | CAN1_TX | CAN1 → U6 TXD |
| PD2 | 83 | SD_CMD | SDMMC1_CMD → J1, R8 |
| PD3 | 84 | CAN1_SILENT | GPIO_CAN1_SILENT OUTPUT LOW GPIO(70) → U6 S |
| PD4 | 85 | — | свободен |
| PD5 | 86 | USART2_TX | SERIAL3 TELEM2 → J4.2 |
| PD6 | 87 | USART2_RX | SERIAL3 ← J4.3 |
| PD7 | 88 | SPI1_MOSI | SPI1 → U2 SDI |
| PD8 | 55 | USART3_TX | SERIAL4 UART3 → J10.2 |
| PD9 | 56 | USART3_RX | SERIAL4 ← J10.3 |
| PD10 | 57 | SAFETY_IN | INPUT PULLUP ← J6.6 |
| PD11 | 58 | SAFETY_LED | LED_SAFETY OUTPUT LOW → R19 → J6.7 |
| PD12 | 59 | PWM7 | TIM4_CH1 PWM(7) GPIO(56) → J12A.7 |
| PD13 | 60 | PWM8 | TIM4_CH2 PWM(8) GPIO(57) → J12A.8 |
| PD14 | 61 | PWM9 | TIM4_CH3 PWM(9) GPIO(58) → J14A.1 |
| PD15 | 62 | PWM10 | TIM4_CH4 PWM(10) GPIO(59) → J14A.2 |
| PE0 | 97 | UART8_RX | SERIAL5 ← J11.3 |
| PE1 | 98 | UART8_TX | SERIAL5 → J11.2 |
| PE2 | 1 | — | свободен |
| PE3 | 2 | LED0 | OUTPUT LOW GPIO(90), синий LED1 (`HAL_GPIO_B_LED_PIN 90`), в загрузчике LED_BOOTLOADER |
| PE4 | 3 | LED1 | OUTPUT LOW GPIO(91), зелёный LED2 (`HAL_GPIO_A_LED_PIN 91`) |
| PE5 | 4 | PWM11 | TIM15_CH1 PWM(11) GPIO(60) → J14A.3 |
| PE6 | 5 | PWM12 | TIM15_CH2 PWM(12) GPIO(61) → J14A.4 |
| PE7 | 37 | UART7_RX | SERIAL1 TELEM1 ← J3.3 |
| PE8 | 38 | UART7_TX | SERIAL1 → J3.2 |
| PE9 | 39 | UART7_RTS | → J3.5 |
| PE10 | 40 | UART7_CTS | ← J3.4 |
| PE11 | 41 | IMU2_CS | CS → U3 CSB |
| PE12 | 42 | SPI4_SCK | SPI4 → U3 SCX |
| PE13 | 43 | SPI4_MISO | SPI4 ← U3 SDO |
| PE14 | 44 | SPI4_MOSI | SPI4 → U3 SDX |
| PE15 | 45 | VBUS_SENSE | VBUS INPUT OPENDRAIN ← R4/R5 (VBUS/2) |
| PH0-OSC_IN | 12 | OSC_IN | кварц Y1 8 МГц, C13 |
| PH1-OSC_OUT | 13 | OSC_OUT | кварц Y1, C14 |
| NRST | 14 | NRST | C15 100 нФ; на разъём не выведен |
| BOOT0 | 94 | BOOT0 | R1 10 кОм → GND, SW1 → +3V3 |
| VCAP1 | 48 | VCAP1 | C9 2.2 мкФ |
| VCAP2 | 73 | VCAP2 | C10 2.2 мкФ |
| VDD, VDDA, VREF+, VBAT | 6, 11, 20, 21, 27, 50, 75, 100 | +3V3 | без бусины/фильтра на VDDA |
| VSS, VSSA | 10, 19, 26, 49, 74, 99 | GND | вывод 99 в схеме на GND — назначение этого вывода у H743 проверить по даташиту |

Итого в схеме задействованы 72 порта GPIO (PA — 16, PB — 13, PC — 13, PD — 15, PE — 15) плюс PH0/PH1 под кварц; свободны PE2, PC14, выводы 17/18, PD4, PB3, PB4, PB5 (README: PB3/PB4/PB5 — SPI3 у Matek под optical flow, можно вывести на разъём EXT SPI).

## 7. Прошивка

**Что и где.** ArduPilot Copter, собственное определение платы `GEC-H743` (`firmware_ArduPilot\GEC-H743\hwdef.dat` — на основе MatekH743, наши изменения помечены «GEC»; `hwdef-bl.dat` — загрузчик, очищен от остатков Matek 22.11.2026). `chibios_hwdef.py` принял оба файла без правок. Сборка выполнена 22.11.2026 на Windows без Cygwin (gcc-arm-none-eabi 10.2.1, GNU make 4.4.1, рабочая копия `C:\ap175\src` — путь только ASCII), 1333/1333 задач, 0 ошибок. Артефакты в `firmware_ArduPilot\build_out\`:

| Файл | Размер | Назначение |
|---|---|---|
| `AP_Bootloader.bin` / `.apj` | 40 084 / 40 053 Б | загрузчик ArduPilot (USB + UART7 как serial-bootloader; умеет прошиваться с microSD) |
| `arducopter.bin` | 1 539 600 Б (≈78 % из 1920 КБ области приложения) | прошивка без загрузчика |
| `arducopter.apj` | 1 376 774 Б | прошивка через Mission Planner / загрузчик |
| `arducopter_with_bl.hex` | 4 699 220 Б | загрузчик + прошивка одним файлом (SWD/DFU) |

Board ID `APJ_BOARD_ID 1200` не зарегистрирован в ArduPilot (достаточно для прототипа). В ROMFS `arducopter.apj` остался предыдущий загрузчик (отличие только в строках PINIO1/CS) — при желании пересобрать copter после копирования нового `AP_Bootloader.bin` в `Tools/bootloaders/GEC-H743_bl.bin`. Пересборка: `cd /c/ap175/src; . ../env.sh; py -3.12 waf copter -j12` (подробно и патчи инструментов — `BUILD_REPORT.md`). На железе ничего не проверено.

**Как прошить.**
- Первый раз — ST-Link по SWD (J13: GND / +3V3 / SWDIO / SWCLK; NRST на разъёме нет) файлом `arducopter_with_bl.hex` либо только загрузчиком `AP_Bootloader.bin`, после чего `arducopter.apj` заливается из Mission Planner по USB (ТЗ §6: «первый старт — bootloader через SWD, дальше через USB»).
- Запасной путь — DFU системного загрузчика ST по USB: удерживая SW1 (BOOT0 → +3V3), подать питание/сброс; прошивать `arducopter_with_bl.hex` DFU-утилитой. Условие — исправная линия USB D− (§9).
- Из загрузчика ArduPilot: также возможна прошивка с microSD (`AP_BOOTLOADER_FLASH_FROM_SD_ENABLED 1` в hwdef-bl).

**Параметры после прошивки (только то, что есть в документах).**

| Параметр | Значение | Откуда |
|---|---|---|
| BATT_MONITOR | 21 (INA226 на внешней I2C) — по умолчанию из hwdef `HAL_BATT_MONITOR_DEFAULT 21`; аналоговый вход — резерв (BATT2) | hwdef, ТЗ §6 |
| BATT_AMP_PERVLT / BATT_SHUNT (для шунта PWR 0.5 мОм) | 40 А/В, 0.0005 | 05 §6.1; в hwdef для аналогового входа стоит `HAL_BATT_CURR_SCALE 80.0`, в Pinmap — 40.0 «пересчитать под шунт PWR» — привести к одному значению |
| HAL_BATT_VOLT_SCALE | 11.0 (делитель на PWR) | hwdef |
| SERIAL1 (TELEM1, UART7) | MAVLink 2, 57600–115200; BRD_SER1_RTSCTS = 0 (TLM-AIR без flow control) | ТЗ, 05 §6.1 |
| SERIAL2 (GPS1, USART1) | GPS (UBX), 230400 | Pinmap |
| SERIAL3 (TELEM2, USART2) | MAVLink 2, до 921600 (HM30) | ТЗ |
| SERIAL4 (UART3) | подвес | Pinmap |
| SERIAL5 (UART8) | резерв | Pinmap |
| SERIAL6 (UART4) | ESC-телеметрия | Pinmap |
| SERIAL7 (USART6) | RC; RC_PROTOCOLS = CRSF | ТЗ §6 |
| ROTATION IMU1/IMU2/компас | в hwdef `ROTATION_NONE` — задать после размещения на плате (открытый вопрос README) | README |
| WS2812 | `HAL_DEFAULT_NEOPIXEL_COUNT 1`, выход PWM(13) | hwdef |
| Safety switch | `HAL_HAVE_SAFETY_SWITCH 1` (PD10/PD11) | hwdef |
| microSD | `HAL_OS_FATFS_IO 1`, логи на карте | hwdef |

Конкретные числа SERIALn_PROTOCOL/BAUD в документах не заданы.

**Чтобы написать свою прошивку с нуля (без ArduPilot).**
- Тактирование: HSE — кварц 8 МГц на PH0/PH1 (нагрузочные 18 пФ); LSE нет — RTC только от LSI; системная частота до 480 МГц через PLL (ArduPilot: `OSCILLATOR_HZ 8000000`, системный таймер ChibiOS — TIM12, `STM32_ST_USE_TIMER 12`). Регулятор ядра — конденсаторы VCAP 2.2 мкФ уже стоят.
- Память: 2 МБ flash, 1 МБ ОЗУ; ArduPilot резервирует первые 128 КБ под загрузчик (`FLASH_RESERVE_START_KB 128`), параметры во flash-странице 14 (`STORAGE_FLASH_PAGE 14`, `HAL_STORAGE_SIZE 32768`).
- Периферия по пинам (§6): SPI1 (PA5/PA6/PD7, CS PC15, INT PB2) — IMU1; SPI4 (PE12/PE13/PE14, CS PE11, INT PC13) — IMU2; SPI2 (PB13/PB14/PB15, CS PB12) — MS5611; все датчики MODE3; I2C1 (PB6/PB7) — IST8310 0x0E; I2C2 (PB10/PB11) — внешняя шина; USART1 PA9/PA10, USART2 PD5/PD6, USART3 PD8/PD9, UART4 PB9/PB8, USART6 PC6/PC7, UART7 PE8/PE7 (+RTS PE9/CTS PE10), UART8 PE1/PE0; FDCAN1 PD0/PD1 + silent PD3; SDMMC1 4 бита PC8–PC12 + PD2; USB OTG_FS PA11/PA12, VBUS sense PE15; SWD PA13/PA14.
- Таймеры выходов: TIM8 CH2N/CH3N (PB0/PB1), TIM5 CH1–4 (PA0–PA3), TIM4 CH1–4 (PD12–PD15), TIM15 CH1/2 (PE5/PE6) — все с DMA (DShot возможен на всех 12 выходах; DMA-каналы делятся с TX UART-ов, как у Matek H743); TIM1_CH1 (PA8) — WS2812; TIM2_CH1 (PA15) — зуммер; TIM3_CH2 (PC7) — захват PPM на RC-входе.
- ADC1: PC0 (канал 10) — напряжение батареи, PC1 (11) — ток, PA4 (18) / PA7 (7) — BATT2 (без разъёма), PC5 (8) — RSSI (без разъёма), PC4 — серво-рейка через делитель 3:1 (20 кОм/10 кОм).
- Ограничения: USART6 без DMA (NODMA в hwdef); SPI1 и SPI4 не должны делить DMA (`DMA_NOSHARE SPI1* SPI4*`); LED0/LED1 активны низким уровнем; SAFETY_IN с внутренней подтяжкой; CAN silent-пин должен быть низким для работы трансивера; инвертор SBUS не включён в тракт (только CRSF/PPM без перемычки); NRST недоступен снаружи.

## 8. Плата

| Параметр | Значение | Источник |
|---|---|---|
| Размер | 50 × 50 мм | ТЗ §5, `BOARD["size_mm"]` |
| Отверстия | 4 × Ø3.0 мм в точках (9.75; 9.75), (40.25; 9.75), (9.75; 40.25), (40.25; 40.25) — шаг 30.5 × 30.5 мм (стандарт FC) | `BOARD["holes"]` |
| Слои | 4; по ТЗ: L2 — земля, L3 — питание (3.3 В / 5 В). В проекте заданы заливки GND (слой 15) и +3V3 (слой 16) по полю 0.5…49.5 мм; физический стек (JLC04161H-7628) через API не задаётся — выбрать в GUI (Ctrl+L) | ТЗ §4, `BOARD["pours"]`, 05 §6.2 |
| Правила DRC (записаны в проект) | дорожка 0.152 мм, зазор 0.152 мм, via 0.3/0.6 мм, до края 0.5 мм | 05 §6.2 |
| Ширины питания | +5V 0.6; +5V_PWR 0.8; +5V_SERVO 1.0; VBUS 0.5; +3V3 и +3V3S 0.4; USB D± 0.2 мм | `BOARD["route"]` |
| Масса | ≤ 15 г без кабелей (требование ТЗ, не проверено) | ТЗ §5 |
| Компонентов / цепей | 96 / 102 | README, Netlist |

**Зонирование (расстановка v2 от 24.11.2026, координаты `PLACE` в `fc_stm32.py`, ось Y — как в описании: «верх» = y ≈ 0).**
- Верхняя кромка (y 1.5–6.6): гребёнки J12A/B/C (x 14), J14A/B/C (x 28.8), J15 (x 38.5), делитель R24/R25. Под ними внутри платы — JST-GH J9 RC (16.4; 11.3), J10 UART3 (25.4; 11.3), J11 UART8 (36.6; 15.3) и инвертор U10 с R19–R21 (кабели выходят поверх J12/J14 через верхнюю кромку).
- Левая кромка: J13 SWD (3; 6), USB-C J2 (3.5; 18) с R6/R7/D1, J4 TELEM2 (3.6; 29.5), делитель VBUS R4/R5. Ниже — колонка питания: D2, D3, D4, C23, C24 (x 7–10, y 30–37), LDO MCU U7 с C25, C28 в левом нижнем углу (6; 45.6), SW1 BOOT / R1 / C15 рядом.
- Центр: U1 (25; 27) с развязками C1–C12 по сторонам, кварц Y1 (11.5; 25) слева от MCU; LDO датчиков U8 с C26/C27 — выше MCU (20; 16.2).
- Нижняя полоса датчиков (y ≈ 41): U4 MS5611 (15.8), U2 IIM-42652 (20.9), U3 BMI270 (25.6), U5 IST8310 (29.9) с развязками, подтяжки I2C1 R2/R3. На нижней стороне под ними — microSD J1 (25; 41.8) и R8–R11 (слой 2).
- Нижняя кромка (y 46.5): J5 POWER (18), J3 TELEM1 (29.5), J8 CAN (40.1, внутри правого нижнего угла), трансивер U6 с R13/C29 в углу (47; 41–48.5).
- Правая кромка: J7 I2C (46.6; 16.1), J6 GPS1 (46.6; 29). Правая колонка 0603 (x 37–39.6): D5, LED1/LED2 с R16/R17, R14/R15, U9 WS2812, C30, R18, R22/R23, C31/C32.

**Состояние трассировки (README, 05 §6.2, 24.11.2026 15:55, итерация 1 / v3 с rip-up).** Трассировано своим трассировщиком через API (A* на сетке 0.1 мм, 2 сигнальных слоя): 1515 дорожек, 220 via. **Не разведено 345 связей, цепей с разрывом 58** — в основном выводы LQFP-100 посередине сторон и зависящие от них цепи; среди разорванных: +3V3, UART4_TX, BOOT0, I2C1_SDA, CAN1_SILENT, SD_CMD, CAN1_TX, SD_D2, BUZZER, VCAP2, SWDIO, USB_DP. DRC через API (`pcb_Drc.check`) на этих платах зависает, пересчёт заливок (`rebuildCopperRegion`) падает — в README: «Check DRC в GUI: 1».

**Что не сделано:**
- дотрассировать 58 разорванных цепей в GUI;
- DRC (strict) и пересчёт заливок (Shift+B) в GUI;
- задать физический стек 4 слоёв (Ctrl+L), подтвердить слои земли/питания;
- обновить `_PCB.pdf` разведённой платы из GUI;
- решить проблему размещения разъёмов: 11 JST-GH (8.6/11/16 × 5.6 мм) на кромки 50 × 50 с отверстиями 30.5 мм не помещаются — на кромках только J2/J4, J6/J7, J5/J3; J8 в углу, J9/J10/J11 внутри. Варианты: JST SH 1.0 мм или плата больше;
- чек-лист `pcb-layout-guide\references\04_checklist.md` и JLC DFM перед заказом (ТЗ системы §10, план §2 этап 3).

## 9. Известные расхождения и что проверить перед заказом

Из README «Расхождения и заметки», ТЗ §7, 05 §6.1 и сверки описания схемы с нетлистом:

1. ~~**USB D− не доведён до MCU.**~~ **Исправлено 25.11.2026:** вывод 6 USBLC6 (D1) и PA11 — одна цепь `USB_DM`; плата перетрассирована.
2. **SBUS-инвертор не в тракте** — решено 25.11.2026 оставить как есть: hwdef GEC-H743 не предусматривает отдельного входа для инвертированного сигнала (PC7 = USART6_RX ALT(1) + TIM3_CH2 RCININT — единственный RC-вход), а STM32H7 инвертирует RX программно (`SERIAL7_OPTIONS = 1`). U10 можно не ставить (DNP).
3. **Кварц.** В BOM.md стоял C70579 (это 48 МГц) — в схеме заменён на C42448903 (8 МГц, CL 12 пФ), нагрузочные 18 пФ. Проверить расчёт нагрузочных для CL 12 пФ по даташиту кварца.
4. **Неиспользуемые входы.** RSSI_ADC (PC5), BATT2 (PA4/PA7) объявлены в hwdef, но в схеме подключены только к MCU — на разъёмы не выведены; UART7 RTS/CTS выведены на J3, но TLM-AIR их не использует. Свободны PE2, PC14, PD4, PB3, PB4, PB5, выводы 17/18.
5. **IST8310.** Адрес 0x0E (CAD1 → VDD, CAD0 → GND), RSTN → VDD; конденсатор C22 220 нФ на выводе C1 — проверить по даташиту. Подтяжки I2C1 (R2/R3) к +3V3, а датчик питается от +3V3S — убедиться, что это допустимо, или перевести подтяжки на +3V3S.
6. **CAN-терминатор** R13 120 Ом впаян постоянно (в ТЗ — перемычкой): для не-крайнего узла не ставить R13 при сборке.
7. **Зуммер.** BUZZ− выведен на GPS1 pin 9, «+» — pin 8 (+3V3). В ТЗ/BOM — «зуммер 5 В пассивный на разъёме BUZZ 2p»; в схеме отдельного разъёма нет, и питание зуммера через J6 — 3.3 В. Решить: достаточно ли 3.3 В, нужен ли отдельный разъём BUZZ на +5V.
8. **VDDA/VREF+** подключены к +3V3 напрямую без бусины/фильтра — при необходимости добавить L/FB.
9. **Защита портов.** ТЗ F17 требует ESD на USB, RC, I2C и UART-разъёмах и защиту серво-рейки. В схеме ESD только на USB (D1) и I2C2 (D5); на RC, TELEM1/2, GPS UART, UART3/UART8 защиты нет; на +5V_SERVO — только делитель, без диода/стабилитрона и TVS.
10. **NRST** не выведен ни на разъём SWD, ни на тест-точку (Pinmap обещал тест-точку). Тест-точки на питаниях (ТЗ F20) в описании схемы отсутствуют.
11. ~~**Серво-рейка / гребёнки.**~~ **Исправлено 25.11.2026:** J12A/B/C — штыревые PZ254V-11-08P (C492407); J14/J15/J13 — штыревые PH2.54 (C42431808), как и были.
12. **Размещение.** 11 разъёмов JST-GH не помещаются на кромки 50 × 50 — J8 в углу, J9/J10/J11 внутри платы под гребёнками (кабели поверх J12/J14). ТЗ §5 требовало «IMU в центре, USB и microSD с одного края» — датчики стоят полосой у нижней кромки (y ≈ 41), USB слева, microSD на нижней стороне с выходом вниз. Решение по виброразвязке (демпферы платы или отдельная установка IMU) по ТЗ §7 должно быть принято до разводки — в документах не зафиксировано.
13. **Ротации IMU/компаса** (`ROTATION_*`) задать в hwdef после окончательного размещения и пересобрать прошивку.
14. **Масштаб тока** (параметр прошивки, в схеме не правится). hwdef: `HAL_BATT_CURR_SCALE 80.0`; шунт PWR в схеме — заглушка 0.5 мОм → 40 А/В; при заказе PWR с 0.25 мОм (C55591764) → 80 А/В, тогда hwdef верен. Перед сборкой прошивки привести к реальному шунту PWR.
15. **ТЗ vs схема по мелочам:** нет кварца 32.768 кГц (RTC от LSI — решение Pinmap); обе LDO — XC6220B331MR-G (ТЗ хотело для датчиков отдельный малошумящий — «либо второй малошумящий LDO» в BOM); WS2812B-2020 питается 5 В, а DI управляется 3.3 В с PA8 — проверить допустимый уровень VIH по даташиту; вывод 99 MCU в схеме на GND — проверить его назначение у STM32H743 LQFP-100 по даташиту.
16. **Перед заказом (ТЗ §7, система §10):** перепроверить остатки LCSC по каждой строке BOM в день заказа (STM32H743VIT6 C5271084 — 686 шт. на 21.11.2026; IIM-42652 — заменить на ICM-42688-P, если появится); JST-GH C133065/C2683602/C189895 — наличие и footprint; DRC по пресету 4 слоя, чек-лист `pcb-layout-guide`; `APJ_BOARD_ID 1200` зарегистрировать перед выпуском.
17. **Экспорты устарели:** `_PCB.pdf` и `Netlist.json` — от 23.11 (до трассировки); `_PCB_2D.png` — 24.11; схема PNG, STEP и 3D PNG — 25.11. Актуальная разводка — в `.epro2`.
