# Vision: Autonomous Machines for the Physical World

*Version 1.0 — March 2026*

---

## One-Line Thesis

We build the AI brain that makes bulldozers, excavators, and graders operate themselves — training in simulation, deploying on any machine, starting with a tractor that drives itself.

---

## The Problem (In Vivid Terms)

A $500 billion construction project just fell behind schedule. Not because of bad weather, not because of engineering failures, not because of supply chain delays — because there aren't enough people who know how to run a bulldozer.

This is happening right now, everywhere, all the time.

**92% of US construction firms** can't fill their open positions (AGC, 2025). The industry is short **501,000 workers** and falling further behind every year. The people who know how to operate heavy equipment — the excavator operators, the grader pilots, the dozer drivers — are retiring at an accelerating pace. **21% of the construction workforce is over 55.** Their replacements aren't showing up.

Meanwhile, the demand side is exploding. AI data centers require billions of square feet of new construction. The US infrastructure bill is pouring $1.2 trillion into roads, bridges, and waterways. Disaster recovery from increasingly severe weather events needs construction crews that don't exist. The cost of doing nothing: **$10.8 billion per year** in lost economic output just from the housing shortage alone (HBI). McKinsey's estimate: if construction productivity caught up to the rest of the economy, it would unlock **$1.6 trillion per year** in additional value.

The machines are ready. The technology is ready. The only missing ingredient is the software that makes them go.

---

## Why Now

Four forces are converging in 2026 that make this the right moment:

### 1. The Labor Crisis is Structural, Not Cyclical

This isn't a post-COVID blip. The construction workforce has been aging for two decades. The pipeline of new operators is broken. Immigration policy is tightening, cutting off the labor supply that previously kept shortages manageable. The AI/data center construction boom is now competing for the same limited pool of workers as housing, infrastructure, and disaster recovery. This gets worse every year. There is no policy fix on the horizon.

### 2. Simulation Has Crossed the Threshold

Training autonomous systems for real machines used to require years of real-world data collection. Today, physics engines like AGX Dynamics simulate soil deformation, hydraulic actuation, and terrain interaction with 75–90% accuracy at real-time speeds. Project Chrono running on an H100 can simulate 29 kilometers of deformable terrain. NVIDIA Isaac Lab can run 90,000 training iterations per second.

ETH Zurich recently demonstrated an RL policy trained entirely in simulation and deployed directly to a 40-ton real machine — at operator-level performance, without fine-tuning. The sim-to-real gap for construction equipment is approximately 10%, and the techniques to close it are understood.

### 3. Sensors Got Cheap and Reliable

A LiDAR that costs $700 today (Livox Mid-360) achieves what required $75,000 in hardware a decade ago. RTK GPS modules are $130. The u-blox ZED-F9P achieves ±2.5 cm accuracy — better than a trained operator — for $200. The sensor stack for a compelling autonomous tractor demo is under $5,000.

### 4. Retrofit-First Is Proven and Preferred

Bedrock Robotics ($1.75B valuation), Built Robotics ($112M), SafeAI ($68M, acquired by Pronto) — the entire successful cohort of autonomous construction equipment startups has been retrofit-first. Attach to the existing machine fleet. Don't ask contractors to buy new equipment. Don't compete with Caterpillar. Own the intelligence layer that works on any brand. This is the right business model and it's now validated.

---

## The Demo That Proves the Thesis

**A tractor connected to a Raspberry Pi that drives itself.**

This is not a research curiosity. It is a proof of the entire thesis, compressed into something a 10-year-old can understand and a venture capitalist can feel in their gut.

The demo sequence:
1. A compact farm tractor, otherwise unmodified, sits in a field.
2. An operator types a field boundary and a path into a tablet. No other input.
3. The tractor starts moving. It follows the path within 2 centimeters. It reaches the headland. It lifts its implement, executes a U-turn, comes back on the next parallel pass.
4. Someone walks in front of it. It stops. They move. It resumes.
5. The operator shows the GPS track on screen: two parallel passes, 1 meter apart, both within 2 cm of the programmed line.
6. "This kit costs $1,000. The commercial version costs $25,000. It runs on any tractor you already own. One operator can run five of these machines simultaneously."

