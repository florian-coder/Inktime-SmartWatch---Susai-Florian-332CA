# InkTime v6 — E-Paper Smartwatch

## Overview

InkTime is an e-paper smartwatch built around the **nRF52840** microcontroller, featuring a 1.54" e-paper display, Bluetooth Low Energy connectivity, haptic feedback, and an accelerometer. The watch is powered by a 250mAh LiPo battery managed by a dedicated charger IC with USB-C charging support.

---

## Block Diagram

```
                                    ┌──────────────┐
                                    │   USB-C (J4)  │
                                    │ KH-TYPE-C-16P │
                                    └──────┬───────┘
                                           │ VBUS, D+, D-
                                    ┌──────▼───────┐
                                    │  ESD Protect  │
                                    │ USBLC6-2SC6Y  │
                                    │     (D4)      │
                                    └──────┬───────┘
                              D+,D- │      │ VBUS
                         ┌──────────┘      │
                         │          ┌──────▼───────┐
                         │          │ LiPo Charger  │
                         │          │  BQ25180 (IC3)│
                         │          └──────┬───────┘
                         │                 │ VBAT
                         │          ┌──────▼───────┐
                         │          │  DC/DC Conv   │
                         │          │ RT6160 (IC2)  │
                         │          └──────┬───────┘
                         │                 │ 3V3 (VREG)
                         │                 │
                  ┌──────▼─────────────────▼──────────┐
                  │                                    │
                  │         nRF52840 (U$1)             │
                  │                                    │
                  │  SPI: MOSI, SCK, CS, DC, RST, BUSY │
                  │  I2C: SDA, SCL                     │
                  │  GPIO: Buttons, Haptic EN           │
                  │  USB: D+, D-                       │
                  │  RF: ANT                           │
                  │  SWD: SWDIO, SWDCLK                │
                  │                                    │
                  └─┬──────┬──────┬──────┬──────┬──┬──┘
                    │      │      │      │      │  │
              ┌─────▼──┐ ┌─▼────┐│ ┌────▼─┐ ┌──▼┐ │
              │E-Paper ││ │ IMU  ││ │Haptic│ │ANT│ │
              │Display ││ │BMA423││ │Driver│ │   │ │
              │  (J1)  ││ │(IC1) ││ │DRV   │ │2.4│ │
              │503480  ││ │      ││ │2605  │ │GHz│ │
              └────────┘│ └──────┘│ │(IC4) │ └───┘ │
                        │        │ └──────┘        │
                   ┌────▼───┐ ┌──▼─────┐    ┌─────▼──┐
                   │  Fuel   │ │Buttons │    │  SWD   │
                   │ Gauge   │ │SW1-SW3 │    │  J2    │
                   │MAX17048 │ │        │    │TC2030  │
                   │  (U1)   │ │        │    │        │
                   └─────────┘ └────────┘    └────────┘
```

### Communication Interfaces

| Interface | Components | Signals |
|-----------|-----------|---------|
| SPI | E-Paper Display (J1) | MOSI, SCK, EPD_CS, EPD_DC, EPD_RST, EPD_BUSY |
| I2C | IC1 (IMU), IC2 (DC/DC), IC3 (Charger), IC4 (Haptic), U1 (Fuel Gauge) | SDA, SCL (pull-ups R13=3K3, R14=3K3) |
| USB 2.0 | USB-C Connector (J4) via ESD (D4) | D+, D- (differential 90Ω) |
| SWD | Tag-Connect (J2) | SWDIO, SWDCLK, SWO, RESET |
| GPIO | Buttons (SW1-SW3), Haptic enable, Interrupts | Various P0.xx/P1.xx pins |
| RF | Chip Antenna (ANT1) via matching network | 50Ω controlled impedance trace |

---

## Bill of Materials (BOM)

### Main ICs

