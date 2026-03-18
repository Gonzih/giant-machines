# Technical Architecture: Autonomous Construction Equipment

*Research date: March 2026*

---

## Executive Summary

Building autonomous construction equipment requires solving four distinct technical problems: (1) simulating soil-machine interaction to train AI policies, (2) perceiving a dynamic, dusty, GPS-occluded job site, (3) commanding hydraulic actuators with precise closed-loop control, and (4) transferring learned policies from simulation to real machines without crashing. Each problem has a known solution — the field is mature enough that a technically strong team can assemble a working stack. The key insight: **no existing AI foundation model is applicable out-of-the-box**. The domain is wide open for a company willing to build the first construction-specific training data asset and RL/VLA policy pipeline.

---

## 1. Simulation Environments

The right simulator depends on whether you're training control policies (need accurate physics), developing perception (need accurate sensors), or doing both. For autonomous construction equipment, **physics accuracy for soil-machine interaction is the non-negotiable requirement**.

### The Options and Their Real Capabilities

#### AGX Dynamics (Algoryx) — The Specialist Leader for Construction

**What it is:** Commercial physics engine from Algoryx Simulation AB (Umeå, Sweden, spun off from Umeå University 2007). The only off-the-shelf, real-time engine with genuine deformable soil simulation + hydraulic dynamics.

**Why it matters for construction:**
- **AGX Terrain:** Real-time deformable soil simulation. The excavator bucket is defined as a "shovel element" — when the cutting edge contacts terrain, AGX evaluates contact angle and soil shear properties, deforms the mesh, and converts excavated material into dynamic particles that interact with terrain, other particles, and the machine. Supports dirt, gravel, sand, defined by Young's Modulus, friction angle, cohesion, and compaction. Mass is rigorously conserved.
- **AGX Hydraulics:** 1D hydraulics (accumulators, valves, motors, pumps) tightly coupled to 3D multibody. Models complete excavator/crane/wheel loader hydraulic circuits.
- **AGX Driveline:** Clutches, motors, gears, mixed mechanical/hydraulic systems.
- **Performance:** 60 Hz real-time at "high fidelity." 75–90% accuracy of full DEM on digging forces, validated with full-scale construction equipment field tests.

**Who uses it:**
- ETH Zurich / Gravis Robotics: primary simulation platform for 40-ton material handler research
- TERA simulator (arXiv:2501.01430): Unity3D + AGX for autonomous excavator training
- The RL rock capture paper (arXiv:2510.04168): PPO agent on CAT 365 model in AGX — 0.80 success rate on digging tasks
- Used in professional operator training simulators industry-wide

**Integration:** Unity (via AGX Dynamics for Unity), Unreal Engine (via AGX Dynamics for Unreal on FAB). ROS bridge available.

**Cost:** Commercial license — contact Algoryx. Not free. Worth it.

---

#### NVIDIA Isaac Sim — Best for Perception, Inadequate for Soil

**What it is:** Built on NVIDIA Omniverse/USD, uses PhysX for rigid-body dynamics. Isaac Lab provides GPU-parallelized RL training.

**Construction-relevant capabilities:**
- GPU parallelization: 90,000+ FPS for legged locomotion training via Isaac Lab
- High-fidelity sensor simulation: LiDAR, cameras, IMUs via OmniSensor USD schema
- Domain randomization of visual appearances, lighting, textures
- NVIDIA Cosmos world foundation model integration (Isaac Sim 4.5+) for scene generation

**Critical limitation:** **No native deformable terrain support.** Granular soil simulation (sinkage, rutting, compaction) is explicitly not on the PhysX roadmap. Developers report it is essentially impossible within the current framework. Workaround is friction/material parameter tuning on rigid meshes — an approximation inadequate for training accurate digging policies.

**The Newton exception:** NVIDIA + Google DeepMind + Disney Research are co-developing the **Newton physics engine**, implementing Implicit MPM (Material Point Method) and MuJoCo Warp solvers. Lightwheel has contributed open-source locomotion assets over deformable sand, soil, and snow terrain. **Newton is in active development as of 2025 and not production-ready.**

**Best use for construction:** Synthetic data generation for perception (LiDAR point clouds, camera images), sensor simulation, visual domain randomization. Not for training digging or grading policies.

---

#### Project Chrono (Open Source) — Best for Large-Scale Policy Training

**What it is:** Open-source multi-physics engine with two relevant terrain models.

