# Demo Concept: Raspberry Pi + Tractor Autonomous Demo

*Research date: March 2026*

---

## Executive Summary

Building a compelling autonomous tractor demo is **highly feasible with $1,500–$5,000 in electronics** on top of a used compact tractor. The open-source community has already solved the core positioning and steering stack (AgOpenGPS achieves commercial-grade ±2.5 cm precision for under $500 in GNSS hardware). A startup needs to layer AI-based obstacle avoidance, a ROS2 mission planner, and polished demo choreography on top of this proven foundation. Total hardware cost for a compelling investor demo: **$10,000–$20,000 all-in** including tractor.

---

## 1. AgOpenGPS: The Proven Open-Source Foundation

### What It Is

AgOpenGPS (AOG) is an open-source precision agriculture platform created by Canadian farmer Brian Tischler around 2016. It is the de facto global standard for DIY tractor autonomy. Two programs: **AgIO** (communications hub handling all hardware I/O) and **AgOpenGPS** (main guidance application with field map, AB lines, and steer command output).

**GitHub:** https://github.com/AgOpenGPS-Official/AgOpenGPS
- Stars: 857 | Forks: 334 | Contributors: 36 | Commits: 3,571
- Latest release: v6.8.1 (November 26, 2025)
- Language: C# | License: GPLv3
- Status: **Actively maintained**, frequent releases

Active community at `discourse.agopengps.com`. The Combine Forum "DIY AutoSteer with AgOpenGPS" thread runs 40+ pages — the best public record of real-world farmer implementations worldwide.

---

### AgOpenGPS Hardware Architecture

**Important:** AgOpenGPS does *not* run on Raspberry Pi as its core compute. The canonical stack is:

**Microcontroller: Teensy 4.1 with Ethernet**
The required microcontroller for the AIO (All-In-One) board. Reads IMU, wheel angle sensor, and GPS data at hardware speed, communicates with the Windows tablet over Ethernet/USB, sends PWM or CAN commands to the steering actuator.

**GPS/GNSS: u-blox ZED-F9P (via ArduSimple simpleRTK2B board, $89–$200)**
The de facto community standard. Provides ±2.5 cm RTK accuracy. Dual-antenna heading (two ZED-F9P boards) supported and increasingly common — eliminates need for IMU heading. RTK corrections available free via community NTRIP networks in most countries.

**IMU: BNO085 breakout ($14–$20 from Adafruit)**
Provides roll/pitch/heading fusion. Runs its own ARM Cortex M0 internally — offloads the Teensy. Used in production AgOpenGPS builds.

**Wheel Angle Sensor (WAS):** Required for closed-loop steering feedback. Can use factory sensor if present, or add-on potentiometer/hall-effect sensor at steering ram or tie rod.

**Steering Actuator:**

*Option A: Motor on steering wheel* (most common for DIY)
DC motor with foam wheel pressed against the steering wheel. Cytron MD13S motor driver is the documented standard. Simple, non-invasive, reversible. ~$80–$150 for motor + mount. Agopen.shop direct-drive kit: ~€780 for a cleaner installation.

*Option B: Proportional hydraulic valve*
Taps into tractor's existing hydraulic steering circuit. More precise, fully integrated. Danfoss OSPE/OSPF valves: $1,000–$2,000.

**CAN bus integration (for CAN-steer tractors):**
Teensy 4.1 has 3 built-in CAN controllers — needs only external transceivers (SN65HVD230, 3.3V compatible). Projects: `AOG_CAN_Teensy4.1` (MechanicTony on GitHub) for J1939/ISOBUS, `tomstix/teensy-aog` specifically for Fendt tractors.

**Display: Windows tablet** (Panasonic CF-20 ~€440, Dell 5290 ~€420, or standard laptop). AgOpenGPS is Windows-only.

---

### What AgOpenGPS Can Do

| Capability | Status |
|---|---|
| AB Line guidance (straight rows) | Full — sub-2.5 cm accuracy |
| AB Curve and contour guidance | Full |
| Auto-steer (closed-loop position control) | Full |
| U-Turn / headland automation | Full — lifts implement, turns off sections, executes turn, relowers implement |
| Section control (up to 64 sections) | Full |
| Relay control (clutch, PTO, implement lift) | Full — 6 relays |
| Field mapping / coverage tracking | Full |
| ISOBUS task controller | Beta / growing |

