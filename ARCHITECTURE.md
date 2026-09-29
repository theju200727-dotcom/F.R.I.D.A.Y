# Architecture

## System Overview

F.R.I.D.A.Y. is a three-node relay chain operating on the license-free 868 MHz ISM band using LoRa modulation.

```
Base Station A  --LPDA-->  [ Uplink radio  0xFA ]
                              DRONE (shared ESP32, SPI bus)
Base Station C  <--IPEX--  [ Downlink radio 0xFF ]
```

- **Base Station A** — transmits status/distress messages via a directional LPDA antenna
- **Drone relay** — carries two independent SX1262 radios on one ESP32; the uplink radio faces Base A (sync word `0xFA`), the downlink radio faces Base C (sync word `0xFF`)
- **Base Station C** — receives relayed data via an omnidirectional IPEX antenna, and can reply back through the same relay

The link is fully bidirectional: a reply from Base C is picked up by the drone's downlink radio and forwarded back through the uplink radio to Base A.

## Radio Firmware Pattern

SX1262's DIO1 pin fires on **both** `RxDone` and `TxDone`, which makes naive interrupt-driven relay logic prone to duplicate or spurious relaying. The firmware instead uses:

- **Edge-triggered polling** (`digitalRead()` LOW→HIGH transition) instead of `attachInterrupt`
- An explicit `radio.standby()` call immediately before every `transmit()`

This combination resolved both the duplicate-relay bug and intermittent `RADIOLIB_ERR_PACKET_TOO_LONG` (-5) errors seen on the C→A path.

## Known RF Constraint — Co-site Interference

The two SX1262 radios sit roughly 20 cm apart on the drone. When one radio transmits, it can flood the other radio's receiver front-end at power levels far above its sensitivity floor — a co-site interference problem that sync words alone do **not** solve, since it's an analog front-end effect rather than a packet-filtering one.

Mitigation requires:
- ≥1.2 MHz frequency separation between the uplink and downlink channels
- TDD-style scheduling with guard intervals between transmit windows

## Power

The ESP32's onboard 3.3V rail cannot reliably source two SX1262 modules transmitting simultaneously. The drone board uses a dedicated external **AMS1117 3.3V regulator** to avoid brownout-induced radio resets. Base A and Base C run on onboard regulation plus external power banks.

## Distance Estimation

RSSI is converted to an approximate distance using a log-distance path-loss model. This is a rough estimate, not a GPS-grade measurement, and should be calibrated against a known reference distance for your specific antenna/environment combination.

## Local Dashboards

Each node hosts its own WiFi access point and a mobile-responsive web dashboard — no internet or external network required.

| Node | SSID | Password | URL |
|---|---|---|---|
| Base Station A | `BASE_A` | `friday123` | `http://192.168.4.1` |
| Drone | `DRONE_RELAY` | `friday123` | `http://192.168.4.1` |
| Base Station C | `BASE_C` | `friday123` | `http://192.168.4.1` |

## Regulatory Note

The current default centre frequency sits at the edge of the 868.0 MHz band. Planned work shifts this to 866.5 or 867.5 MHz and caps conducted power around 17 dBm to stay within the 500 mW ERP limit under India's G.S.R. 853(E) 2021 licence-exempt regulations. Confirm applicable regulations for your deployment region before field use.