**SCM (Soil Contact Model):** Semi-empirical deformable terrain based on the Bekker-Wong model generalized for 3D contact shapes. Virtual Cartesian grid; deformed nodes in a hash map — virtually unlimited terrain dimensions with cost scaling only with active contact area. Bulldozing side-rut algorithm included. Near-real-time performance. Validated against rover and off-road vehicle data. Best for wheeled/tracked vehicles on large terrains.

**CRM (Chrono::CRM, GPU-accelerated SPH):** High-fidelity SPH with Mohr-Coulomb plasticity and rate-dependent friction. Validated (R² = 0.9714 for sphere cratering, 3.5% mean relative error for cone penetration). Can simulate 6 km terrain patches on a single RTX 4080Ti, 29 km on H100, up to 100 million SPH particles at near-interactive rates. **An RL policy for autonomous grading was trained and demonstrated within Chrono::CRM (arXiv:2507.05643).** This is an end-to-end open-source pipeline from simulation training to construction-relevant task completion.

**SynChrono:** Multi-vehicle synchronization over deformable terrain — fleet training.

**Best use:** Large-scale GPU-parallel policy training for dozing, grading, and earthmoving tasks. Open-source and H100-scalable means this is the training infrastructure backbone.

---

#### Gazebo (ROS) — Not Suitable for Heavy Equipment

**Why not:** Platforms over ~35 tons cause simulation crashes or ground penetration due to required joint softness parameters (ERP/CFM tradeoffs). No deformable terrain, no track-soil interaction model, no hydraulic dynamics. Existing construction worlds are designed for lightweight UGVs.

**Limited use:** Rapid prototyping of ROS integration, sensor interfaces, and communication pipelines where physics fidelity doesn't matter.

---

#### CARLA — Perception Only

Road-centric (OpenDRIVE standard). No construction equipment assets, no soft-soil terrain, no construction site environments. The AVL VSM co-simulation coupling adds soft-soil tire physics but with limited community uptake.

**Limited use:** Developing and testing perception algorithms when you can accept simplified terrain physics.

---

#### Unreal Engine 5 + Chaos Physics + MATLAB/Simulink

Chaos Modular Vehicles (UE 5.5) support real-time vehicle assembly but no native deformable soil.

The most developed UE-based construction autonomy pipeline is **MATLAB Robotics System Toolbox Offroad Autonomy Library + Simulink 3D Animation**: Simscape Multibody excavator models (boom, stick, bucket, swing, tracks as prismatic/revolute joints; hydraulic pistons as Cylindrical/Universal Joint blocks), 4x LiDAR sensors, RRT-based path planning with collision checking against `occupancyMap3D`, TOPPRA trajectory optimization, MIMO PI hydraulic controllers. MathWorks provides an open-source testbench (`mathworks-robotics/autonomous-excavator`).

**Best use:** Controller algorithm development and testing, not RL policy training. Good for teams with Simulink infrastructure.

---

### Recommended Simulation Stack for Construction Autonomy Startup

| Purpose | Tool | Why |
|---|---|---|
| Primary RL training (digging, excavating) | **AGX Dynamics + Unity** | Only real-time deformable soil + hydraulics; proven in ETH Zurich/Gravis stack |
| Large-scale GPU training (grading, dozing) | **Project Chrono::CRM** | Open-source, H100-scalable, RL policy training demonstrated |
| Perception data generation, sensor sim | **NVIDIA Isaac Sim** | GPU-parallel, sensor-accurate, domain randomizable |
| Controller design / hydraulic modeling | **MATLAB/Simulink Simscape** | Standard industry tool; generates deployable C/C++ code |
| Semi-empirical terrain at scale | **Chrono SCM** | Near-real-time, validated, covers wheeled/tracked vehicles on large terrains |

---

## 2. Perception Stack

### LiDAR for Construction Sites

**Primary recommendation: Ouster OS2-128 or Hesai XT32/AT128**

| Sensor | Key Specs | Notes |
|---|---|---|
| Ouster OS2-128 | 120m range, IP68/IP69K, REV7 doubles shock/vibration resistance, ~95% automotive-grade components | Strongest vibration rating; documented in Sandvik/Liebherr mining deployments |
| Hesai XT32 | 32-ch, 120m, 5mm precision (1σ), zero blind range, -40°C validated | Used on Boonray autonomous mining trucks; excellent in heavy-equipment vibration |
| Hesai AT128 | 128-ch hybrid solid-state, 1.53M pts/sec, 200m range, automotive ASIC | Best cost/performance for high-channel production builds |
| Livox Mid-360 | 360° FOV, IP67, 200K pts/sec, 70m range, non-repetitive scan | Best value for outdoor 3D demos ($650–$750) |

