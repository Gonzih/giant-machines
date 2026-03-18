# Autonomous Construction Equipment — Company Foundation Documents

This repository contains the foundational research and strategic vision for a startup building AI-powered autonomy for bulldozers, excavators, graders, and tractors.

## Documents

### [`VISION.md`](./VISION.md)
**Start here.** The company-defining document: thesis, problem statement, why now, demo concept, 3-year roadmap, defensibility, and founding team's unfair advantage.

### [`docs/landscape.md`](./docs/landscape.md)
**Market & competitive analysis.** Who is working on autonomous construction equipment right now, what they've shipped vs. what's vaporware, total addressable market ($4.4B → $9.77B by 2030 at 14.2% CAGR), regulatory pathways, labor shortage data (501K workers short), and where the gaps are.

### [`docs/technical-landscape.md`](./docs/technical-landscape.md)
**Technical architecture research.** Best simulation environments for off-road heavy equipment (AGX Dynamics vs. NVIDIA Isaac Sim vs. Project Chrono), perception stack (LiDAR, stereo cameras, RTK GPS, IMU fusion), control systems (CAN bus J1939, hydraulic actuation, Parker IQAN), foundation models for robotics (RT-2, OpenVLA, π₀), and sim-to-real transfer techniques that work.

### [`docs/demo-concept.md`](./docs/demo-concept.md)
**Raspberry Pi + tractor demo plan.** Deep dive into AgOpenGPS (the open-source tractor autonomy stack), hardware bill of materials for minimum viable and full-featured demos, the 8-minute investor demo script, and what makes a compelling hardware demo at each funding stage.

### [`docs/go-to-market.md`](./docs/go-to-market.md)
**Incubators, funding, and pitch strategy.** Suffolk Technologies BOOST, HAX, Y Combinator, DCVC, Lux Capital, Builders VC, and defense-adjacent funding (SBIR, DIU, AFWERX). What a winning pitch looks like, milestone gates by funding stage (pre-seed through Series A), and the demo bar at each stage.

### [`docs/naming.md`](./docs/naming.md)
**Naming research and proposals.** Analysis of "Neural Automation" availability, 10 name proposals with rationale, naming principles for the space.

---

## Key Facts for Pitching

**The problem:** 92% of US construction firms can't fill open positions (AGC 2025). 501,000 workers short in 2024. 21% of the workforce is 55+. McKinsey: $1.6T/year in value locked in the productivity gap.

**The market:** $4.4B (2024) → $9.77B (2030) autonomous construction equipment TAM, 14.2% CAGR (MarketsandMarkets).

**The approach:** RL policies trained in physics-accurate simulation (AGX Dynamics + Chrono), deployed via retrofit hardware module on any CAN-bus machine, OEM-agnostic.

**The demo:** A tractor connected to a Raspberry Pi that drives itself. $1,000 in electronics. Commercial-grade ±2.5 cm precision.

**The moat:** Proprietary construction-domain training data. OEM-agnostic software layer. ML infrastructure expertise no incumbent can match.

**Comparable:** Bedrock Robotics (ex-Waymo, $1.75B valuation, $350M raised in 2025) is the benchmark for what this space can become.

---

## Research Date

March 2026. Market data, funding figures, and company statuses current as of this date.
