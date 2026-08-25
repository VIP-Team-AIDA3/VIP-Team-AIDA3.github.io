# VIP Team AIrTonomy

## AI for Autonomy Research

*Machine learning, control, and formal methods for fixed-wing aircraft that fly themselves.*

:::{admonition} Welcome
:class: tip

This site is the public home for VIP Team AIrTonomy. Enrolled students should use Brightspace for announcements, assignments, grades, rubrics, and course resources. Mini-tutorials are posted here as they are released. An overview of the syllabus is below; the full syllabus is on Brightspace.
:::

## Team overview

AIrTonomy develops machine learning models for fixed-wing UAVs and competes in national and international autonomy competitions, both virtual and physical. Students organize into smaller subteams within the program's core research thrusts: perception, navigation, and control; digital twin and simulation; and projects in the broader area of AI for autonomous aviation such as human autonomy teaming.

Subteams establish new methods rather than reproduce existing ones. Recent directions include nonlinear model predictive control, reinforcement learning, physics-informed machine learning, generative AI, and formal methods for safety-critical language. Typical work is software development in Python, C++, and ROS 2, training and benchmarking models, building and testing simulation data pipelines, and running experiments in the wind tunnel and in flight.

Over multiple semesters, teams compete in different events and produce tools, datasets, and publications for autonomous aviation. Students who stay with the team often continue their work past the end of the semester and co-author papers with their mentors.

::::{grid} 1 2 2 4
:gutter: 2
:margin: 4 0 4 0

:::{grid-item-card} Perception and control
Vision-based navigation, learned and classical guidance, and state estimation.
:::

:::{grid-item-card} Digital twin
System identification, wind tunnel measurement, and simulation fidelity.
:::

:::{grid-item-card} Airspace intelligence
3D airspace geometry, live traffic, and geofencing from published FAA data.
:::

:::{grid-item-card} Human autonomy teaming
Formal grammars, temporal logic, and safe language between operators, aircraft, and ATC.
:::

::::

## Course information

| | |
| :--- | :--- |
| **Team meeting** | Wednesdays, 5:30 – 7:20 p.m. Eastern Time |
| **Location** | Physics Building, Room 223 |
| **Dates of instruction** | August 24 – December 12, 2026 |
| **Course numbers** | VIP 17910, 17911, 27920, 37920, 47920, 47921, 47922 |
| **Credit hours** | 2 credits: 27920, 37920, 47920 · 1 credit: 17910, 17911 · 0 credits: 47921, 47922 |
| **Registration** | [CRN lookup and registration](https://engineering.purdue.edu/vip/register) |

## Learning outcomes

By the end of the semester, students should be able to:

- apply engineering design and develop software, experimental, and data artifacts that advance autonomous aviation and meet specified design requirements, with consideration of safety, performance, robustness, and novelty;
- plan and conduct simulation, laboratory, and flight-test experiments, analyze telemetry, measurement, and annotated data against appropriate performance metrics, and use engineering judgment to improve autonomy performance;
- identify, formulate, and solve complex engineering problems arising in autonomous aviation by applying principles of controls, dynamics, estimation, statistical inference, formal methods, and software engineering;
- integrate autonomy software across a full pipeline using reproducible workflows and engineering best practices;
- function effectively on a multidisciplinary team, practicing the collaboration norms used in flight-test and autonomy engineering;
- communicate effectively in a written report, an oral presentation, and where appropriate a published dataset or research artifact;
- acquire and apply new knowledge as needed, using appropriate learning strategies; and
- recognize the ethical and professional responsibilities of this work, including the responsible conduct of human-subject research, the handling and public release of recorded data, and safe operation in shared airspace.

## Subteams

Students select a subteam in week 3. No prior aviation background is expected.

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} Reinforcement Learning for GNC
Learned path-planning policies for fixed-wing flight, trained in a Gymnasium environment matched to the competition simulator and benchmarked against classical baselines under disturbance and modeling error.

+++
[Read more](subteams/reinforcement-learning)
:::

:::{grid-item-card} System Identification for UAVs
Estimate the parameters, and sometimes the functional form, of the differential equations describing aircraft behavior, from a simulated mass-damper to real 6-DOF flight data from Windracers.