**HEAP excavator** (ETH Zurich) specifically chose two Velodyne VLP-16 LiDARs over cameras due to superior performance in heavy dust.

**Dust and vibration:** Most construction LiDARs maintain functional detection at operational distances despite typical dust concentrations. Silica dust from rock drilling is worst-case — where 4D radar becomes necessary. Vibration isolation mounts used in extreme deployments; Ouster REV7 designed to handle most construction vibration without isolation.

---

### Stereo Cameras

Triangulation-based depth from stereo works in direct sunlight (unlike structured-light or ToF). For outdoor construction:

- **Stereolabs ZED 2i:** IP66, polarizing filter, 20m range, 110° FOV, built-in 9-DOF IMU, ROS2 support. Best all-around for outdoor demos.
- **Stereolabs ZED X:** IP67, GMSL2, thermal control. Industrial production choice.

Camera lenses require wiper systems or air-blast cleaning on long deployments in mud.

---

### GPS/RTK Positioning

RTK achieves 1–2 cm horizontal, 2–4 cm vertical accuracy using carrier-phase GNSS with base station or corrections network.

- **Trimble (MS992/MS995/MS996):** Industry standard for machine control. xFill satellite backup for base station outages (bridges up to 15–20 min). CAN-bus compatible (J1939).
- **Leica iCON iXE3:** Used on HEAP excavator. Strong performance in obstructed environments.
- **u-blox ZED-F9P:** Dominant chip for embedded/custom systems. PointPerfect Network RTK corrections. ~$130–$200 for the module. The AgOpenGPS community standard.

For construction sites: network RTK corrections (Trimble CenterPoint RTX, NTRIP networks) provide site-wide cm-level positioning without a dedicated base station.

---

### IMU + GPS Fusion Architecture

Standard approach: **Tightly Coupled GNSS/INS with Error-State Kalman Filter (ESKF)**

- IMU (100–400 Hz) provides short-term high-rate position/velocity/attitude via dead reckoning
- GNSS RTK (1–10 Hz) corrects accumulated IMU drift
- Tightly coupled filters fuse raw GNSS pseudoranges with IMU — functions with fewer than 4 satellites (unlike loosely coupled which needs full fix)
- In GNSS-degraded environments (tree canopy, deep excavations, trenches), IMU bridges outages

**Emerging: Factor Graph Optimization (FGO)** — batch state estimation shown to reduce positioning errors by up to 41% relative to standard approaches in signal-compromised environments.

ETH Zurich's HEAP excavator state estimation: unified filter combining IMU predictions, GNSS-RTK updates, wheel rolling predictions, and leg kinematics — the first formulation for wheeled-legged excavators.

---

### Radar for Dust/Weather

**4D imaging radar** (Oculii, Arbe, Continental ARS540) for all-weather detection unaffected by particulate matter. Dust clouds from active digging can obscure LiDAR returns (especially fine silica); radar penetrates without signal loss. Sparse point clouds but sufficient for safety zone monitoring and large-obstacle detection.

**Best architecture:** Radar + LiDAR fusion — radar for coarse all-weather detection, LiDAR for precise mapping when conditions allow.

---

### Terrain Mapping and Point Cloud Processing

Standard pipeline:
1. LiDAR-based SLAM (LIO-SAM, FAST-LIO2, LOAM variants) to build 3D work area map
2. Progressive updates as terrain changes from earthmoving
3. Soil volume estimation via before/after point cloud differencing
4. 3D occupancy maps for path planning and collision avoidance

GPS-denied scenarios require pure LiDAR/visual SLAM with IMU + encoder fusion for loop closure.

---

## 3. Control Systems

### SAE J1939 — The Protocol

J1939 is the dominant protocol for heavy construction and agricultural equipment. Runs over CAN bus at 250–500 kbit/s, 29-bit extended identifiers only. Messages contain 8 bytes of data + 29-bit header with a **Parameter Group Number (PGN)** identifying message type and format. Data elements are **Suspect Parameter Numbers (SPNs)** — engine speed, throttle position, hydraulic pressure are all standardized SPNs with defined scaling.