**Demonstrated fully automated headland sequence:** AOG turns off 4WD 2 meters before headland, turns off sections, lifts implement, performs U-turn, lowers implement 3 meters before headland, re-enables sections at headland, re-enables 4WD 2 meters in. This is a substantially automated field operation.

---

### What AgOpenGPS Cannot Do

- **No obstacle detection or avoidance** — zero sensors for this. Community recommends independent obstacle sensors that cut power to steering controller if triggered.
- **No throttle/speed control** — steering and implement only; forward speed set by operator
- **No full driverless operation** — designed as driver-assisted; unmanned requires added safety hardware (geofence kill switches, obstacle sensors, remote E-stop)
- **No road driving** — field operations only
- **Windows-only** — no Linux/Pi-native version
- **Complex setup** — even experienced engineers report 2+ months of trial-and-error to commission

---

### Cost to Implement AgOpenGPS

| Configuration | Cost (USD) |
|---|---|
| Basic lightbar (no autosteer) | ~$500–$1,000 |
| Full DIY autosteer (motor-on-wheel, self-sourced parts) | ~$1,700–$2,500 |
| Mid-range with pre-assembled autosteer kit | ~$3,500–$4,000 |
| Complete kit (autosteer + RTK rover + base station) | ~$5,000–$5,225 |
| agopen.shop Direct Drive kit (assembled) | ~€2,190+ VAT |

Commercial alternatives: Trimble, Raven, John Deere StarFire = **$15,000–$30,000+ with annual subscriptions**. AgOpenGPS delivers equivalent precision at 1/10 the cost.

---

## 2. Related Open-Source Projects

### ROS2 Tractor/Farm Projects

**ros-agriculture organization:** `lawn_tractor` repo (self-driving lawn tractor on ROS), `ntrip_ros` (ROS driver for RTK corrections), `gps_navigation_goal` (set Nav2 goals from lat/lon).

**farm-ng Amiga:** All-electric open-source micro-tractor. NVIDIA Xavier NX brain (21 TOPS), IP65, up to 600 lb payload, 8-hour battery, CANbus-enabled, ROS Noetic bridge (`farm-ng/amiga-ros-bridge`). Good platform for software development. Estimated $15,000–$25,000.

**Olin Robotics Lab Tractor Sim** (`olinrobotics/tractor_sim`): Gazebo simulation + RViz for testing tractor code. Configures tractor_delay, max_acceleration, max_steering_angle_velocity.

**FroboMind / ASuBot:** Massey Ferguson 38-15 garden tractor running ROS + SICK laser range finder + Topcon AES-25 steering. Can navigate autonomously without GPS using laser-based localization. European university research platform.

**2025 Journal of Field Robotics paper:** Autonomous agricultural transport vehicle using ROS + EKF + SLAM + LiDAR + IMU + A* global path planning + DWA obstacle avoidance. The academic state of the art.

### Sabanto (Commercial Analog)

Chicago-based startup, founded 2018, raised $21M total. Retrofit autonomy kit installable in ~4 hours on existing tractors (Kubota, Fendt, John Deere). Hardware: cameras, obstacle detection sensors, GNSS, main control unit. Software: Vehicle OS (vOS) + Vehicle Path Finding Module (vPFM). Supports mowing, roto-tilling, rolling, aerating, seeding. Geofenced operation, wireless kill switch, remote E-stop, real-time monitoring. Currently expanding to Australia.

**Why Sabanto matters:** They proved the retrofit-in-4-hours story works for commercial customers. Their Series A pitch was largely about the installation simplicity and retrofit model ("works on the tractor the farmer already owns").

### John Deere AutoTrac (Commercial Benchmark)

- StarFire 7500 Receiver + Terrain Compensation Module + six stereo camera pairs for 360° obstacle detection
- Sub-inch accuracy with RTK
- AutoTrac Vision: merges GPS correction with camera-based crop row detection in real-time
- CES 2025: 9 Series full autonomy — operator assigns field and implement, machine executes entire field operation including headlands
- **Cost: $15,000–$30,000+ added to tractor price** — this is what AgOpenGPS undercuts

---

## 3. Raspberry Pi Integration Architecture

### The Dual-Controller Architecture (Proven)

**Pi 5 does not replace the Teensy/Arduino.** The proven architecture separates concerns:

