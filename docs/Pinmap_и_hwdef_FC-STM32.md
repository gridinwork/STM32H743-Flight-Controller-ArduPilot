# 08. FC-STM32: пин-карта STM32H743VIT6 и черновик hwdef ArduPilot

Версия 0.1, 2026-11-21. Эталон: `libraries/AP_HAL_ChibiOS/hwdef/MatekH743/hwdef.dat` (ArduPilot master, скачан 21.11.2026, копия в `00 - ТЗ и система\research\`). Наши отличия от Matek помечены **(наше)**. Формат — по `electronics\references\01`: пин ↔ сигнал прошивки ↔ узел схемы. Проверено в исходниках ArduPilot master 21.11.2026: `AP_InertialSensor_Invensensev3.cpp` содержит `INV3_ID_IIM42652 0x6f` (IIM-42652 поддержан тем же драйвером, что ICM-42688), драйвер `AP_InertialSensor_BMI270.cpp` существует.

## 1. Назначение пинов

| Пин | Сигнал hwdef | Узел схемы | Примечание |
|---|---|---|---|
| **Отладка / USB** | | | |
| PA11 / PA12 | OTG_FS_DM / DP | USB-C, ESD-матрица | USB 2.0 FS |
| PA13 / PA14 | SWDIO / SWCLK | SWD 4-pin 2.54 мм | + NRST тест-точка |
| PE15 **(наше)** | VBUS_SENSE (делитель 2:1 → 3.3 В) | USB-C VBUS | `PE15 VBUS INPUT OPENDRAIN` |
| **IMU1 IIM-42652 (SPI1)** | | | |
| PA5 / PA6 / PD7 | SPI1_SCK / MISO / MOSI | U_IMU1 | 16 МГц, MODE3 |
| PC15 | IMU1_CS | U_IMU1 CS | |
| PB2 **(наше)** | IMU1_DRDY (EXTI) | U_IMU1 INT1 | `PB2 IMU1_DRDY INPUT` |
| **IMU2 BMI270 (SPI4)** | | | |
| PE12 / PE13 / PE14 | SPI4_SCK / MISO / MOSI | U_IMU2 | 10 МГц |
| PE11 | IMU2_CS | U_IMU2 CS | |
| PC13 **(наше)** | IMU2_DRDY | U_IMU2 INT1 | у Matek PC13 = IMU3_CS, третьего IMU у нас нет |
| **Барометр MS5611 (SPI2) (наше)** | | | |
| PB13 / PB14 / PB15 | SPI2_SCK / MISO / MOSI | U_BARO | у Matek SPI2 = OSD; OSD у нас нет |
| PB12 | BARO_CS | U_BARO CS | `SPIDEV ms5611 SPI2 DEVID2 BARO_CS MODE3 10*MHZ 10*MHZ`, `BARO MS5611 SPI:ms5611` |
| **I2C** | | | |
| PB6 / PB7 | I2C1_SCL / SDA | IST8310 (0x0E) на плате | внутренняя шина, подтяжки 2.2 кОм |
| PB10 / PB11 | I2C2_SCL / SDA | разъём GPS1 (компас GPS 0x0D/0x0E), разъём I2C, INA226 модуля PWR (0x40) | внешняя шина, подтяжки 2.2 кОм, ESD; `I2C_ORDER I2C2 I2C1` как у Matek (внешняя = I2C:0) |
| **UART** (SERIAL_ORDER OTG1 UART7 USART1 USART2 USART3 UART8 UART4 USART6 OTG2) | | | |
| PE8 / PE7 / PE9 / PE10 | UART7 TX / RX / RTS / CTS | **TELEM1** JST-GH 6p → TLM-AIR | SERIAL1, MAVLink2, flow control для mLRS |
| PA9 / PA10 | USART1 TX / RX | **GPS1** JST-GH 10p | SERIAL2, UBX 230400 |
| PD5 / PD6 | USART2 TX / RX | **TELEM2** JST-GH 6p → SiYI HM30 / компаньон | SERIAL3, до 921600 |
| PD8 / PD9 | USART3 TX / RX | **UART3** JST-GH 4p: подвес (SBUS-out через инвертор или UART) | SERIAL4 |
| PE1 / PE0 | UART8 TX / RX | **UART8** JST-GH 4p: второй GPS / дальномер | SERIAL5 |
| PB9 / PB8 | UART4 TX / RX | **ESC-телеметрия** (RX) на ESC-гребёнке | SERIAL6 |
| PC6 / PC7 | USART6 TX / RX (NODMA), PC7 также TIM3_CH2 RCININT | **RC** JST-GH 4p: CRSF (DBR4); SBUS/PPM через инвертор 74LVC1G14 на PC7 | SERIAL7; RC_PROTOCOLS CRSF |
| **CAN** | | | |
| PD0 / PD1 | CAN1_RX / TX | TJA1051T/3, JST-GH 4p CAN1, терминатор 120 Ом перемычкой | DroneCAN |
| PD3 | GPIO_CAN1_SILENT (GPIO 70) | TJA1051 pin S | как у Matek |
| **PWM / DShot** (TIM8, TIM5, TIM4, TIM15 — все с DMA) | | | |
| PB0 / PB1 | TIM8_CH2N / CH3N → M1 / M2 | ESC-гребёнка 1–2 | группа TIM8 |
| PA0 / PA1 / PA2 / PA3 | TIM5_CH1..4 → M3..M6 | ESC-гребёнка 3–6 | группа TIM5 |
| PD12 / PD13 / PD14 / PD15 | TIM4_CH1..4 → M7..M10 | ESC-гребёнка 7–8, AUX 9–10 | группа TIM4 |
| PE5 / PE6 | TIM15_CH1 / CH2 → AUX 11–12 | подвес pan/tilt | группа TIM15 |
| PA8 | TIM1_CH1 → WS2812 (PWM 13) | RGB-LED на плате + выход на внешнюю ленту | `define HAL_DEFAULT_NEOPIXEL_COUNT 1` |
| PA15 | TIM2_CH1 ALARM | ключ зуммера (2N7002 → разъём BUZZ 5 В) | |
| **microSD** | | | |
| PC8 / PC9 / PC10 / PC11 | SDMMC1_D0..D3 | TF-01A | 4 бит |
| PC12 / PD2 | SDMMC1_CK / CMD | TF-01A | подтяжки 10 кОм на CMD/D0–D3 |
| **ADC** | | | |
| PC0 | BATT_VOLTAGE_SENS (pin 10) | разъём POWER pin V | делитель на PWR, HAL_BATT_VOLT_SCALE 11.0 |
| PC1 | BATT_CURRENT_SENS (pin 11) | разъём POWER pin I | HAL_BATT_CURR_SCALE по шунту PWR |
| PA4 / PA7 | BATT2_VOLTAGE / CURRENT (18 / 7) | резерв (второй POWER) | |
| PC5 | RSSI_ADC (pin 8) | RC-разъём, аналоговый RSSI | |
| PC4 **(наше)** | SERVO_RAIL_SENS (3:1) | серво-рейка ESC-гребёнки | вместо PRESSURE_SENS у Matek |
| **GPIO** | | | |
| PE3 / PE4 | LED0 (синий) / LED1 (зелёный) | светодиоды | |
| PD10 **(наше)** | SAFETY_IN (GPIO 81, INPUT PULLUP) | разъём GPS1 pin safety switch | `define HAL_HAVE_SAFETY_SWITCH 1` |
| PD11 **(наше)** | SAFETY_LED (GPIO 82) | разъём GPS1 pin LED | |
| PH0 / PH1 | OSC_IN / OSC_OUT | кварц 8 МГц, 2 × 12–20 пФ по CL | `OSCILLATOR_HZ 8000000` |
| PC14 / PC15 | — | PC15 занят IMU1_CS, LSE-кварц не ставим | RTC от LSI |
| VCAP1 / VCAP2 | — | 2 × 2.2 µF low-ESR | обязательно (урок платы 160) |
| BOOT0 | — | 10 кОм на GND, кнопка на 3.3 В (DFU) | |

Свободны (резерв на будущее): PB3/PB4/PB5 (SPI3 у Matek под optical flow — можно вывести на разъём EXT SPI), PD4, PE2, PA9/PA10 заняты. Всего использовано ≈ 70 из 82 I/O.

## 2. Разъёмы (JST-GH 1.25 мм, нумерация по Pixhawk Connector Standard)

| Разъём | Пины | Примечание |
|---|---|---|
| POWER (6p) | 1 VCC 5V, 2 VCC 5V, 3 CURRENT, 4 VOLTAGE, 5 GND, 6 GND | стандарт Pixhawk; I2C INA226 — отдельный 4p I2C на PWR → FC I2C |
| TELEM1 (6p) | 1 VCC 5V, 2 TX, 3 RX, 4 CTS, 5 RTS, 6 GND | TX = выход FC |
| TELEM2 (6p) | то же | |
| GPS1 (10p) | 1 VCC 5V, 2 TX, 3 RX, 4 SCL, 5 SDA, 6 SAFETY_SW, 7 SAFETY_LED, 8 VDD3V3, 9 BUZZER-, 10 GND | как Pixhawk 4 GPS MODULE |
| I2C (4p) | 1 VCC 5V, 2 SCL, 3 SDA, 4 GND | ESD |
| CAN1 (4p) | 1 VCC 5V, 2 CAN_H, 3 CAN_L, 4 GND | |
| RC (4p) | 1 VCC 5V, 2 RX (CRSF/SBUS), 3 TX (CRSF телеметрия), 4 GND | + RSSI на UART3-разъёме по желанию |
| UART3, UART8 (4p) | 1 VCC 5V, 2 TX, 3 RX, 4 GND | |
| ESC (2.54 мм 3×8 + 3×4 AUX) | ряд S: M1–M8 / A9–A12; ряд +: серво-рейка 5 В; ряд −: GND | ESC-телеметрия отдельным пином на краю |
| BUZZ (2p), SWD (4p 2.54), USB-C | | |

## 3. Черновик hwdef.dat (ключевые строки, на основе Matek H743)

```
MCU STM32H7xx STM32H743xx
APJ_BOARD_ID 1200          # выбрать свободный ID или зарегистрировать
OSCILLATOR_HZ 8000000
FLASH_SIZE_KB 2048
FLASH_RESERVE_START_KB 128
STM32_ST_USE_TIMER 12
define CH_CFG_ST_RESOLUTION 16
# USB, SWD
PA11 OTG_FS_DM OTG1
PA12 OTG_FS_DP OTG1
PA13 JTMS-SWDIO SWD
PA14 JTCK-SWCLK SWD
PE15 VBUS INPUT OPENDRAIN
# SPI1 IMU1, SPI4 IMU2, SPI2 baro
PA5 SPI1_SCK SPI1
PA6 SPI1_MISO SPI1
PD7 SPI1_MOSI SPI1
PC15 IMU1_CS CS
PB2 IMU1_DRDY INPUT
PE12 SPI4_SCK SPI4
PE13 SPI4_MISO SPI4
PE14 SPI4_MOSI SPI4
PE11 IMU2_CS CS
PC13 IMU2_DRDY INPUT
PB13 SPI2_SCK SPI2
PB14 SPI2_MISO SPI2
PB15 SPI2_MOSI SPI2
PB12 BARO_CS CS
# I2C
I2C_ORDER I2C2 I2C1
PB6 I2C1_SCL I2C1
PB7 I2C1_SDA I2C1
PB10 I2C2_SCL I2C2
PB11 I2C2_SDA I2C2
# ADC
PC0 BATT_VOLTAGE_SENS ADC1 SCALE(1)
PC1 BATT_CURRENT_SENS ADC1 SCALE(1)
PA4 BATT2_VOLTAGE_SENS ADC1 SCALE(1)
PA7 BATT2_CURRENT_SENS ADC1 SCALE(1)
PC5 RSSI_ADC ADC1
PC4 SERVO_RAIL_SENS ADC1 SCALE(3)   # проверить имя/поддержку
define HAL_BATT_MONITOR_DEFAULT 21   # INA226 на I2C внешней шины
define HAL_BATT_VOLT_PIN 10
define HAL_BATT_CURR_PIN 11
define HAL_BATT_VOLT_SCALE 11.0
define HAL_BATT_CURR_SCALE 40.0      # пересчитать под шунт PWR
# UART
SERIAL_ORDER OTG1 UART7 USART1 USART2 USART3 UART8 UART4 USART6 OTG2
PE8 UART7_TX UART7
PE7 UART7_RX UART7
PE9 UART7_RTS UART7
PE10 UART7_CTS UART7
PA9 USART1_TX USART1
PA10 USART1_RX USART1
PD5 USART2_TX USART2
PD6 USART2_RX USART2
PD8 USART3_TX USART3
PD9 USART3_RX USART3
PE1 UART8_TX UART8
PE0 UART8_RX UART8
PB9 UART4_TX UART4
PB8 UART4_RX UART4
PC6 USART6_TX USART6 NODMA
PC7 TIM3_CH2 TIM3 RCININT PULLDOWN LOW
PC7 USART6_RX USART6 NODMA ALT(1)
# CAN
PD0 CAN1_RX CAN1
PD1 CAN1_TX CAN1
PD3 GPIO_CAN1_SILENT OUTPUT PUSHPULL SPEED_LOW LOW GPIO(70)
# PWM
PB0  TIM8_CH2N TIM8  PWM(1)  GPIO(50)
PB1  TIM8_CH3N TIM8  PWM(2)  GPIO(51)
PA0  TIM5_CH1  TIM5  PWM(3)  GPIO(52)
PA1  TIM5_CH2  TIM5  PWM(4)  GPIO(53)
PA2  TIM5_CH3  TIM5  PWM(5)  GPIO(54)
PA3  TIM5_CH4  TIM5  PWM(6)  GPIO(55)
PD12 TIM4_CH1 TIM4 PWM(7) GPIO(56)
PD13 TIM4_CH2 TIM4 PWM(8) GPIO(57)
PD14 TIM4_CH3 TIM4 PWM(9) GPIO(58)
PD15 TIM4_CH4 TIM4 PWM(10) GPIO(59)
PE5  TIM15_CH1 TIM15 PWM(11) GPIO(60)
PE6  TIM15_CH2 TIM15 PWM(12) GPIO(61)
PA8  TIM1_CH1 TIM1 PWM(13) GPIO(62)   # WS2812
PA15 TIM2_CH1 TIM2 GPIO(32) ALARM
# SD
PC8 SDMMC1_D0 SDMMC1
PC9 SDMMC1_D1 SDMMC1
PC10 SDMMC1_D2 SDMMC1
PC11 SDMMC1_D3 SDMMC1
PC12 SDMMC1_CK SDMMC1
PD2 SDMMC1_CMD SDMMC1
define HAL_OS_FATFS_IO 1
# GPIO
PE3 LED0 OUTPUT LOW GPIO(90)
PE4 LED1 OUTPUT LOW GPIO(91)
PD10 SAFETY_IN INPUT PULLUP
PD11 SAFETY_LED OUTPUT LOW
define HAL_HAVE_SAFETY_SWITCH 1
# устройства
SPIDEV iim42652 SPI1 DEVID1 IMU1_CS MODE3 2*MHZ 16*MHZ
SPIDEV bmi270   SPI4 DEVID1 IMU2_CS MODE3 1*MHZ 10*MHZ
SPIDEV ms5611   SPI2 DEVID2 BARO_CS MODE3 10*MHZ 10*MHZ
DMA_NOSHARE SPI1* SPI4*
IMU Invensensev3 SPI:iim42652 ROTATION_NONE      # ротацию задать по ориентации корпуса на плате
IMU BMI270 SPI:bmi270 ROTATION_NONE
BARO MS5611 SPI:ms5611
COMPASS IST8310 I2C:1:0x0E false ROTATION_NONE   # внутренний, I2C1 = индекс 1 при I2C_ORDER I2C2 I2C1
define AP_COMPASS_PROBING_ENABLED 1
define HAL_DEFAULT_INS_FAST_SAMPLE 1
define HAL_DEFAULT_NEOPIXEL_COUNT 1
DMA_PRIORITY S*
define HAL_STORAGE_SIZE 32768
STORAGE_FLASH_PAGE 14
```

Что проверить при сборке прошивки (не из памяти): имя `SERVO_RAIL_SENS`/поддержка servo-rail ADC на H7; корректный синтаксис `SPIDEV` для BMI270 (в других hwdef, например KakuteH7, есть пример); `ROTATION_*` после размещения датчиков на плате; свободный `APJ_BOARD_ID`.

## 4. Расхождения с системным ТЗ

- ТЗ просило ≥ 8 ESC + ≥ 4 AUX: сделано 10 + 2 (M1–M10 с DShot, A11–A12 PWM). При необходимости M9/M10 используются как AUX.
- INA226 модуля PWR сидит на внешней шине I2C2 вместе с компасом GPS — адреса не конфликтуют (0x40 vs 0x0D/0x0E), но длина кабеля до PWR должна быть короткой (≤ 20 см) или шина 100 кГц.