In construction equipment: engine ECU, transmission, hydraulic controllers, sensor modules, and machine controller all communicate over J1939. Modern excavators and dozers expose a factory CAN connection — retrofit systems like Trimble Earthworks use this to bypass custom valve blocks.

---

### Hydraulic Actuation — The Electronic Control Chain

Construction equipment actuates via hydraulic cylinders and motors. Electronic control chain:

1. **Operator joystick or autonomous controller** → command signal (±10V analog, PWM, or CAN message)
2. **Proportional solenoid valve** (Bosch Rexroth D*WE series, Parker IQAN-controlled) converts electrical signal to hydraulic flow command
3. **Hydraulic cylinder/motor** moves in proportion to flow; controlled via:
   - Open-loop: command proportional to desired velocity (joystick control)
   - Closed-loop: position feedback from cylinder sensors (linear encoders, magnetostrictive transducers, or pressure-derived estimates)

**Bosch Rexroth** provides the production stack:
- E-Motion Plus valve for boom control: electronic control, fine movement, electronic pressure compensation
- IFB proportional directional control valves with Multi-Ethernet (Sercos, EtherCAT, PROFINET, EtherNet/IP)
- Connected Hydraulics platform for IoT integration

---

### The Joystick Problem: Hydraulic Pilot Pressure

Most production construction equipment uses **hydraulic pilot pressure joysticks** — the operator moves a joystick that modulates hydraulic pilot pressure, which positions the main control valve spool. **This is not electronic at the joystick level — it is a direct hydraulic signal path.**

Three approaches to retrofit autonomy (in increasing integration complexity):

| Approach | Method | Invasiveness | Notes |
|---|---|---|---|
| **Intercept pilot lines** | Proportional pilot pressure reducing valves controlled by MCU | Low | Most common retrofit; reversible; operator can override with higher pressure |
| **Install E/H valves** | Replace/supplement main control valve with Bosch Rexroth/Parker IQAN-compatible valves | Medium | Receives direct electrical command; more precise |
| **Factory EoH** | OEM-designed electronic control from factory (Komatsu iMC 3.0, some Cat machines) | N/A (built in) | Full machine controller access; only available on compatible new machines |

---

### Parker IQAN — The Machine Control Platform

**IQAN** is Parker's complete electro-hydraulic control system, the most widely used in off-highway mobile equipment for custom machine control.

**IQAN-MC31 (the key component for a startup):**
- 4 CAN buses (J1939, CANopen, other protocols)
- Fully programmable as standalone master or networked
- Closed-loop current control for hydraulic valves
- IQANdesign graphical programming environment (state machines, valve commands, sensor reading, J1939 PGN databases built in)

**IQAN Connect:** J1939 CAN bus-based electrohydraulic motion control platform. Integrates pumps, valves, motors, cylinders, displays, joysticks, cameras, and IoT gateways. Bridges machine to IoT.

**For a startup:** IQAN-MC31 + IQANdesign is the fastest path to building a custom autonomous control system on any off-highway machine. Program the state machine and supervisory controller in IQANdesign; the IQAN master handles real-time valve control loops.

---

### Safety Architecture (Minimum Viable)

Per ISO 3691-4:2023 (driverless industrial trucks) and ISO 13850 (E-stop), the minimum viable safety architecture:

| Component | Requirement | Standard |
|---|---|---|
| **E-Stop** | Stops all movements immediately; cannot auto-restart without deliberate operator action | ISO 13850 |
| **Watchdog timer** | Expires if main control loop freezes → safe state (valves center, brakes apply, hydraulics to neutral) | IEC 62061 |
| **Personnel detection** | 360° camera-based person detection + LiDAR proximity zones | ISO 3691-4 |
| **Geofencing** | GPS-defined work zone boundaries; machine halts at boundary violation | Best practice |
| **Functional safety rating** | Performance Level assignment via risk assessment | ISO 13849 |

Parker IQAN and Bosch Rexroth valves have integrated safety functions compatible with these standards. ASIL D (automotive safety integrity level D, used by SafeAI) is the highest safety certification available — a defensible moat if achieved early.

---

### Caterpillar Grade Control — Technical Reference