+++
[Read more](subteams/system-identification)
:::

:::{grid-item-card} Vision-based Navigation and Runway Detection
Find the runway from onboard imagery and use it to drive an automated landing, through the full loop of data collection, training, evaluation, and flight testing.

+++
[Read more](subteams/runway-detection)
:::

:::{grid-item-card} Wind Tunnel Testing and Aerodynamic Characterization
Measure lift and drag on a powered UAV in the Boeing Wind Tunnel, separate propulsive force from aerodynamic drag, and hand verified coefficients to the digital twin.

+++
[Read more](subteams/wind-tunnel)
:::

:::{grid-item-card} Bayesian Methods for Modeling and Uncertainty Quantification
Return a distribution over parameters instead of a single fit. Posterior inference, residual models for unmodeled dynamics, and calibration against held-out flight data.

+++
[Read more](subteams/bayesian-methods)
:::

:::{grid-item-card} Large Language Models for Safe UAV Operations
Formal grammars and signal temporal logic applied to conversational mission planning, evaluated through human-subject testing with pilots and controllers.

+++
[Read more](subteams/llm-safe-operations)
:::

:::{grid-item-card} Geospatial Airspace Modeling and Live Operational Awareness
Turn published FAA airspace boundaries into watertight 3D solids, add Part 77 surfaces and runway protection zones, then layer live restrictions, weather, and ADS-B traffic on top.

+++
[Read more](subteams/geospatial-airspace)
:::

:::{grid-item-card} ATC and ADS-B Data Mining
Record controller and pilot radio traffic, align it with ADS-B tracks on a common clock, and label it against a grammar of standard phraseology. The output is a public dataset and a paper.

+++
[Read more](subteams/atc-adsb-mining)
:::

::::

## Course Instructor

::::{grid} 1 1 2 2
:gutter: 3

:::{grid-item-card} Instructor and faculty mentor
**Prof. Sabine Brunswicker**

[sbrunswi@purdue.edu](mailto:sbrunswi@purdue.edu)

**Office:** MRGN 148

**Office hours:** by appointment
:::

::::

## Semester schedule (subject to change)

| Week | Date | Mini-tutorial | Design activity phase | PD deliverables | Student assignments and deliverables |
| :---: | :---: | :--- | :--- | :--- | :--- |
| 1 | 08/26/26 | Introduction to the team and logistics | Problem formulation / project description | | |
| 2 | 09/02/26 | | Problem formulation / project description | Activity 1 due Sept 4 at 11:59 PM | Team selection |
| 3 | 09/09/26 | Newton's laws of motion for a point mass | Problem formulation / project description | Professional Development Plan due Sept 11 at 11:59 PM | Research proposal: research question, technical plan (model, data, experiment, validation), timeline and roadmap, tools and frameworks to be used. Based on template, 1 page max |
| 4 | 09/16/26 | Coordinate systems in aerospace: Euler, quaternions, Euclidean | Data collection and modeling | | |
| 5 | 09/23/26 | Air traffic control: what it is and why it exists | Data collection and modeling | | |
| 6 | 09/30/26 | Fixed-wing aircraft and 3-DOF longitudinal equations of motion | Data collection and modeling | | |
| 7 | 10/07/26 | Basic probability | Data collection and modeling | | |
| 8 | 10/14/26 | Basic probability | Simulation, testing, validation | Activity 2 due Oct 14 at 11:59 PM | |
| 9 | 10/21/26 | Linear regression and gradient descent | Simulation, testing, validation | | |
| 10 | 10/28/26 | Linear regression and gradient descent | Simulation, testing, validation | | |
| 11 | 11/04/26 | Neural networks and backpropagation | Simulation, testing, validation | | |
| 12 | 11/11/26 | Convolutional networks and basic computer vision | Simulation, testing, validation | | |
| 13 | 11/18/26 | Formal logic | Simulation, testing, validation | Activity 3 due Nov 17 at 11:59 PM | |
| 14 | 11/25/26 | Formal logic | Analyses and implications | | |
| 15 | 12/02/26 | Signal temporal logic | Analyses and implications | | |
| 16 | 12/09/26 | Formal grammars | Analyses and implications | Activities 4 through 10 due Dec 11 at 11:59 PM | Final paper and presentations, submission due Dec 11 at 11:59 PM |