```
Raspberry Pi 5 (High-level)          Teensy 4.1 (Real-time)
├── ROS2 Humble node graph           ├── Steering PWM/CAN commands
├── LiDAR SLAM (FAST-LIO2)           ├── WAS encoder reading
├── Obstacle detection (ML)          ├── IMU fusion (BNO085)
├── Mission planner (Nav2)           ├── RTK GPS data parsing
├── AgOpenGPS bridge                 ├── Safety watchdog loop
└── Data logging                     └── E-stop hardware interrupt
         ↕ Ethernet/USB serial ↕
```

Pi 5 (8GB) handles AI vision processing, path planning, ROS2 nodes, and mission management. Teensy 4.1 handles motor PWM, sensor fusion, safety loop, and sub-millisecond response. This mirrors commercial Pi 5 robot kits (MentorPi, MicroROS-Pi5).

### CAN Bus on Raspberry Pi

**Waveshare 2-CH CAN HAT (recommended, ~$13–$25):**
- MCP2515 (CAN controller) + SN65HVD230 (CAN transceiver, **3.3V compatible**)
- SPI on standard 40-pin GPIO header
- 2 isolated CAN channels, CAN 2.0B (extended frames = required for J1939/ISOBUS)
- TVS diode protection, onboard 120Ω termination resistor (jumper-selectable)
- Confirmed compatible with Pi Zero, 2B, 3B, 4B, 5
- Python examples included using `python-can` library

**Technical gotcha:** Bare MCP2515 modules with TJA1050 transceivers are **not 3.3V safe** for SPI data lines on the Pi. The Waveshare board solves this with SN65HVD230. This burns builders who use the cheap modules.

SocketCAN interface: `sudo ip link set can0 up type can bitrate 500000`. Full J1939 layer via `can-j1939` library.

### ROS2 on Raspberry Pi 5

Fully supported — ROS2 Humble runs well on Pi 5.
- Not real-time OS by default; co-processor (Teensy) handles hard real-time
- Thermal throttling under sustained compute: heatsink/fan required
- 8GB RAM recommended for full ROS2 nav stack + camera processing
- Pi 5 handles high-level planning; delegates low-level control to Teensy

---

## 4. Hardware Bill of Materials

### Which Tractor to Use

**Tier 2: Small/Compact Farm Tractor (25–75 HP) — Recommended**

Best choice for a demo that's both real and manageable. Can be transported to incubator demo days. Demonstrates genuine applicability to agricultural/construction markets.

**Recommended: Kubota BX series or John Deere 3025/3038**
- Widely available used ($5,000–$15,000)
- Good parts support, manageable size (~1,800–3,500 lbs)
- Usable in a field or large parking lot
- Modern compact tractors have CAN bus (less feature-rich than large equipment, but accessible)

---

### Full Component BOM

#### RTK GPS

| Option | Price | Notes |
|---|---|---|
| ArduSimple simpleRTK2B Basic Starter Kit (ZED-F9P + IP67 antenna) | ~$160–$200 | De facto standard |
| Dual-antenna heading kit (2x ZED-F9P boards) | ~$350–$450 | Eliminates IMU for heading |
| Emlid Reach RS2+ (complete rover) | ~$2,699 | Turn-key; best as base station |

**Recommendation:** Two ArduSimple ZED-F9P boards in moving-baseline/heading config (~$350 total) + free NTRIP corrections. Or third ZED-F9P as local base station (~$200).

#### LiDAR

| Option | Price | Notes |
|---|---|---|
| RPLIDAR S1 | ~$200–$250 | 2D, 40m, works in direct sunlight |
| **Livox Mid-360** | ~$650–$750 | **3D 360°, IP67, 70m, outdoor-rated** — recommended |
| Ouster OS0-32 | ~$6,000+ | Over-spec for early demo |

**Recommendation:** Livox Mid-360 at ~$700. IP67, 3D, 360° coverage, outdoor sunlight resistance. Best value for a compelling 3D obstacle avoidance demo.

#### Depth/Stereo Camera

| Option | Price | Notes |
|---|---|---|
| Intel RealSense D435i | ~$180–$230 | Solid indoor/controlled; not IP-rated |
| **Stereolabs ZED 2i** | ~$499–$600 | **IP66, outdoor-rated, built-in IMU** — recommended |
| Stereolabs ZED X | ~$549–$599 | IP67, GMSL2, industrial-grade |

**Recommendation:** ZED 2i at ~$550. IP66, built-in IMU, AI depth, ROS2 support, 20m range. Best all-around for outdoor tractor demo.

#### CAN Bus Interface