Cat Grade (industry benchmark for semi-autonomous grade control):
- Dual GNSS antennas → 3D positioning + machine heading
- IMU → pitch, roll, yaw compensation
- Cylinder position sensors on boom, stick, bucket → bucket cutting edge position within a few mm
- 3D design file loaded to in-cab display; system computes cutting edge vs. design surface delta continuously
- **Grade Assist (semi-autonomous):** Operator controls swing and travel; system auto-commands boom and stick hydraulics to maintain cutting edge on design surface. Single-lever digging. **Claimed 45% faster grade achievement.**
- All communications over J1939 CAN.

External integration requires Cat's proprietary API (OEM partnership required) or CAN bus interception.

---

## 4. Foundation Models for Robotics

### Current State: No Off-the-Shelf Solution for Construction Equipment

Every current robotics foundation model was trained on tabletop manipulation data. None have construction equipment data, outdoor operation data, or hydraulic actuation data. This is both the problem and the opportunity.

### The Models and Their Limitations

**RT-2 (Google DeepMind):**
- 55B parameter VLA (PaLI-X + robot trajectory co-training)
- Output: 6-DOF end-effector delta + gripper — incompatible with hydraulic velocity commands
- Not open-source
- Not applicable to construction equipment without fundamental redesign

**OpenVLA (Stanford, June 2024):**
- 7B parameters, open-source (Apache 2.0), on Hugging Face
- Outperforms RT-2-X (55B) by 16.5% on 29 manipulation tasks with 7x fewer parameters
- Output: 7-DOF end-effector deltas — same action space incompatibility
- LoRA fine-tuning in 10–15 hours on A100
- **Value for construction:** The visual reasoning and language grounding are genuinely useful. Challenge is adapting the action head to output hydraulic valve commands (continuous, multi-DOF, non-end-effector) and collecting domain-relevant training data.

**π₀ (Physical Intelligence, October 2024):**
- PaLI-Gemma VLM + "action expert" module trained with **flow matching** (produces smooth 50 Hz action trajectories)
- Trained on 7 robots, 68 tasks + Open X-Embodiment
- Open-sourced as `openpi` with ALOHA and DROID checkpoints
- **π₀.5** generalizes to new homes not seen in training — the environmental generalization capability that construction sites will require
- Physical Intelligence raised $400M
- **Value for construction:** The flow matching architecture is the right approach for smooth continuous hydraulic valve command generation (unlike discrete token approaches). Would require: new action space definition (joint velocity/position vs. end-effector), domain-specific training data from real or simulated construction operations, new "action expert" head for the hydraulic action space.

**Octo (UC Berkeley / Stanford / CMU, RSS 2024):**
- 27M–93M parameters, diffusion policy head, open-source
- Trained on 800K trajectories from 25 OXE datasets
- **Most actionable for construction:** Fine-tuning to new observation spaces (force-torque inputs) and action spaces (joint position control) in a few hours on consumer GPUs. Modular design enables action head replacement.

**Open X-Embodiment Dataset:** 1M+ episodes from 22 robot embodiments at 21 institutions. **Critical gap: zero construction equipment data in OXE.** All data is tabletop manipulation or indoor navigation. Building a construction-domain equivalent dataset is a **12–36 month data collection effort** — and the startup that builds it owns a foundational moat.

### The Proven Baseline: PPO in Domain-Specific Simulation

While VLA approaches are aspirational, **Proximal Policy Optimization (PPO) in AGX Dynamics or Chrono** is the demonstrated working approach for construction equipment today:

- ETH Zurich (arXiv:2410.05093): RL-based control of 40-ton material handler — **deployed to real machine at operator-level performance**
- TERA simulator (arXiv:2501.01430): excavator path tracking, 1.376m RMSE vs. real machine
- Rock capture paper (arXiv:2510.04168): PPO on CAT 365 model in AGX, 0.80 success rate

This is the realistic 2-year technical roadmap: PPO policies for specific tasks (grading, trenching, loading) trained in AGX/Chrono, deployed via sim-to-real transfer techniques. VLA approaches become relevant when you have enough domain data.

---

## 5. Sim-to-Real Transfer

### The Quantified Gap

ETH Zurich's Aoshima/Serbin paper (arXiv:2310.05765) provides the only direct measurement: **~10% sim-to-real gap** in wheel loader bucket filling forces/motion. Critical finding: the gap showed **weak dependence on simulation fidelity** — a lower-resolution DEM model achieved nearly the same transfer performance as high-resolution. This means you don't need perfect soil simulation; you need well-calibrated bulk soil parameters.

### The Techniques That Work

