# Neon Pebble — Heltec Wireless Tracker V2 port (first-stage)

This branch is based on `firmware-new-io`. It adds a separate
`heltec-wireless-tracker-v2` PlatformIO target, preserving the working
`heltec-v4` target and the existing Neon Pebble Friend Finder code.

## Build

```bash
pio run -e heltec-wireless-tracker-v2
```

The intended output is `.pio/build/heltec-wireless-tracker-v2/firmware.bin`.
This is an **unverified port** until a PlatformIO build and board tests pass.

**IMPORTANT:** Select `heltec-wireless-tracker-v2`, **not** `heltec-v4`.
Back up any Meshtastic configuration and encryption keys before flashing.
Connect a suitable LoRa antenna before transmitting. Don't use high radio
power until RF operation has been checked.

## First-stage hardware mapping

| Peripheral | GPIO or details |
| --- | --- |
| Built-in TFT | ST7735S, CLK 41, MOSI 42, CS 38, DC 40, RST 39, BL 21 |
| Radio | SX1262, CLK 9, MOSI 10, MISO 11, CS 8, RST 12, BUSY 13, IRQ 14 |
| RF front end | KCT8103L, power 7, CSD 4, CTX 5, CPS via radio DIO2 |
| Built-in GPS | UC6580, RX 33, TX 34, RESET 35, PPS 36 |
| Primary I2C / compass | SDA GPIO6, SCL GPIO17 |
| Secondary I2C (optional LIS3DH) | SDA GPIO15, SCL GPIO16 |
| User button | GPIO0 |
| Battery measurement | GPIO1, ADC enable GPIO2 |

**Do not connect the compass to GPIO5**: on the Tracker V2, GPIO5
switches the RF front-end receive/transmit mode.

On this initial port, the Neon Pebble menu is drawn to the **built-in
160 × 80 ST7735S screen** using the existing monochrome-style menu
renderer. The external circular GC9A01 display and custom colour UI
are a later feature.

## Verification checklist

1. Build `pio run -e heltec-wireless-tracker-v2` successfully.
2. Connect via serial (115200 bps); confirm correct board identity,
   no boot loops, and normal region onboarding.
3. Set region ANZ for use in Australia. Confirm the Neon Pebble menu
   appears after region has already been provisioned and the board restarts.
4. Confirm button navigation and built-in TFT text readability.
5. Confirm GPS fixes outdoors and battery percentage.
6. Confirm LoRa TX **and** RX with another known-working Meshtastic node
   on the same legal radio preset, frequency slot and channel. Compare
   RSSI/SNR at equal distances before trying high TX power.
7. Attach the QMC compass to primary I2C GPIO6/GPIO17. Check the
   serial I2C scan, heading response and calibration. The existing
   driver is written for QMC5883L-compatible registers; any QMC5883P
   variant needs hardware verification.
8. Confirm Friend Finder pairing, saved places and repeated GPS
   position exchange in both directions.
9. Test standby/sleep and measure current consumption.

The RF FEM and PA gain curve were adapted from newer upstream
Meshtastic Tracker V2 support; **the gain and legal EIRP still require
real-hardware verification**.

## Scope and known uncertainties

- Build and flash tests have **not** been performed in this environment.
- Only board support, radio front-end logic and existing UI integration
  have been changed; Friend Finder packet formats remain unchanged.
- The V4-only magnetometer sensorless test patch will **not** apply to
  the Tracker V2 build, so compass detection should run.
- The TFT is not round; some menu labels may require shortening.
- Features depend on the specific board revision and external compass
  hardware.