## Deliverables and grading

Each student produces a one-page research proposal aligned with their subteam, regular research progress updates, a final presentation and written report with supporting videos and figures, and final code committed to the team repository. Beyond the required deliverables, subteams commonly produce Overleaf manuscripts, published datasets, simulation demos, tutorial write-ups, and conference abstracts co-authored with mentors.

| Assignment | Due | Weight |
| :--- | :---: | :---: |
| CATME and VIP activities | Ongoing | 5% |
| Competition Activities | Ongoing | 5% |
| Quizzes | Ongoing | 5% |
| Professional development activities | Ongoing | 10% |
| One-page research proposal | September 13, 2026 | 15% |
| Final presentation | December 11, 2026 | 20% |
| Final report | December 11, 2026 | 25% |

Grades reflect accomplishments and effort, documentation, and teamwork and interactions, assessed once at mid-semester as formative feedback and again at the end of the semester. Full criteria and rubrics are posted on Brightspace.

## Research center and industry partner

This VIP team is organized by Prof. Sabine Brunswicker and collaborates with AIDA3, the Center on AI for Digital, Autonomous and Augmented Aviation, founded by Purdue University and Windracers in 2024. The center develops safe and trustworthy AI for aerial autonomy across onboard intelligence and control, human autonomy teaming, supply chain and airspace intelligence, remote sensing, and cybersecurity. It is based at the smart operating center at the Purdue Augmented Aviation Lab, where students and faculty test their work on real aircraft.

Windracers is the founding industry partner. The UK company builds and operates the ULTRA, a twin-engine fixed-wing UAV that has flown delivery and logistics missions for the Royal Mail, the Royal Navy, and the British Antarctic Survey. One ULTRA, named Earhart, made its first fully automated U.S. flight from Jasper County Airport in April 2026 and is now used for research at the center. For students, the partnership means regular check-ins with a working engineering team and access to flight data from aircraft with a real operational record.

## Course platforms

::::{grid} 1 1 3 3
:gutter: 2

:::{grid-item-card} Brightspace
Announcements, schedules, assignments, grades, feedback, rubrics, and course resources.

[Open Purdue Brightspace](https://purdue.brightspace.com/)
:::

:::{grid-item-card} CATME
Peer and team evaluations, submitted at mid-semester and at the end of the semester.

[Open CATME](https://info.catme.org/)
:::

:::{grid-item-card} GitHub
Team repositories, mini-tutorials, and project-specific pages maintained by each subteam.

[Open the team organization](https://github.com/VIP-Team-AIDA3)
:::

::::

## Technology requirements

Tools vary by subteam. All students use the following:

- **Python 3.10 or newer**, and C++ where a subteam requires it
- **Jupyter Notebook or JupyterLab**
- **Git and GitHub**, including the team repositories
- **LaTeX and Overleaf** for reports and manuscripts
- **MS Office and Adobe PDF** for course deliverables
- A dedicated Python virtual environment for subteam packages
- An editor such as Visual Studio Code, recommended but not required

Subteam-specific tools include Gymnasium and PyTorch for reinforcement learning, MATLAB and log-processing pipelines for system identification, OpenCV for vision, LabVIEW for wind tunnel acquisition, PyMC or NumPyro for Bayesian methods, QGIS with shapely, pyproj, and trimesh for geospatial work, and speech and grammar tooling for the ATC dataset. ROS 2, Modelica, and CasADi are optional and used where a project calls for them.

## Additional resources

- [AIDA3 research center](https://www.purdue.edu/computes/aida3/)
- [AIDA3 and Windracers ULTRA flight](https://youtu.be/gs_ZTh7p5bM)
- [LLM route planning demo](https://youtu.be/OmfWZqO04Ug)
- [VIP @ Purdue program](https://engineering.purdue.edu/vip/)
- General VIP questions: [vip@purdue.edu](mailto:vip@purdue.edu) · West Lafayette: [VIP_WL@purdue.edu](mailto:VIP_WL@purdue.edu)

:::{note}
Content and details may be updated during the semester. Mini-tutorials and subteam pages will be posted on this website as they are released.
:::