**1. Neural Network Actuator Models**
Rather than analytically modeling hydraulic valve dynamics (highly nonlinear — pressure-dependent flow, temperature variation, mechanical hysteresis), train a neural network on real machine data to capture actuator behavior.

ETH Zurich approach: dual MLPs for slew motor (pressure prediction + velocity prediction using 10-step history of controls, pressure, velocity, and configuration-dependent inertia). First-order systems with delay for arm joints. Result: controller transfers directly to 40-ton real machine **without fine-tuning**, achieving operator-level speeds with reduced tool oscillations.

**2. Action Delay Simulation**
Hydraulic systems have significant command-to-response delays (tens to hundreds of milliseconds depending on valve type, line length, oil viscosity). Explicitly training with simulated action delays is necessary for successful real-world transfer.

**3. Domain Randomization on Actuator Parameters**
Randomize soil cohesion, friction angle, bulk density, hydraulic delays (±20–50ms), tool inertia (empty vs. full bucket: 2x+ mass difference), and terrain surface roughness. The "noise" in simulated hydraulic load acts as implicit domain randomization — policies trained this way are inherently robust to parameter mismatch.

**4. Smooth Control Penalties**
Bang-bang (on/off) control in simulation is physically impossible with hydraulic actuators. Reward penalties for erratic control actions prevent policies from learning strategies that work in simulation but not hardware.

**5. Sim-to-Seg for Perception**
Sim2Seg (ICML 2023): Zero-real-data autonomous off-road driving by translating between sim images and segmentation/depth maps, then applying translation to real images. Directly applicable to construction site visual perception — eliminates need for large real-world labeled datasets.

**6. Adaptive Terrain Curriculum**
Adaptive Diffusion Terrain Generator (ADTG, arXiv:2410.10766): Diffusion models generate terrain geometries that progressively challenge the learning agent — curriculum learning for terrain traversal. Applicable to tracked equipment navigation policies.

---

### Sim-to-Real Results Table

| Paper | Method | Machine | Key Finding |
|---|---|---|---|
| Aoshima & Servin, 2024 | DEM sim-to-real gap study | Wheel loader | ~10% gap; weak fidelity dependence |
| ETH Zurich (arXiv:2410.05093) | NN actuator model + PPO | 40-ton material handler | **Direct transfer, operator-level performance** |
| TERA (arXiv:2501.01430) | Unity/AGX simulation | Excavator | 1.376m RMSE vs. real machine path |
| Rock capture (arXiv:2510.04168) | PPO in AGX | CAT 365 excavator | 0.80 success rate in sim |
| Active suspension (SciDirect 2024) | RL + domain rand | Forestry hydraulic vehicle | Near-equal sim/real with DR + delays |

---

## Technical Architecture Summary

For a construction autonomy startup building toward the first demo:

```
TRAINING STACK
├── Physics: AGX Dynamics (digging, loading) + Chrono::CRM (grading, dozing)
├── Perception synthesis: NVIDIA Isaac Sim (LiDAR/camera data generation)
├── Policy training: PPO via stable-baselines3 or custom RL framework
└── Controller design: MATLAB/Simulink Simscape (hydraulics, HIL testing)

REAL-WORLD STACK
├── Localization: u-blox ZED-F9P RTK + ESKF IMU fusion (tightly coupled)
├── Perception: Ouster OS1/OS2 or Hesai AT128 + ZED 2i stereo camera
├── Machine controller: Parker IQAN-MC31 + IQANdesign
├── Hydraulic interface: CAN J1939 + proportional valve interception
├── Safety: Dual-channel safety relay + FORT wireless E-stop + geofence
└── Compute: NVIDIA Jetson AGX Orin (edge inference, sensor fusion)

AI LAYER
├── Near-term: Task-specific PPO policies (grade, trench, load, haul)
├── Medium-term: Fine-tuned Octo/OpenVLA with construction-domain data
└── Long-term: Construction-native VLA trained on proprietary fleet data
```

---

*Sources: Algoryx AGX Dynamics documentation, NVIDIA Isaac Lab documentation, Project Chrono documentation, arXiv papers (2310.05765, 2501.01430, 2507.05643, 2410.05093, 2510.04168, 2106.05059, 2410.24164, 2406.09246, 2405.12213), Parker IQAN documentation, SAE J1939 specification, ISO 3691-4:2023, Trimble Earthworks documentation, MathWorks Autonomous Excavator demo.*