| Option | Price | Notes |
|---|---|---|
| Bare MCP2515 (TJA1050) | ~$5–$10 | **Not recommended** — voltage level issue on Pi |
| **Waveshare 2-CH CAN HAT** | ~$13–$25 | MCP2515 + SN65HVD230, isolated — **recommended** |
| PICAN2 DUO HAT | ~$55–$65 | More ruggedized, 2 CAN ports |

#### IMU

| Option | Price | Notes |
|---|---|---|
| **BNO085 (Adafruit breakout)** | ~$14–$20 | **Recommended** — onboard ARM Cortex M0 does all fusion |
| ICM-42688-P (SparkFun) | ~$15–$20 | High-performance raw 6-DOF; requires your own fusion code |
| VectorNav VN-100 | ~$800 | Industrial-grade overkill for demo |

#### Steering Actuator

| Option | Price | Notes |
|---|---|---|
| DC motor + foam wheel on steering wheel | ~$80–$150 | Non-invasive, reversible, proven |
| Direct-drive kit (agopen.shop) | ~€780 + VAT | Cleaner, more reliable |
| Proportional hydraulic valve (Danfoss OSPF) | ~$500–$1,500 | Most precise; requires hydraulic knowledge |

#### Safety / E-Stop

| Option | Price | Notes |
|---|---|---|
| Hardwired mushroom-head E-stop (normally-closed) | ~$15–$30 | Cuts 12V to all actuator relays |
| Dual-channel safety relay (Banner, Pilz) | ~$80–$200 | PLd Category 3; 25ms response |
| **FORT Robotics wireless E-stop** | ~$500–$2,000 | Purpose-built for outdoor autonomous vehicles; **recommended for demo** |

---

### Cost Summary

#### Minimum Viable Demo (GPS auto-steer, controlled environment)

| Item | Cost |
|---|---|
| Raspberry Pi 5 (8GB) + SD card + power supply | ~$100 |
| Teensy 4.1 with Ethernet | ~$35 |
| ArduSimple simpleRTK2B Basic Starter Kit | ~$200 |
| BNO085 IMU breakout | ~$17 |
| Wheel Angle Sensor (potentiometer + mount) | ~$40 |
| Cytron MD13S motor driver | ~$20 |
| DC motor + foam wheel mount for steering | ~$100 |
| Waveshare 2-CH CAN HAT | ~$25 |
| Basic E-stop button + relay | ~$50 |
| Windows tablet (refurbished, for AgOpenGPS) | ~$300 |
| Power distribution (12V bus, fuses, relays) | ~$75 |
| Wire harness, connectors, enclosure | ~$125 |
| **Electronics total (excl. tractor)** | **~$1,090** |
| Used compact tractor (Kubota BX / JD 3025) | ~$8,000–$12,000 |
| **Total all-in** | **~$9,000–$13,000** |

#### Full-Featured Demo (with 3D LiDAR, obstacle avoidance, polished hardware)

| Item | Cost |
|---|---|
| Minimum viable electronics (above) | ~$1,090 |
| Livox Mid-360 LiDAR | ~$700 |
| Stereolabs ZED 2i stereo camera | ~$550 |
| Second ZED-F9P for dual-antenna heading | ~$200 |
| FORT Robotics wireless E-stop | ~$800 |
| NVIDIA Jetson Orin Nano (obstacle detection compute) | ~$250 |
| Direct-drive steering motor kit (agopen.shop) | ~$870 |
| Industrial-grade wiring harness + enclosure upgrade | ~$300 |
| **Electronics total (excl. tractor)** | **~$4,760** |
| Used compact tractor | ~$10,000–$15,000 |
| **Total all-in** | **~$15,000–$20,000** |

---

## 5. The Demo Sequence

### What Makes a Compelling Demo

Based on what has converted hardware robotics pitches to investment at YC, BOOST, and top-tier VCs:

**Core principles:**
1. Show the problem before showing the solution (10-second footage: operator fatigue, labor shortage, cost overrun)
2. Quantify the value in the first 30 seconds ("each autonomous pass saves $X in labor")
3. Real autonomy, not teleoperation — "hands off, watch it go" is the moment that lands
4. Obstacle avoidance is the magic trick — walk in front of the tractor, it stops, you move, it resumes
5. Show precision — a line on the ground, the GPS track overlay, 2cm accuracy demonstrated visually
6. Show the software — large monitor with live ROS RViz (GPS track, field boundary, LiDAR point cloud) proves the system is real

