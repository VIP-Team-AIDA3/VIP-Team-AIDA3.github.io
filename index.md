# AI for Autonomy Research

**VIP Team AIDA3 · Purdue University · Fall 2026**

Machine learning, control, and formal methods for fixed-wing aircraft that fly themselves. Student subteams build the pieces of an autonomous aviation stack, from wind tunnel coefficients to airspace geometry to the language a controller uses on the radio. The work flies on real aircraft at the Purdue Augmented Aviation Lab.

| | |
|---|---|
| **Team meeting** | Wednesdays, 5:30–7:20 pm |
| **Location** | Physics Building, Room 223 |
| **Dates** | August 24 – December 12, 2026 |
| **Course numbers** | VIP 17910, 17911, 27920, 37920, 47920, 47921, 47922 |
| **Credit hours** | 2 cr: 27920, 37920, 47920 · 1 cr: 17910, 17911 · 0 cr: 47921, 47922 |
| **Register** | [engineering.purdue.edu/vip/register](https://engineering.purdue.edu/vip/register) |

## What this team does

AIDA3 develops machine learning models for fixed-wing UAVs and competes in national and international autonomy competitions, both virtual and physical. Students organize into smaller subteams within the program's core research thrusts: perception, navigation, and control; digital twin and simulation; and projects in the broader area of AI for autonomous aviation such as human autonomy teaming.

Subteams are encouraged to establish new methods rather than reproduce existing ones. Recent directions include nonlinear model predictive control, reinforcement learning, physics-informed machine learning, generative AI, and formal methods for safety-critical language. Typical work is software development in Python, C++, and ROS 2, training and benchmarking models, building and testing simulation data pipelines, and running experiments in the wind tunnel and in flight.

Over multiple semesters, teams compete in different events and produce tools, datasets, and publications for autonomous aviation. Students who stay with the team often continue their work past the end of the semester and co-author papers with their mentors.

## Subteams

You choose one in week 3. No prior aviation background is expected.

- **[Reinforcement Learning for Guidance, Navigation, and Control](subteams/reinforcement-learning)** — Learned path-planning policies trained in a Gymnasium environment and benchmarked against classical baselines.
- **[System Identification for UAVs](subteams/system-identification)** — Estimate the parameters and functional form of the equations describing how the aircraft behaves, from a simulated mass-damper to real 6-DOF flight data.
- **[Vision-based Navigation and Runway Detection](subteams/runway-detection)** — Find the runway from onboard imagery and use it to drive an automated landing.
- **[Wind Tunnel Testing and Aerodynamic Characterization](subteams/wind-tunnel)** — Measure lift and drag on a powered UAV in the Boeing Wind Tunnel and hand verified coefficients to the digital twin.
- **[Bayesian Methods for Modeling and Uncertainty Quantification](subteams/bayesian-methods)** — Distributions over parameters instead of point estimates, applied to UAV dynamics and state estimation.
- **[Large Language Models for Safe UAV Operations](subteams/llm-safe-operations)** — Formal grammars and temporal logic applied to conversational mission planning, evaluated with pilots and controllers.
- **[Geospatial Airspace Modeling and Live Operational Awareness](subteams/geospatial-airspace)** — Turn published FAA airspace boundaries into watertight 3D solids and layer live traffic on top.
- **[ATC and ADS-B Data Mining for Open Aviation Datasets](subteams/atc-adsb-mining)** — Record radio traffic, align it with ADS-B tracks, label it against a phraseology grammar, publish the dataset.

## The semester

Mini-tutorials run every week and start from Newton's laws. Schedule subject to change.

| Week | Mini-tutorial | Phase | Due |
|---|---|---|---|
| 1 · Aug 26 | Introduction to the team and logistics | Problem formulation | |
| 2 · Sep 2 | | | PD plan |
| 3 · Sep 9 | Newton's laws of motion for a point mass | Problem formulation | Subteam selection |
| 4 · Sep 16 | Coordinate systems in aerospace: Euler, quaternions, Euclidean | | |
| 5 · Sep 23 | Air traffic control: what it is and why it exists | Problem formulation | Research proposal |
| 6 · Sep 30 | Fixed-wing aircraft and 3-DOF longitudinal equations of motion | | |
| 7 · Oct 7 | Basic probability | Data collection and modeling | |
| 8 · Oct 14 | Basic probability | | Mid-semester IPE |
| 9 · Oct 21 | Linear regression and gradient descent | Data collection and modeling | |
| 10 · Oct 28 | Linear regression and gradient descent | | |
| 11 · Nov 4 | Neural networks and backpropagation | Data collection and modeling | Written report update |
| 12 · Nov 11 | Convolutional networks and basic computer vision | | |
| 13 · Nov 18 | Formal logic | Data collection and modeling | |
| 14 · Nov 25 | Formal logic | | |
| 15 · Dec 2 | Signal temporal logic | Simulation, testing, validation | |
| 16 · Dec 9 | Formal grammars | Final paper and presentations | Report and presentation |

## What you produce

1. A one-page research proposal aligned with your subteam: question, technical plan, validation approach, and timeline.
2. Research progress updates through the semester.
3. A final presentation and written report, with supporting videos, figures, and data.
4. Final code committed to the team repository, with your mentors' support.

Beyond the required deliverables, subteams commonly produce Overleaf manuscripts, published datasets, simulation demos, tutorial write-ups, and conference abstracts co-authored with mentors.

| Assignment | Weight |
|---|---|
| CATME and VIP activities | 5% |
| Quizzes | 10% |
| Professional development activities | 10% |
| One-page research proposal | 15% |
| Final presentation | 20% |
| Final report | 25% |

Grades reflect accomplishments and effort, documentation, and teamwork and interactions, assessed at mid-semester and again at the end. Full criteria and rubrics are on Brightspace.

## Who we work with

This VIP team is organized by Dr. Sabine Brunswicker and collaborates with AIDA3, the Center on AI for Digital, Autonomous and Augmented Aviation, founded by Purdue University and Windracers in 2024. The center develops safe and trustworthy AI for aerial autonomy across onboard intelligence and control, human autonomy teaming, supply chain and airspace intelligence, remote sensing, and cybersecurity. It is based at the smart operating center at the Purdue Augmented Aviation Lab.

Windracers is the founding industry partner. The UK company builds and operates the ULTRA, a twin-engine fixed-wing UAV that has flown delivery and logistics missions for the Royal Mail, the Royal Navy, and the British Antarctic Survey. One ULTRA, named Earhart, made its first fully automated U.S. flight from Jasper County Airport in April 2026 and is now used for research at the center. For students, the partnership means regular check-ins with a working engineering team and access to flight data from aircraft with a real operational record.

## Mentors

Office hours by appointment.

| Role | Name | Office | Email |
|---|---|---|---|
| Instructor / Faculty mentor | Dr. Sabine Brunswicker | MRGN 148 | sbrunswi@purdue.edu |
| Doctoral student / Mentor | Christopher Rashidian | — | crashid@purdue.edu |
| Graduate student / Mentor | Richard Ajagu | TBA | rajagu@purdue.edu |

## More

- [AIDA3 research center](https://www.purdue.edu/computes/aida3/)
- [AIDA3 and Windracers ULTRA flight](https://youtu.be/gs_ZTh7p5bM)
- [LLM route planning demo](https://youtu.be/OmfWZqO04Ug)
- [VIP @ Purdue program](https://engineering.purdue.edu/vip/)
- General VIP questions: vip@purdue.edu · West Lafayette: VIP_WL@purdue.edu