This demo proves:
- The physics problem is solved (cm-level positioning, real-time control)
- The AI safety problem is managed (obstacle detection, E-stop, geofencing)
- The business model is right (retrofit, not replacement)
- The team can ship (it's running right now, not in a slide)

**What makes this specific demo powerful:** A tractor is universally understood. Everyone's grandfather drove one. The simplicity of "it drives itself" maps directly onto "a $750,000 excavator does this too." The demo is a scaling story wrapped in something tangible.

---

## The Three-Year Roadmap

### Year 1: Prove It Works on One Machine
- Complete the tractor demo (Raspberry Pi + RTK GPS + LiDAR + ROS2 on a Kubota BX)
- Apply to and complete Suffolk BOOST accelerator + HAX; get first $400K–$700K
- Secure SBIR Phase I ($150K non-dilutive)
- Deploy on one real construction site with one named customer (GC partner or equipment rental company)
- Publish first dataset of construction equipment telemetry from real deployments
- Raise seed: $2M–$4M; primary investors: Builders VC, DCVC, or Lux Capital
- Hire: hardware co-founder / ML engineer with robotics background

### Year 2: Prove It Works at Scale
- Retrofit system running on 5–10 machines at 3–5 customer sites
- Three machine types: tractor/dozer, excavator, grader
- SBIR Phase II: $1.5M–$3M non-dilutive
- 1,000+ hours of autonomous operation logged; zero safety incidents
- Beginning to build the proprietary construction-domain training dataset (the long-term moat)
- First recurring revenue: $500K ARR on subscription/service model
- Raise Series A: $10M–$15M

### Year 3: Own the Construction Site
- 50+ machines across 20+ customers
- Fleet coordination software: multiple machines on a single site, coordinated via shared map
- AI training pipeline producing domain-specific policies better than anything trained on generic robot data
- Beginning OEM conversations: could be licensed into Komatsu/Volvo/CNH as the intelligence layer
- $3M–$5M ARR
- Raise Series B: $30M–$50M

---

## What Makes This Defensible

### 1. The Data Flywheel
Every hour of real-world autonomous operation produces training data that makes the system smarter. The first mover who builds a fleet accumulates proprietary construction-domain data that competitors cannot buy. This is the same moat that made Waymo valuable: not the hardware, but the miles. **The construction industry has never had a systematic data collection effort for machine operation.** The company that starts now owns the dataset.

### 2. ML Infrastructure Expertise Is Rare in This Domain
Komatsu has been selling machine control systems since 2013. Their engineers know hydraulics. They don't know how to build a foundation model training pipeline, run distributed RL at scale, or close the sim-to-real gap for a new machine type in days rather than months. This is the founder's native capability. 18 years of ML infrastructure means knowing how to scale training, how to build evaluation pipelines, how to deploy models to hardware with constrained compute. The incumbents are hiring for this. You start with it.

### 3. OEM-Agnostic Moat
Komatsu's iMC only works on Komatsu machines. Caterpillar's Grade only works on Cat machines. Every construction site has 4–6 different equipment brands. The company that builds an intelligence layer that works on any CAN-bus machine — regardless of OEM — becomes the Mobileye of construction equipment. OEMs either license from you or acquire you. They don't outcompete you because they can't sell to each other's customers.

### 4. Private Site Regulatory Advantage
The regulatory moat for public roads is brutal: 50-state compliance, FMCSA rules, liability hell. Construction sites are private, geofenced, controlled. The regulatory environment is closer to a mine than a highway. Caterpillar has operated 690 autonomous trucks in mines with zero injuries for 11 years. The path is proven. Private site autonomy doesn't require public road approval.

### 5. Network Effects in Safety Records
Once you have 1,000+ hours of incident-free operation, you have the most powerful sales tool in construction: a documented safety record. Insurance companies lower premiums. General contractors approve faster. Regulators cite your track record. This compounds. The first company to 1 million incident-free autonomous hours owns the safety narrative for a generation.

---

## Founding Team's Unfair Advantage

**18 years in ML infrastructure and autonomous agent systems.**

This is not a background in robotics from the outside. It is direct experience building the systems that make AI agents work at scale: training pipelines, distributed compute, model deployment to hardware, evaluation frameworks, simulation environments, system reliability. These are exactly the hard problems in autonomous construction equipment:

- How do you train a policy on deformable terrain at scale? (Training infrastructure)
- How do you deploy a model to an edge compute unit on a machine with 4GB RAM? (Model optimization and deployment)
- How do you know if your simulation is accurate enough to trust for real-world deployment? (Evaluation and benchmarking)
- How do you update an autonomous system in the field without breaking it? (MLOps for hardware)

The incumbents (Komatsu, Caterpillar) are hardware companies trying to bolt on AI. Bedrock Robotics is an autonomous driving company pivoting to construction. This company is **an ML infrastructure company applying its core competency to the most underautomated heavy-equipment market in the world.** That is a different and defensible position.

---

## The Honest Risks

**Technical risks (manageable):**
- Sim-to-real transfer gaps will require real-world data collection to close fully — plan for it
- Construction sites are GPS-occluded in ways mining sites are not — requires robust fallback to LiDAR SLAM
- CAN bus access varies by machine model and year — will require hardware partnerships or reverse engineering

**Market risks (real but not fatal):**
- Bedrock Robotics has a $1.75B head start with $350M to deploy; competing head-to-head in mass excavation is inadvisable
- Caterpillar's CES 2026 construction autonomy announcement signals they are moving fast; the window for retrofit leadership is 2–3 years
- Construction contractor sales cycles are long (6–18 months) and relationship-driven; need a named GC partner early

**Mitigations:**
- Start in agricultural tractors (faster sales cycles, simpler terrain, proven open-source foundation) then expand to construction
- Target machine types Bedrock doesn't (tractors, graders, compactors, telehandlers) before competing on excavators
- Build the OEM-agnostic positioning early; frame for OEM partnership/licensing conversations from year 1

---

## The One-Page Version

**What:** AI autonomy software for construction and agricultural equipment — trains in simulation, deploys on any machine via retrofit.

**Why:** 501K construction workers short in 2024. Machines are sitting idle. $1.6T/year in economic value locked up in a productivity gap that AI can close.

**How:** RL policies trained in physics-accurate simulation (AGX Dynamics + Chrono), deployed via a retrofit hardware module (Parker IQAN + LiDAR + RTK GPS), on any CAN-bus machine regardless of brand.

**Demo:** A tractor that drives itself. Today. For $1,000 in hardware.

**Business:** Retrofit kit + software subscription. $X per machine per month. One operator runs five machines. 12-month customer payback.

**Moat:** Proprietary construction-domain training data from every deployed machine. OEM-agnostic (works on any brand). Sim-to-real expertise no incumbent can match.

**Team:** 18 years building ML infrastructure and autonomous agent systems that work at scale. Exactly the capability the industry needs and cannot hire.

**Ask:** $X for X months of runway to first commercial deployment.

---

*This document is a living strategy document. Update as market intelligence, technical capabilities, and customer feedback evolve.*
