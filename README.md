F.R.I.D.A.Y.
Flying Relay Infrastructure for Disaster Area sYstem
A drone-mounted, dual-radio LoRa relay that bridges two ground stations over a license-free 868 MHz link — built for disaster response, defence, and off-grid communication where cellular towers are down or were never there.
Overview
Disasters routinely take down cellular towers and base stations, cutting off the exact regions that need coordination the most. F.R.I.D.A.Y. lifts a communication relay into the sky using a drone, turning it into a mobile, rapidly deployable bridge between two isolated ground stations — with no fixed towers, no internet, and no cellular dependency.
The drone carries two independent LoRa radios on a single ESP32, isolated by sync word, each facing a different ground station. Messages flow bidirectionally: either station can transmit or receive, with live RSSI-based distance estimation on every hop.
```
Base Station A  ⇄  Drone Relay (dual radio)  ⇄  Base Station C
   (LPDA antenna)      Uplink 0xFA / Downlink 0xFF      (IPEX antenna)
```
Key Features
Bidirectional relay — both ground stations can send and receive through the drone
RSSI-based distance estimation — log-distance path-loss model, no GPS required
8-point compass direction tagging on every message
Per-station local WiFi dashboards — mobile-accessible, zero internet dependency
One-tap emergency presets — SOS, Medical Emergency, All Clear
CAP-style disaster alert simulation — Earthquake / Flood / Landslide triggers
Hardware disaster trigger — SW-420 vibration sensor for physical seismic simulation
Hardware
Component	Spec
Microcontroller	ESP32 DevKit V1 (30-pin) × 3
Radio module	Waveshare Core1262 (SX1262), 868 MHz × 4
Antennas	LPDA (directional, uplink) + IPEX omnidirectional
Power regulation	AMS1117 3.3V (drone board)
Sensor	SW-420 vibration/seismic trigger
Full pin mapping in `docs/PINOUT.md`.
Repository Structure
```
FRIDAY/
├── firmware/
│   ├── base_a/         # Transmitter node — LPDA antenna, WiFi dashboard, presets
│   ├── drone/          # Dual-radio relay node — bidirectional bridge
│   └── base_c/         # Receiver node — IPEX antenna, WiFi dashboard
├── docs/
│   ├── ARCHITECTURE.md # System design, sync words, data flow
│   ├── PINOUT.md       # Verified GPIO mapping for all three nodes
│   └── images/         # Architecture diagrams, photos
├── hardware/           # Wiring notes, BOM
└── README.md
```
Getting Started
Install the ESP32 board package in Arduino IDE via Boards Manager (`esp32` by Espressif Systems)
Install RadioLib via Library Manager (author: Jan Gromes)
Flash `firmware/base_a/base_a.ino` → Base Station A ESP32
Flash `firmware/drone/drone.ino` → Drone ESP32
Flash `firmware/base_c/base_c.ino` → Base Station C ESP32
Connect to the relevant WiFi AP (`BASE_A`, `DRONE_RELAY`, or `BASE_C`, password `friday123`) and open `http://192.168.4.1`
See `docs/ARCHITECTURE.md` for the full system design and `hardware/` for wiring.
Current Status
Bidirectional relay logic implemented and largely verified
RSSI-based distance estimation, dashboards, and emergency presets working
C→A path undergoing final hardware-level verification (antenna, RXEN/TXEN, ground continuity)
Roadmap: AES-256-GCM encryption layer, counter-based deduplication, regulatory frequency compliance (see Roadmap)
Roadmap
[ ] Resolve remaining C→A bidirectional diagnostics
[ ] AES-256-GCM encryption (`friday_crypto`) for the defence-use pitch
[ ] Counter-based message deduplication (replacing content-based dedup, required before encryption)
[ ] Shift centre frequency to 866.5/867.5 MHz and cap conducted power (~17 dBm) for regulatory compliance
[ ] Defence vs. civilian dual-mode architecture
Team
Built by Team F.R.I.D.A.Y., a 5-person student engineering team.
License
MIT — see `LICENSE`.