---

### The 8-Minute Investor Demo Script

**Phase 1: Setup (2 min)**
"This is all the setup required." Operator types in field boundary on tablet. Sets AB line. No mechanical adjustments. Shows the simplicity.

**Phase 2: Autonomous run (3–4 min)**
Tractor begins moving autonomously. Follows first pass within ~2cm of programmed line — show on monitor. Reaches headland: automatically stops PTO/sections, lifts implement, executes U-turn, comes back on next parallel pass. No human input.

**Phase 3: Obstacle demo (1 min)**
Someone walks in front of the tractor. System detects via LiDAR/camera, stops automatically. Person moves away. System resumes. **This is the scene investors remember.**

**Phase 4: Precision proof (30 sec)**
Show GPS track log on monitor — two parallel passes 1 meter apart, both within 2–3cm of programmed lines. This is commercial-grade precision ($30K systems do this; you just did it for $1,000).

**Phase 5: The scale story (30 sec, spoken)**
"This kit, installed in 4 hours on any tractor, lets one operator manage 3–5 machines simultaneously. At $X per kit with $Y ARR subscription, payback period is under one season. The market is 501,000 workers short and getting shorter."

---

### What Technical Judges Will Look For

- **Sensor fusion:** Is it GPS-only or fusing GPS + IMU + LiDAR? Judges know the difference.
- **ROS2 integration:** Visible Nav2 stack signals real robotics engineering.
- **Safety architecture:** Hardware-level E-stop (dual-channel relay) vs. software-only geofence.
- **CAN bus knowledge:** Can you explain ISOBUS vs J1939? Do you know which tractors expose CAN-steer vs require motor actuation?
- **Control algorithm:** Pure Pursuit? Stanley? MPC? Explaining your path tracking algorithm and tuning parameters impresses robotics judges.
- **The honest limitations slide:** Judges trust founders who clearly articulate what doesn't work yet. "No public road driving, obstacle avoidance works within 10 meters at current sensor configuration, full driverless requires safety operator" = credible engineering team.

---

### Demo Logistics

**Location options (in order of impressiveness):**
1. Real farm or construction site (most impressive; hardest to arrange for incubator demos)
2. Large parking lot with painted field boundary lines (very workable; shows it's not tied to a specific location)
3. Indoor large warehouse/convention center (manageable; good for BOOST/YC demo days)
4. Rented agricultural land near demo site (best all-around for planned demo days)

**Trailer/transport:** A compact tractor (3,000 lbs) fits on a standard 5x10 utility trailer behind a 3/4-ton pickup. Demo is genuinely portable.

**Weather contingency:** Have a tight 4-minute indoor version with the laptop showing a recorded real-world run if outdoor conditions fail. "We'd normally do this outside but here's the live data system running on yesterday's real field data."

---

## 6. Biggest Technical Risks

| Risk | Likelihood | Mitigation |
|---|---|---|
| RTK signal loss (metal buildings, tree canopy) | Medium | Use NTRIP network corrections + Emlid RS2+ base station as backup; test site for GNSS quality first |
| Steering motor slipping on wet/cold wheel | Medium | Direct-drive kit eliminates this; foam wheel slip is the main failure mode of motor-on-wheel approach |
| Windows Update breaking tablet in the field | Low-Medium | Lock Windows Update on demo tablet; use offline WSUS or just disable auto-updates |
| CAN bus access restrictions on modern tractors | Medium | Test CAN access on your specific tractor model before committing; older (pre-2020) tractors are more accessible |
| LiDAR obstruction (mud, dust accumulation) | Low | Wiper system or air-blast cleaning for the sensor head; IP67 rating handles weather |
| Vibration disconnecting wiring | Low-Medium | Use locking connectors (Deutsch DT series); strain-relief all harness runs |

---

*Sources: AgOpenGPS Official GitHub (v6.8.1), AgOpenGPS Discourse community, Farmers Weekly DIY autosteer guide, AGGPS.CA cost guides, ArduSimple product pages, Waveshare CAN HAT documentation, Adafruit BNO085, Livox Mid-360 specs, Stereolabs ZED 2i specs, farm-ng Amiga product page, Sabanto Ag press releases, John Deere AutoTrac documentation, ros-agriculture GitHub organization, Olin Robotics tractor_sim, FORT Robotics E-stop specs, YC Demo Day guide, Sabanto/Monarch Tractor/Blue White Robotics funding announcements.*
