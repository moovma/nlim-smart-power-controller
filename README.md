# ⚡ NILM Smart Switch
### Intelligent Energy Monitoring, Load Identification & Remote Control
<img src="Capture d'écran 2026-08-09 183044.png" width="850">
**Learn. Identify. Control. Optimize.**

The **NILM Smart Switch** is an intelligent 230 VAC power monitoring and control platform that goes far beyond a conventional energy meter. It doesn't just tell you *how much* electricity a house is consuming — it learns to tell you **which appliance is consuming it**, lets you **control every load remotely from your phone**, and even allows the **utility provider (STEG)** to remotely manage power delivery to a household.

The project combines **Non-Intrusive Load Monitoring (NILM)**, precision voltage/current acquisition, Fourier/harmonic analysis, active relay-based load isolation, and human-in-the-loop machine learning to progressively build a complete electrical "map" of a home.

---

## 📖 Table of Contents

- [Overview](#-overview)
- [The Core Concept](#-the-core-concept)
- [What is NILM?](#-what-is-nilm)
- [System Layout](#-system-layout)
- [How the System Learns Your Loads](#-how-the-system-learns-your-loads)
- [Electrical Signature Analysis](#-electrical-signature-analysis)
- [Schematic Design](#-schematic-design)
- [Remote Load Control & Per-Load Consumption](#-remote-load-control--per-load-consumption)
- [Utility-Level Control (STEG Integration)](#-utility-level-control-steg-integration)
- [3D Model](#-3d-model)
- [Advantages of the System](#-advantages-of-the-system)
- [Hardware Architecture](#-hardware-architecture)
- [Software Architecture](#-software-architecture)
- [Repository Structure](#-repository-structure)
- [Safety](#-safety)
- [Roadmap](#-roadmap)
- [Contributing](#-contributing)
- [Contact](#-contact)

---

## 🚀 Overview

Traditional energy meters work like this:

```
Current + Voltage
        ↓
  Calculate Power
        ↓
 Display Total Consumption
```

The NILM Smart Switch goes several steps further:

```
Voltage + Current Waveforms
             ↓
      Signal Acquisition
             ↓
   DSP & Feature Extraction
             ↓
 Harmonics + Transients + Power
             ↓
    Load Signature Analysis
             ↓
     Device Identification
             ↓
     User Confirmation
             ↓
   Improved Load Database
             ↓
   Remote Monitoring & Control
```

Instead of only answering *"how much power is being used?"*, the system answers a more useful question:

> **"What device is consuming this energy, and can I control it from anywhere?"**

---

## 💡 The Core Concept

An electrical installation already carries information about the devices connected to it — every appliance leaves a distinct "fingerprint" in the current and voltage waveforms it draws. The NILM Smart Switch is built to extract that information using:

- High-resolution voltage/current acquisition
- Real-time digital signal processing (FFT, harmonic analysis)
- Active load isolation through 12 independently switched channels
- A human-in-the-loop learning system where the homeowner confirms what each load actually is
- A cloud/mobile connection that turns raw electrical signatures into an interactive, remotely controllable smart-home layer

The result is a single central device that can:

1. **Identify** which appliance just turned on or off
2. **Measure** the real-time and historical consumption of each individual load
3. **Control** any load remotely, from anywhere
4. **Allow the utility company** to manage power delivery to the property when required

---

## ⚡ What is NILM?

**NILM** stands for **Non-Intrusive Load Monitoring** — the principle of monitoring an entire electrical installation from a single, central measurement point instead of installing a separate meter inside every appliance.

```
                HOUSE
                  │
                  ▼
          Total Current Signal
                  │
                  ▼
            NILM SYSTEM
                  │
                  ▼
      "Which loads are active?"
```

Mathematically, the aggregate current measured at the panel is the sum of every individual load's current:

$$ I_{total}(t) = I_1(t) + I_2(t) + I_3(t) + ... + I_n(t) $$

The core challenge NILM Smart Switch explores is estimating each individual component $I_1(t), I_2(t), ..., I_n(t)$ from the single aggregate measurement $I_{total}(t)$ — and then combining that estimate with **active switching** to confirm it.

---

## 🖼 System Layout

<!-- IMAGE 1 — SYSTEM / BOARD LAYOUT (view 1) -->
<p align="center">
  <img src="Capture d'écran 2026-08-09 183044.png" width="850">
</p>

*Figure 1 — NILM Smart Switch board layout, functional block placement.*

<!-- IMAGE 2 — SYSTEM / BOARD LAYOUT (view 2) -->
<p align="center">
  <img src="Capture d'écran 2026-08-10 034332.png" width="850">
</p>

*Figure 2 — Layout detail showing measurement, processing, and relay zones.*

The layout separates the board into four functional zones:

```
┌──────────────────────────────────────────────┐
│            230 VAC / RELAY SECTION            │
├──────────────────────────────────────────────┤
│        VOLTAGE / CURRENT MEASUREMENT          │
├──────────────────────────────────────────────┤
│   ADS131M04   │   STM32H7A3   │  QSPI FLASH   │
├──────────────────────────────────────────────┤
│                  ESP32-S3                     │
└──────────────────────────────────────────────┘
```

High-voltage and low-voltage areas are physically separated to respect creepage/clearance requirements while keeping signal paths short between the ADC and the DSP engine.

---

## 🧠 How the System Learns Your Loads

This is the heart of the project. The NILM Smart Switch does not assume it already knows what's plugged into your house — **it learns it, together with you.**

### Step 1 — Detecting a New Load

The STM32 continuously monitors the aggregate current and power. When a device turns on or off, it produces a measurable step change:

```
Power
  │
  │        ┌────────────
  │        │
──┴────────┘
           ↑
     Switching Event Detected
```

This event triggers a capture window around the transition, and the DSP engine extracts a full feature set: RMS current, active/reactive power, power factor, harmonic content, THD, inrush current, and rise/settling time.

### Step 2 — Comparing Against Known Signatures

The extracted feature vector is compared against the on-device **signature database**:

```
Measured Signature
        │
        ▼
┌─────────────────────┐
│ Refrigerator        │ → Score: 82%
├─────────────────────┤
│ Microwave           │ → Score: 23%
├─────────────────────┤
│ Washing Machine      │ → Score: 91%
├─────────────────────┤
│ Unknown              │ → Score: —
└─────────────────────┘
```

If the confidence score is high enough, the system identifies the load automatically. If not, it falls back to asking the homeowner.

### Step 3 — Asking the User

When the system isn't confident, it sends a notification to the mobile app asking the user to classify what just turned on:

```
┌───────────────────────────────┐
│       New Load Detected       │
│                                │
│  Estimated Power: 850 W       │
│  Channel: Kitchen Outlet 2    │
│                                │
│  What kind of device is this? │
│                                │
│  [ Fridge ] [ Washer ] [ AC ]  │
│  [ TV ] [ Microwave ] [ Other ]│
└───────────────────────────────┘
```

The user simply taps the correct category (or types a custom label such as *"Kids' Room Heater"*). This response is used to permanently label that electrical signature.

### Step 4 — Confirming with Active Isolation (Optional)

For loads the system still isn't sure about, it can use its **12 independently switched relay channels** to run a small controlled experiment:

```
All Loads Steady
      │
      ▼
Toggle Suspected Channel
      │
      ▼
Measure the Resulting Change
      │
      ▼
Match Change Against Candidate Signature
      │
      ▼
Confirm or Reject
```

This turns passive observation into an active experiment — instead of only asking *"what does the aggregate signal look like?"*, the system asks *"what changes when I switch this specific channel?"*, which makes identification far more reliable than passive NILM alone.

### Step 5 — Building the House Profile

Every confirmed label is stored in the signature database (on external QSPI flash and synced to the cloud). Over time, the system builds a complete profile of the house:

```
Signature Database
────────────────────────────
Channel 1  → Refrigerator     (confirmed)
Channel 2  → Washing Machine  (confirmed)
Channel 3  → Air Conditioner  (confirmed)
Channel 4  → Kitchen Outlet   (learning)
...
```

The more the household uses its appliances, the smarter and more accurate the system becomes — no manual configuration file, no per-device sensor installation required.

---

## 🔬 Electrical Signature Analysis

Two appliances can consume the exact same 100 W while behaving completely differently on the line. The system extracts a multi-dimensional signature rather than relying on power alone:

| Feature | Description |
|---|---|
| RMS Current | $I_{RMS} = \sqrt{\frac{1}{N}\sum V_n^2}$ |
| Active Power | $P = \frac{1}{N}\sum V_n I_n$ |
| Apparent Power | $S = V_{RMS} I_{RMS}$ |
| Power Factor | $PF = P / S$ |
| THD | $\sqrt{I_2^2+I_3^2+...+I_n^2}\ /\ I_1$ |
| Harmonic Spectrum | H1, H3, H5, H7, H9 amplitudes via FFT |
| Inrush Current | Startup current peak |
| Rise/Settling Time | Transient shape at switch-on |

```
Amplitude
    │
    │       █
    │       █
    │   █   █
    │   █   █       █
────┼──────────────────────── Frequency
    50  150 250     350  Hz
     Fundamental   Harmonics
```

Because real loads change behavior with voltage, temperature, and operating mode, the system deliberately avoids relying on a single fixed waveform template — it uses this full feature vector plus similarity scoring (Euclidean distance, cosine similarity, or a lightweight classifier) instead.

---

## 🧩 Schematic Design

<!-- IMAGE 3 — SCHEMATIC 1 -->
<p align="center">
  <img src="Capture d'écran 2026-08-24 223719.png" width="900">
</p>

*Figure 3 — Voltage and current measurement front-end schematic.*

<!-- IMAGE 4 — SCHEMATIC 2 -->
<p align="center">
  <img src="Capture d'écran 2026-08-24 223752.png" width="900">
</p>

*Figure 4 — Esp32s3 wireless connectivity .*

<!-- IMAGE 5 — SCHEMATIC 3 -->
<p align="center">
  <img src="Capture d'écran 2026-08-24 223832.png" width="900">
</p>

*Figure 5 — Relay bank section.*

---

## 📱 Remote Load Control & Per-Load Consumption

Once a channel has been identified and labeled, the mobile app turns it into a fully controllable smart outlet:

```
Real-Time Monitoring
─────────────────────────
Voltage:       229.7 V
Total Power:   910 W
Power Factor:  0.94

Load Status
─────────────────────────
✓ Refrigerator      ON    → 118 W
✓ Air Conditioner   ON    → 620 W
✗ Washing Machine   OFF   → 0 W
✓ Kitchen Outlet    ON    → 172 W
```

From the app, the homeowner can:

- **Turn any individual load ON/OFF remotely**, from anywhere with an internet connection, through the ESP32-S3's Wi-Fi/cloud link
- **See the live power draw of each identified load individually**, not just the house total
- **View historical consumption per appliance** (daily/weekly/monthly), enabling targeted energy-saving decisions
- **Set schedules or automation rules** per channel (e.g. auto-shutoff of a heater after a set time)
- **Get notified** the moment an unusual or unexpected load switches on

```
STM32H7A3 (Relay Control + Metering)
        │
        │ UART
        ▼
   ESP32-S3 (Wi-Fi / BLE)
        │
        ▼
     Cloud / App
        │
        ▼
   User's Smartphone
   (anywhere in the world)
```

---

## 🏢 Utility-Level Control (STEG Integration)

Beyond homeowner-level control, the platform is designed to support an additional layer of access reserved for the **electricity utility provider (STEG)**.

Through the same cloud/connectivity path used by the mobile app, an authorized utility system can remotely manage the main supply relay of a given household — the same way modern smart meters allow utilities to perform **remote connect/disconnect** operations. This is useful for legitimate grid-management scenarios such as:

- Scheduled maintenance or planned outages in a specific area
- Emergency load shedding during grid overload conditions
- Remote service disconnection/reconnection for account management, without requiring a technician visit
- Selectively identifying and isolating a single household's main supply from the utility's control center

```
        STEG Control Center
                │
                ▼
          Cloud Platform
                │
                ▼
   NILM Smart Switch (per household)
                │
                ▼
        Main Supply Relay
                │
        ┌───────┴───────┐
        │               │
     Connect        Disconnect
```

This capability is architecturally separate from the homeowner's per-appliance control: the utility layer only ever operates at the **main incoming supply level**, while individual appliance channels remain under the homeowner's control. Any such remote-disconnect feature must be implemented with strict authentication, encryption, and audit logging, and must comply with local utility regulations before being connected to a real grid-facing system.

---

## 🧊 3D Model
<!-- IMAGE 6 — 3D PCB MODEL -->
<p align="center">
  <img src="bgclear_transparent_original (5).png" width="950">
</p>

*Figure 6 — 3D rendered model of the NILM Smart Switch PCB.*

---

## 🌟 Advantages of the System

- **No per-appliance sensors required** — a single central unit monitors the entire installation
- **Learns automatically over time** through human-in-the-loop feedback, with no manual wiring diagrams needed
- **Per-load energy visibility** — know exactly how much each individual appliance costs to run
- **True remote control** of any of the 12 channels from anywhere, not just local automation
- **Active identification** using controlled switching, which is significantly more reliable than passive NILM alone
- **Anomaly & fault awareness** — an unexpected signature (e.g. a device drawing more current than usual) can be flagged early
- **Utility-grade remote management** compatible with smart-grid connect/disconnect operations
- **Local-first learning** — the signature database lives on-device (QSPI flash), so identification keeps working even without connectivity
- **Scalable** — the same architecture generalizes from a single apartment circuit to a full household distribution panel

---

## 🏗️ Hardware Architecture

| Component | Function |
|---|---|
| STM32H7A3VGTx | Real-time DSP, feature extraction, relay control |
| ESP32-S3 | Wi-Fi / BLE connectivity, cloud & app link |
| ADS131M04 | 24-bit precision voltage/current acquisition |
| W25Q128JV | External QSPI flash — signature database, logs |
| Current Transformer | Isolated aggregate current sensing |
| Voltage Sensor | Isolated mains voltage sensing |
| Zero-Cross Detector | Mains timing/phase reference |
| 12× Relays | Independent per-load switching |
| Isolated AC/DC Supply | Digital + analog power domains |
| 4-Layer PCB | High/low-voltage separated board |

```
230 VAC
   │
   ├── Voltage Sensor ──┐
   │                    ▼
   └── Current Sensor → ADS131M04 → STM32H7A3 → ESP32-S3 → Cloud/App
                                        │
                                        ▼
                                 12× Relay Channels
```

---

## 💻 Software Architecture

```
┌───────────────────────────────┐
│        ADC Acquisition        │
│   DRDY → SPI → DMA → Buffer   │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│       Signal Processing       │
│ Filtering / Calibration       │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        Feature Engine         │
│ RMS / Power / THD / FFT       │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        NILM Engine            │
│ Detection / Matching / Score  │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│        Control Engine         │
│ Relays / Safety / Learning    │
└───────────────┬───────────────┘
                ▼
┌───────────────────────────────┐
│   Connectivity (ESP32-S3)     │
│ App / Cloud / Utility Layer   │
└───────────────────────────────┘
```
## ⚠️ Safety

**WARNING: This project interfaces directly with 230 VAC mains voltage. It contains potentially lethal voltages.**

The design must include, and be independently reviewed for:

- Proper isolation, fuses, and surge protection
- Adequate creepage and clearance distances
- Correctly rated relay contacts and current-carrying traces
- Enclosure, earthing, and thermal protection
- Compliance with applicable local electrical safety standards

The control MCU and firmware must **never** be treated as the only safety mechanism. Any remote-disconnect or utility-control feature must additionally be secured with strong authentication and must not bypass local safety interlocks.

The PCB must not be connected to a live residential installation without proper testing and professional review.

---

## 📋 Roadmap

- [x] System architecture & concept
- [x] Voltage/current measurement design
- [x] 12-channel relay architecture
- [x] 4-layer PCB design
- [ ] PCB manufacturing & hardware validation
- [ ] ADC calibration
- [ ] FFT & harmonic feature extraction (CMSIS-DSP)
- [ ] Event detection engine
- [ ] Signature database & similarity scoring
- [ ] Active-isolation learning mode
- [ ] Mobile application (identification + remote control)
- [ ] Cloud layer & utility-level connect/disconnect API
- [ ] Real-world appliance validation
- [ ] EMC & safety compliance testing

---

## 🤝 Contributing

Contributions and ideas are welcome, especially in:

- NILM algorithms & feature engineering
- DSP / FFT optimization on STM32H7
- Appliance signature datasets
- Machine learning classification
- Embedded firmware (STM32 / ESP32)
- PCB design & hardware validation
- Mobile app development
- Cloud/utility integration

## 📬 Contact

If you're interested in the project, hardware design, NILM, embedded systems, or want to contribute ideas, feel free to open an issue or start a discussion in this repository.

---

**⚡ NILM Smart Switch**
*Learn. Identify. Control. Optimize.*
*From measuring electricity to understanding it — and controlling it, from anywhere.*
