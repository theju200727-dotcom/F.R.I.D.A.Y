# Pin Mapping

All three nodes use ESP32 DevKit V1 (30-pin) boards with Waveshare Core1262 (SX1262) LoRa modules, driven via [RadioLib](https://github.com/jgromes/RadioLib).

> **Why RadioLib?** The older `LoRa` library (Sandeep Mistry) does not support the BUSY, RXEN, TXEN, DIO1, and TCXO pins that the SX1262 requires. RadioLib is a hard requirement for this hardware.

## Base Station A / Base Station C (single radio each)

| Signal | GPIO |
|---|---|
| NSS | 5 |
| RESET | 4 |
| BUSY | 34 |
| DIO1 | 35 |
| RXEN | 25 |
| TXEN | 26 |
| SCK / MISO / MOSI | 18 / 19 / 23 |

## Drone (dual radio, shared SPI bus)

**Uplink radio** (faces Base A, sync word `0xFA`)

| Signal | GPIO |
|---|---|
| NSS | 5 |
| RESET | 4 |
| BUSY | 34 |
| DIO1 | 35 |
| RXEN | 25 |
| TXEN | 26 |

**Downlink radio** (faces Base C, sync word `0xFF`)

| Signal | GPIO |
|---|---|
| NSS | 27 |
| RESET | 32 |
| BUSY | 14 |
| DIO1 | 21 |
| RXEN | 33 |
| TXEN | 13 |

**Shared bus:** SCK = GPIO 18, MISO = GPIO 19, MOSI = GPIO 23

## Auxiliary

| Signal | GPIO | Notes |
|---|---|---|
| SW-420 vibration sensor | 32 | Base A only — hardware disaster trigger |

## Important Notes

- **GPIO 34 and 35 are input-only** with no internal pull resistors. Floating DIO1 lines cause phantom edges — use external pull-downs.
- Verify continuity and connector seating on all antenna/RF paths before assuming a firmware bug; several past issues traced back to a loose downlink antenna or swapped RXEN/TXEN wiring rather than code.