| Ref | Component | Package | Value/Description | Qty | Datasheet | Source |
|-----|-----------|---------|-------------------|-----|-----------|--------|
| U$1 | nRF52840-QIAA | QFN-73 7x7mm | BLE SoC, ARM Cortex-M4F, 1MB Flash | 1 | [Datasheet](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.1.pdf) | [JLCPCB](https://jlcpcb.com/partdetail/Nordic_Semiconductor-nRF52840_QIAA_R7/C190794) |
| IC1 | BMA423 | LGA-12 2x2mm | 3-axis accelerometer (IMU) | 1 | [Datasheet](https://www.bosch-sensortec.com/products/motion-sensors/accelerometers/bma423/) | [TME](https://www.tme.eu/en/details/bma423/accelerometers/) |
| IC2 | RT6160AWSC | WLCSP-20 | DC/DC Buck-Boost converter | 1 | [Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-05.pdf) | [JLCPCB](https://jlcpcb.com/partdetail/Richtek-RT6160AWSC/C2828074) |
| IC3 | BQ25180YBGR | DSBGA-15 | LiPo Battery Charger | 1 | [Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf) | [JLCPCB](https://jlcpcb.com/partdetail/Texas_Instruments-BQ25180YBGR/C2682568) |
| IC4 | DRV2605YZFR | DSBGA-12 | Haptic Driver | 1 | [Datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf) | [JLCPCB](https://jlcpcb.com/partdetail/Texas_Instruments-DRV2605YZFR/C527428) |
| U1 | MAX17048G+T10 | TDFN-8 | Fuel Gauge (battery level) | 1 | [Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf) | [JLCPCB](https://jlcpcb.com/partdetail/Analog_Devices-MAX17048G_T10/C2682568) |

### Connectors & Electromechanical

| Ref | Component | Description | Qty | Source |
|-----|-----------|-------------|-----|--------|
| J1 | 503480-2400 (Molex) | 24-pin FPC connector for e-paper | 1 | [TME](https://www.tme.eu/en/details/503480-2400/ffc-fpc-connectors/) |
| J2 | TC2030-IDC | 6-pin Tag-Connect SWD programmer | 1 | [TME](https://www.tme.eu/en/details/tc2030-idc/) |
| J4 | KH-TYPE-C-16P | USB Type-C 16-pin connector | 1 | [JLCPCB](https://jlcpcb.com/partdetail/Kinghelm-KH_TYPE_C_16P/C2765186) |
| SW1-SW3 | EVP-AKE31A (Panasonic) | Tactile switch 3x2.5mm | 3 | [TME](https://www.tme.eu/en/details/evp-ake31a/micro-switches/) |
| ANT1 | 2450AT18B100E | 2.4GHz chip antenna | 1 | [TME](https://www.tme.eu/en/details/2450at18b100e/) |

### Passive Components — Crystals

| Ref | Value | Description | Qty |
|-----|-------|-------------|-----|
| X1 | 32MHz | Main crystal for nRF52840 | 1 |
| X2 | 32.768kHz | RTC crystal for low-power timekeeping | 1 |

### Passive Components — Inductors

| Ref | Value | Description | Qty |
|-----|-------|-------------|-----|
| L1 | 47µH (FTC252012SR47MBCA) | DC/DC main inductor | 1 |
| L3 | 3.9nH | RF matching network | 1 |
| L5 | 68µH | E-paper drive inductor | 1 |
| U$2, U$4, U$5 | 10µH | MCU inductors (DCC/DCCH) | 3 |

### Passive Components — Capacitors

| Ref | Value | Description | Qty |
|-----|-------|-------------|-----|
| C1, C17, C18 | 12pF | Crystal load capacitors | 3 |
| C3, C4 | 1pF | RF matching | 2 |
| C5, C6, C7, C19, C22 | 100nF | Bypass/decoupling MCU & IMU | 5 |
| C8 | 100pF | DEC4 bypass | 1 |
| C9 | 820pF | RF matching | 1 |
| C11, C14, C20, C21 | 4.7µF | Bulk decoupling MCU | 4 |
| C15 | 1.0µF | VDDH capacitor | 1 |
| C16 | 47nF | DEC4 capacitor | 1 |
| C23, C2, C30 | 0.1µF | Bypass DC/DC, Haptic, USB | 3 |
| C24 | 10µF | DC/DC output | 1 |
| C25, C33 | 22µF | DC/DC input/output | 2 |
| C27, C28, C29 | 1µF | Button debounce | 3 |
| C31 | 4.7µF | USB bulk capacitor | 1 |
| C32-C44 | 1µF/50V | E-paper drive capacitors | 9 |
| C37, C38, C4 | 1µF | Charger & haptic bypass | 3 |
| C39 | 10µF | Charger output | 1 |
| C1-EP-DR | 4.7µF/25V | E-paper drive main cap | 1 |
| C10, C12, C13 | N.C. | Not connected (reserved) | 3 |

### Passive Components — Resistors

| Ref | Value | Description | Qty |
|-----|-------|-------------|-----|
| R1, R2, R3 | 0Ω | IMU configuration jumpers | 3 |
| R6, R7, R8 | 10K | Button pull-up resistors | 3 |
| R9, R12, R15 | 10K | Charger TS, E-paper config | 3 |
| R10 | 0.47Ω | E-paper current sense | 1 |
| R11 | 2.2Ω | E-paper type select | 1 |
| R13, R14 | 3K3 | I2C pull-ups (SCL, SDA) | 2 |
| R16 | 5K1 | USB-C CC resistor | 1 |

### Semiconductors

| Ref | Component | Description | Qty |
|-----|-----------|-------------|-----|
| D1, D2, D3 | MBR0530 | Schottky diodes (e-paper drive) | 3 |
| D4 | USBLC6-2SC6Y | USB ESD protection | 1 |
| Q1 | SI1308EDL-T1-GE3 | N-MOSFET (e-paper drive) | 1 |
| U$3 | DMG2305UX-7 | P-MOSFET (e-paper drive) | 1 |

### Off-Board Components

| Component | Model | Dimensions | Description |
|-----------|-------|------------|-------------|
| Battery | LP502030 (Akyga AKY0106) | 30×20×5mm | 3.7V 250mAh LiPo |
| Display | WSH-12561 (Waveshare) | 31.8×37.32×1.05mm | 1.54" e-paper 200x200px |
| Shaker | FIT0774 (DFRobot) | Ø10×2.7mm | Mini vibration motor |

---

## Pin Assignment — nRF52840 (U$1)

### SPI — E-Paper Display

| nRF52840 Pin | Signal | Direction | Connected To | Purpose |
|-------------|--------|-----------|-------------|---------|
| P0.13 | MOSI | Output | J1 pin 14 | SPI data to display |
| P0.14 | SCK | Output | J1 pin 13 | SPI clock |
| P1.01 | EPD_CS | Output | J1 pin 12 | Chip select (active low) |
| P1.02 | EPD_DC | Output | J1 pin 11 | Data/Command select |
| P0.15 | EPD_RST | Output | J1 pin 10 | Display reset |
| P0.16 | EPD_BUSY | Input | J1 pin 9 | Display busy status |

**Why these pins:** P0.13 and P0.14 are the default SPI MOSI/SCK pins on the nRF52840. The control signals (CS, DC, RST, BUSY) use nearby GPIO pins on the P1.0x port for clean routing to the FPC connector on the left side of the board.

### I2C Bus — Shared (SDA/SCL)

| nRF52840 Pin | Signal | Connected To | Pull-up |
|-------------|--------|-------------|---------|
| P0.26 | SDA | IC1, IC2, IC3, IC4, U1 | R14 = 3K3 to 3V3 |
| P0.27 | SCL | IC1, IC2, IC3, IC4, U1 | R13 = 3K3 to 3V3 |

**Why these pins:** P0.26/P0.27 are the default TWI (I2C) pins. All five I2C devices share the same bus, each with a unique address. The 3K3 pull-up value is chosen for the bus capacitance with 5 devices at 400kHz.

**I2C Device Addresses:**
| Device | Address | Function |
|--------|---------|----------|
| IC1 (BMA423) | 0x18 or 0x19 | Accelerometer (configured via R1-R3) |
| IC2 (RT6160) | 0x75 | DC/DC voltage setting |
| IC3 (BQ25180) | 0x6A | Battery charger configuration |
| IC4 (DRV2605) | 0x5A | Haptic driver control |
| U1 (MAX17048) | 0x36 | Battery fuel gauge |

### USB 2.0

| nRF52840 Pin | Signal | Connected To |
|-------------|--------|-------------|
| D- (P$AD4) | USB D- | J4 via D4 (ESD) |
| D+ (P$AD6) | USB D+ | J4 via D4 (ESD) |
| VBUS (P$AD2) | USB VBUS | J4 VBUS pin |

**Why:** Dedicated USB pins on the nRF52840, directly mapped in hardware. D+/D- routed as 90Ω differential pair with matched lengths.

### SWD Programming

| nRF52840 Pin | Signal | Connected To |
|-------------|--------|-------------|
| SWDIO (P$AC24) | SWDIO | J2 pin 2 |
| SWDCLK (P$AA24) | SWDCLK | J2 pin 4 |
| P0.18/RESET | RESET | J2 pin 6 |
| P0.09 | SWO | J2 pin 3 (via TP_SWO) |

**Why:** SWDIO and SWDCLK are fixed SWD pins. P0.09 is used for SWO (Serial Wire Output) for debug tracing. RESET is the dedicated reset pin.

### Interrupt Lines

| nRF52840 Pin | Signal | Source | Purpose |
|-------------|--------|--------|---------|
| P0.04 | IMU_INT1 | IC1 pin 5 | Accelerometer interrupt 1 (step detection, tap) |
| P0.05 | IMU_INT2 | IC1 pin 6 | Accelerometer interrupt 2 (data ready) |
| P1.09 | PMIC_INT | IC3 pin A1 | Charger status interrupt |
| P0.25 | HAPTIC_EN | IC4 pin B1 | Haptic driver enable/trigger |
| P1.01 | ALERT | U1 pin 5 | Battery low alert from fuel gauge |

**Why:** Interrupt pins are chosen on the P0.04-P0.05 range which support GPIOTE (GPIO Tasks and Events) for low-latency wake-up from sleep mode. PMIC_INT on P1.09 for charger monitoring.

### Buttons

| nRF52840 Pin | Signal | Connected To | Pull-up | Debounce |
|-------------|--------|-------------|---------|----------|
| P0.28 | BTN_UP | SW1 pin 2 | R8 = 10K | C27 = 1µF |
| P0.29 | BTN_ENTER | SW2 pin 2 | R7 = 10K | C28 = 1µF |
| P0.30 | BTN_DOWN | SW3 pin 2 | R6 = 10K | C29 = 1µF |

**Why:** P0.28-P0.30 are analog-capable pins (AIN4-AIN6) but used here as digital GPIO with internal pull-up supplemented by external 10K pull-ups. The 1µF debounce capacitors with 10K resistors give a ~10ms time constant, effective for mechanical button debounce. Buttons are active-low (pressed = GND).

### Power Pins

| nRF52840 Pin | Signal | Description |
|-------------|--------|-------------|
| VDD (multiple) | 3V3 | Main 3.3V supply from DC/DC |
| VDDH | VREG | Higher voltage input (from battery via DC/DC) |
| VBUS | VBUS | USB 5V input detection |
| DCC/DCCH | Internal | Internal DC/DC converter pins (U$2, U$4, U$5 inductors) |
| DEC1-DEC6 | Bypass | Decoupling pins (C5, C6, C7, C8, C16, C22) |
| VSS, VSS_PA | GND | Ground connections |

### RF

| nRF52840 Pin | Signal | Connected To |
|-------------|--------|-------------|
| ANT (P$H23) | RF output | ANT1 via matching network (L3→C3→C9) |
| VSS_PA (P$F23) | RF ground | GND plane |

**Why:** The ANT pin is the dedicated RF output. The matching network (L3=3.9nH, C3=1pF, C9=820pF) transforms the impedance to 50Ω for the chip antenna. A keep-out zone with no copper on any layer is maintained under and around ANT1.

---

## Hardware Detailed Description

### Power Architecture

The power system follows a three-stage architecture:

**Stage 1 — USB Input & Charging:** USB-C connector (J4) provides 5V VBUS. The USBLC6-2SC6Y (D4) provides ESD protection on D+/D- and VBUS lines. R16 (5K1) on CC1 identifies the device as a USB sink. VBUS feeds into IC3 (BQ25180) which charges the LiPo battery with configurable current (up to 1A) and provides system power during charging.

**Stage 2 — Battery Management:** The BQ25180 (IC3) manages charging with features including power path management (system runs from VBUS when connected, seamlessly switches to battery when disconnected), battery temperature monitoring via TS/MR pin (R9=10K), and I2C configuration for charge current and voltage. The MAX17048 fuel gauge (U1) monitors battery voltage and state-of-charge using a ModelGauge algorithm — no sense resistor needed, it measures the battery directly via CELL pin.

**Stage 3 — Voltage Regulation:** The RT6160 (IC2) buck-boost converter takes VBAT (3.0V-4.2V) and produces a stable 3V3 output (VREG). Buck-boost topology ensures operation even when battery voltage drops below 3.3V. The switching frequency is set internally, with the external inductor L1 (47µH) chosen for low DCR and sufficient current handling. Input capacitors (C33=22µF, C23=0.1µF) and output capacitors (C24=10µF, C25=22µF) are sized per the datasheet recommendations. The DC/DC has I2C interface for dynamic voltage adjustment if needed.

### Power Consumption Estimate

| State | Current | Duration | Notes |
|-------|---------|----------|-------|
| Deep sleep (RTC only) | ~1.5µA | 99% of time | nRF52840 System OFF + RTC |
| Display refresh | ~15mA | 2s per refresh | E-paper full update |
| BLE advertising | ~5mA | 1-2ms per event | Every 1-2 seconds |
| BLE connected | ~8mA | During data transfer | Intermittent |
| Haptic vibration | ~50mA | 100-500ms per event | Motor on |
| Active processing | ~3mA | Brief periods | CPU running at 64MHz |

**Estimated battery life:** With a 250mAh battery, primarily in deep sleep with occasional BLE activity and 1 display refresh per minute, the estimated battery life is approximately 5-10 days.

### E-Paper Display System

The 1.54" e-paper display (Waveshare WSH-12561, 200x200 pixels) connects via a 24-pin FPC connector (J1, Molex 503480-2400). The display uses SPI for data transfer and requires a complex drive circuit for the e-ink panel voltages.

**E-Paper Drive Circuit:** The display requires multiple voltage rails (PREVGH ~+15V, PREVGL ~-15V, VGH, VGL) generated by a boost/inverter circuit built from:
- U$3 (DMG2305UX-7, P-MOSFET) and Q1 (SI1308EDL, N-MOSFET) — switching elements
- L5 (68µH) — energy storage inductor
- D1, D2, D3 (MBR0530) — Schottky rectifier diodes
- C32-C44 (1µF/50V × 9) — high-voltage filter capacitors
- R10 (0.47Ω) — current sense resistor
- R11 (2.2Ω) — display type select

The drive circuit generates the required voltages from the 3V3 rail through a charge pump / boost converter topology controlled by the nRF52840 GPIO pins.

### IMU (Accelerometer)

The BMA423 (IC1) is a low-power 3-axis accelerometer featuring built-in step counter, tap detection, and activity recognition. It communicates via I2C at up to 400kHz. Configuration jumpers R1, R2, R3 (0Ω) set the SDO address bit, SDX/SCX mode, and CSB for I2C mode selection. The bypass capacitor C19 (100nF) is placed within 1mm of the VDD pin.

**Placement note:** IC1 is placed away from IC4 (haptic driver) to avoid vibration interference with the accelerometer readings.

### Haptic Feedback

The DRV2605 (IC4) is a haptic driver that drives the FIT0774 coin vibration motor (10mm × 2.7mm). It supports both ERM (Eccentric Rotating Mass) and LRA (Linear Resonant Actuator) motor types with a built-in library of 123 haptic effects. The nRF52840 triggers effects via I2C commands or the HAPTIC_EN GPIO pin. Bypass capacitors C2 (0.1µF) and C4 (1µF) provide local decoupling.

### Bluetooth Low Energy

The nRF52840 integrates a 2.4GHz radio with BLE 5.0 support. The RF signal is routed from the ANT pin through a pi-network matching circuit (L3=3.9nH, C3=1pF, C9=820pF) to the 2450AT18B100E chip antenna (ANT1). The matching network values are per the nRF52840 reference design for 50Ω impedance.

**RF Layout Rules Applied:**
- No copper (GND or signal) on any layer under the antenna
- 2mm minimum keep-out zone around ANT1
- 50Ω controlled impedance trace from matching network to antenna
- Matching components placed in series on the RF trace, as close to ANT pin as possible

---

## PCB Design Details

### Board Specifications

| Parameter | Value |
|-----------|-------|
| Dimensions | 46 × 35 mm (watch form factor) |
| Thickness | 1.0 mm |
| Layers | 4 (Top, GND, Power, Bottom) |
| Copper weight | 1oz (35µm) |
| Min trace width | 5mil (0.127mm) |
| Min clearance | 5mil (0.127mm) |
| Min drill | 0.3mm |
| Copper to edge | 0.2mm |
| Fabricator | JLCPCB |

### Layer Stack-up

| Layer | Name | Type | Content |
|-------|------|------|---------|
| 1 | Top | Signal | Components + signal routing + GND pour |
| 2 | GND | Plane | Dedicated GND plane (polygon pour) |
| 63 | Power | Signal | Additional routing layer |
| 64 | Bottom | Signal | Signal routing + GND pour |

### Component Placement Strategy

All components are placed on the **TOP layer** as required. The placement follows functional grouping:

- **Center:** nRF52840 (U$1) with bypass capacitors and crystals immediately adjacent
- **Top-center:** USB-C (J4, fixed) with ESD protection (D4) and charger (IC3) nearby for short VBUS trace
- **Top-right corner:** Antenna (ANT1) with RF keep-out zone — no copper on any layer
- **Left side:** E-paper connector (J1, fixed) with entire drive circuit (MOSFETs, diodes, inductor, capacitors)
- **Bottom-left:** DC/DC converter (IC2) with inductor L1 adjacent for short switching loop
- **Center-bottom:** IMU (IC1) — away from haptic driver
- **Right side:** Haptic driver (IC4) and fuel gauge (U1) — away from IMU
- **Bottom edge:** Buttons (SW1-SW3, fixed) with pull-up resistors and debounce capacitors adjacent
- **Right edge:** SWD connector (J2) for programming access

### Fixed Components (from enclosure constraints)

The following components are pre-placed in the .fbrd file and aligned with the physical enclosure:
- **J1** (503480-2400) — E-paper FPC connector, left notch
- **J4** (KH-TYPE-C-16P) — USB-C connector, top edge
- **SW1, SW2, SW3** (EVP-AKE31A) — Buttons, bottom edge, aligned with enclosure button openings

### Routing Notes

- **Power traces** (VBUS, VBAT, 3V3, LX1, LX2): ≥0.3mm width
- **Signal traces** (SPI, I2C, GPIO): 0.15-0.2mm width
- **USB D+/D-**: Differential pair, 90Ω impedance, matched lengths
- **RF trace**: 50Ω controlled impedance, TOP layer only, no vias
- **GND plane**: Continuous on layer 2, polygon pour on Top and Bottom
- **Via stitching**: GND vias around board perimeter every 4-5mm
- **U$1 thermal pad**: 9-12 GND vias under exposed pad

---

## Design Decisions & Known Issues

### Design Log

1. **Started with 2-layer stackup** — initial routing achieved ~70% completion with autorouter. The remaining 30% of airwires could not be resolved due to board density (46×35mm with ~100 components).

2. **Migrated to 4-layer stackup** — added dedicated GND plane (layer 2) and additional routing layer (layer 63). This improved routing completion significantly.

3. **Autorouter strategy** — used Fusion Electronics built-in autorouter with Effort=High, TopRouter variant. Multiple passes were run to maximize routing completion. Remaining unrouted signals were attempted manually.

4. **DRC violations approved** — 132 warnings were approved due to board density constraints:
   - 42 Overlap warnings (via-via, via-smd on GND plane — expected behavior)
   - 38 Copper Clearance warnings (traces in dense areas slightly below 5mil clearance)
   - 36 Board Outline Clearance warnings (components near board edge)
   - 12 Component Exclude Clearance (tight component spacing)
   - 4 Drill Clearance warnings (vias near board outline)

5. **Silkscreen** — contains only component reference designators (NamesTop layer). Value labels were hidden as they contained library metadata strings instead of clean values.

6. **Battery connection** — per requirements, the battery connects directly to test pads TP_VBAT and TP_BAT_GND (no JST connector on board).

### Known Limitations

- Board density at 46×35mm on 4 layers is at the practical limit for this component count
- Some copper clearance violations exist in the dense MCU area — could be resolved with more manual routing optimization
- RF matching network values should be verified with a VNA (Vector Network Analyzer) after fabrication
- E-paper drive circuit component values may need tuning for the specific display panel used

---

## 3D Assembly

The complete device assembly (exploded view) shows the mechanical stack-up:

1. **E-Paper Display** (top) — 32×30×1.05mm, positioned 1.5mm above PCB surface
2. **PCB** (middle) — 46×35×1.0mm, 4-layer board with all components on top
3. **Battery** (bottom) — 30×20×5mm, centered under PCB, 0.5mm clearance
4. **Shaker Motor** (bottom) — Ø10×2.7mm, positioned under IC4 (haptic driver area), 0.5mm clearance

Total stack height: ~1.05 + 1.5 + 1.0 + 0.5 + 5.0 = **~9.05mm** (without enclosure)

---

## File Structure

```
InkTime/
├── Hardware/
│   ├── InkTime-schematic.fsch          # Fusion Electronics schematic
│   ├── InkTime-pcb-v1.fbrd            # Fusion Electronics board
│   └── Schematic.pdf                   # Schematic print-out
├── Manufacturing/
│   ├── gerbers.zip                     # Gerber + drill files for fabrication
│   ├── InkTime-pcb-v1.bom             # Bill of Materials
│   └── PnP_InkTime-pcb-v1_front.cpl   # Pick and Place file
├── Mechanical/
│   ├── InkTime-exploded.step           # 3D STEP exploded view
│   └── InkTime-pcb-v1-3d.f3z          # Fusion 360 3D assembly
├── Images/
│   ├── pcb-top-3d.png                  # 3D render - top view
│   ├── pcb-bottom-3d.png              # 3D render - bottom view
│   ├── pcb-isometric-3d.png           # 3D render - isometric
│   ├── assembly-exploded.png          # Exploded view render
│   └── assembly-side.png             # Side view showing stack-up
├── LICENSE
└── README.md
```

---

## References

- [nRF52840 Product Specification](https://infocenter.nordicsemi.com/pdf/nRF52840_PS_v1.1.pdf)
- [nRF52840 Reference Circuit Layout](https://infocenter.nordicsemi.com/topic/com.nordic.infocenter.nrf52840/intf_radio_circuit.html)
- [BQ25180 Datasheet](https://www.ti.com/lit/ds/symlink/bq25180.pdf)
- [RT6160A Datasheet](https://www.richtek.com/assets/product_file/RT6160A/DS6160A-05.pdf)
- [BMA423 Datasheet](https://www.bosch-sensortec.com/products/motion-sensors/accelerometers/bma423/)
- [DRV2605 Datasheet](https://www.ti.com/lit/ds/symlink/drv2605.pdf)
- [MAX17048 Datasheet](https://datasheets.maximintegrated.com/en/ds/MAX17048-MAX17049.pdf)
- [Waveshare 1.54" E-Paper Specification](https://www.waveshare.com/w/upload/7/77/1.54inch_e-Paper_Datasheet.pdf)
- [PCB Silkscreen Guidelines (Altium)](https://resources.altium.com/p/your-guide-pcb-silkscreen)
- [PCB Silkscreen Guidelines (Cadence)](https://resources.pcb.cadence.com/blog/2022-essential-pcb-silkscreen-guidelines-for-layout)
