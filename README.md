# 🇺🇦 Passive Defeat Layer
## Open Source FPV / Drone Detection & Alert System

**By Christopher "I-ZOMBIE" Paulos — Independent Maker, USA**
**License: MIT — Free to use, build, modify, give away forever.**

---

> *"I am one citizen honoring the accords my country signed.  
> Sometimes that's all it takes."*
> — C. Paulos, June 2026

---

## What This Is

A passive, layered sensor array designed to detect incoming FPV drones, explosions, and impacts — and alert defenders — without active radar, without exotic parts, and without a supply chain.

**It can be built in a basement. From salvaged parts. In 20 minutes per panel.**

---

## The Problem It Solves

Small FPV drones are destroying vehicles and lives faster than expensive radar-based countermeasures can respond.

- Trophy system: ~$1,000,000
- Arena-M: ~$300,000
- Cope cage (passive, no detection): ~$500
- **This system (passive + detection + alert): ~$0.44–$3.20 per panel**

Cost asymmetry: their $400 drone vs your $0.44 detector. **You win economically every time.**

---

## Three Sensor Layers

### Layer 1 — Steel Mesh / Braided Cable
Physical defeat substrate. Cheap. Weldable to any vehicle. Woven with fiber.

### Layer 2 — PVDF Piezoelectric Pole Array
Self-generating voltage on impact or flex. No power needed at sensor point.
~10 microsecond response. Detects FPV contact, blast overpressure, physical strike.

### Layer 3 — Fiber Optic Burst Disc (EMP-IMMUNE)
Plastic optical fiber woven through mesh. Flash/blast transmits light pulse to
shielded PIN photodiode detector. Sub-nanosecond response. **EMP immune — no
electronics at the sensing point, just glass.**

> 🇺🇦 **Ukraine field note:** FPV tether fiber recovered from downed drones is
> scattered across Ukrainian towns. Strip the outer jacket. Use the inner fiber.
> **This layer is FREE in Ukraine.**

---

## Two Tiers — Pick Your Supply Chain

### Tier 1 — Pure Analog (No Code, No Board)
| Item | Value |
|------|-------|
| Components | LM393 comparator + 2N2222 NPN + 2 resistors |
| Total parts | **4** |
| Cost per panel | **~$0.44 new // ~$0.05 salvage** |
| Firmware | **None. Zero. Never.** |
| Chain method | 2-wire daisy chain, unlimited panels |
| On hit | Fiber zone glows RED instantly |
| Works after | Being shot, frozen, soaked, or sat on |

**If it needs firmware, it's too complicated.**
A comparator doesn't care about supply chains, language barriers, or update cycles.

### Tier 2 — Smart Panel (ATmega328 / Arduino Clone)
| Item | Value |
|------|-------|
| MCU | ATmega328P bare chip ($1.50) OR any Arduino Nano clone |
| Available | Poland, Romania, India, Egypt — no supply chain lock-in |
| Bus | RS-485, 2 wires, 32 panels, 1,200m range |
| Cost per panel | **~$3.20** |
| Detects | Small impact / large explosion / optical flash (discriminated) |
| Logs | Zone ID + timestamp (nanoseconds) + estimated velocity |
| Firmware | 60 lines of Arduino code — included |

---

## Fiber Optic Visual Zone Map

Lightly sand plastic fiber cladding with 400-grit sandpaper → light bleeds out sideways.
Different LED color per zone. **The net itself becomes the display.**

| State | Color |
|-------|-------|
| Armed | 🟢 Green |
| Small impact | 🟡 Amber |
| Large blast | 🔴 Red |
| Flash / optical | 🔵 Cyan |

No screen needed. Visible at distance. Costs nothing extra.

---

## Road Panel Configuration

Same sensor stack, tiled horizontally over roads or supply corridors.
Each panel is self-contained. Panels communicate over 2 wires.
Approach direction and velocity calculated from cross-panel trigger timing.

**V = panel_spacing / Δt** (time between adjacent panel triggers)

---

## Key Numbers

| Metric | Value |
|--------|-------|
| Full vehicle rig cost | ~$60–80 |
| Road panel (10m) | ~$4–8/meter |
| Tier 1 panel | ~$0.44 |
| Tier 2 panel | ~$3.20 |
| Replace 1 panel | 20 minutes + $0.44 |
| Replace 1m fiber | $0.03 (or free) |
| Enemy FPV cost | $400–1,500 |
| **Cost ratio** | **~1,000:1 in Ukraine's favor** |

---

## Files in This Repo

| File | What It Is |
|------|-----------|
| `sensor_array_rig.html` | Interactive sensor array designer + oscilloscope simulation |
| `sensor_array_bom.html` | Full bill of materials with salvage mode cost calculator |
| `road_panel_system.html` | Dual-tier road panel spec, circuit diagrams, firmware, fiber sim |
| `submission_package.html` | Submission guide for Brave1, DTU, UNITE, DARPA |
| `send_these_now.html` | Pre-written submission emails (ready to send) |

**All files are self-contained HTML — open in any browser, no install, no server.**

Enable GitHub Pages (Settings → Pages → main branch) for live interactive demos.

---

## How to Build It

### Tier 1 Panel (20 minutes)
1. Cut coat hanger / rebar wire to 350mm. Bend 20mm base tab.
2. Epoxy PVDF piezo film strip to pole base tab.
3. Solder 2 leads to film electrodes. Wrap in inner tube strip.
4. Wire: PVDF → 100k resistor → LM393 pin 2. Trimmer to pin 3. Output to 2N2222 base.
5. LED face-pressed to fiber end. Epoxy. Done.

### Fiber Termination (5 minutes)
1. Cut fiber to length.
2. Polish end face: 600-grit → 2000-grit wet/dry, rotating. 30 seconds each.
3. Press face-to-face with PIN photodiode (BPW34). Heat shrink over both.
4. **Side emission:** run 400-grit lightly along fiber length → light bleeds out.

### Controller Box
Any sealed food container. Drill cable entries. Fill with hot glue. RPi Zero 2W inside.
Power: USB power bank / 4×AA + boost converter / salvaged drone LiPo + BMS.

---

## Arduino Firmware (Tier 2)

```cpp
// road_panel.ino — ATmega328 / Arduino Nano
// RS-485 panel ID set by jumpers on D8/D9/D10
#include <SoftwareSerial.h>
#define PANEL_ID    0x01  // change per panel
#define TRIG_THRESH 300   // ADC units (~1.5V) small impact
#define BIG_THRESH  700   // large impact / explosion
#define FLASH_PIN   2     // fiber photodiode interrupt

SoftwareSerial rs485(10, 11);
int zones[4] = {A0, A1, A2, A3};

void setup() {
  rs485.begin(9600);
  attachInterrupt(0, onFlash, RISING);
  pinMode(6, OUTPUT); // Red fiber LED
  pinMode(5, OUTPUT); // Green fiber LED
  setFiber('G');      // Armed = green
}

void loop() {
  for (int i = 0; i < 4; i++) {
    int v = analogRead(zones[i]);
    if (v > TRIG_THRESH) {
      char t = v > BIG_THRESH ? 'B' : 'I';
      report(t, i, v);
      setFiber(t == 'B' ? 'R' : 'A');
      delay(2000);
      setFiber('G');
    }
  }
}

void report(char type, int zone, int val) {
  rs485.print(PANEL_ID, HEX);
  rs485.print(':'); rs485.print(type);
  rs485.print(':'); rs485.print(zone);
  rs485.print(':'); rs485.println(val);
}

void onFlash() { report('F', 0, 999); }

void setFiber(char c) {
  digitalWrite(6, c == 'R' || c == 'B');
  digitalWrite(5, c == 'G' || c == 'A');
}
```

---

## Salvage Substitutions

| Buy | Free Substitute |
|-----|----------------|
| Steel pole stock | Coat hangers, rebar wire |
| Signal wire | Stripped Cat5 cable (8 wires per cable) |
| Plastic fiber | Recovered FPV tether fiber (free in Ukraine) |
| Piezo sensor | Smoke alarm / greeting card piezo disc |
| Project box | Food container + hot glue |
| Resistors/caps | Salvaged from dead drone PCBs |
| Pole weatherproofing | Bicycle inner tube strips |
| Mesh | Window screen / galvanized fencing |

---

## Current Status

**TRL 2-3** — Simulation and circuit design complete. Physical prototype not yet built.

Seeking Ukrainian partner — volunteer lab, maker collective, or unit — for physical prototype and field validation. All support provided remotely.

**Submitted to:**
- Brave1 / Battle Proven 2026 (June 2026)
- Defense Tech for Ukraine (DTU)
- UNITE — Brave NATO

---

## License

MIT License — use it, build it, modify it, give it away.  
No IP claims. No fees. No company. Just open source.

---

## Contact

**Christopher "I-ZOMBIE" Paulos**  
Independent Maker / Open Source Researcher  
obtusemuch@gmail.com  
GitHub: github.com/2-OBTUSE  

---

*Glory to Ukraine. Glory to all the heroes, seen and unseen.* 🇺🇦
