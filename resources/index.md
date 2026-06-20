# Resources

Here is the introduction [presentation](https://docs.google.com/presentation/d/1_Ht_8tzPB61Tt3jDoSm8XyahU1n4aTsWaPcUqu2fs0w/edit?usp=sharing) - 19th June 2026.

## Subteam Project Descriptions

### Reinforcement Learning for Guidance, Navigation, and Control
This subteam explores learning-based policies for path-planning for fixed-wing UAVs using a gymnasium environment. The research focus is Reinforcement Learning for guidance and control (policy learning, reward design, sim-to-real strategies) and how learned policies compare to classical baselines under disturbances and modeling error. 

Expected activities include: 1) Literature review 2) Mathematical Problem Formulation 3) Coding a Gymnasium Environment for Dynamic path-planning (no-ROS) that is compatible with the Simulator used for the Fixed-wing drone competition 4) Integration of the RL path-planner with a PID control and State-Simulator 5) Training the RL model for  fixed-wing drone in the developed Gymnasium 6) Validating the Path-planner in the Simulator of the Competition, and 7) Deploying it on a real fixed-wing vehicle to be flown in PURT. 

### System Identification (SysID) for UAVs 
This subteam focuses on System identification (SysID) with a focus on guidance,navigation, and control (GNC). SysID focuses on accurately simulating and predicting the behavior of UAVs, in particular during abnormal flight conditions such as certain maneuvers or external disturbances. SysID also sets the foundation for model-predictive control (MPC), based on accurate state estimation. SysID methods subsume classical and data-driven methods including frequency-based methods, time-domain methods range from output error estimation to neural network-based approaches. The task of SysID is to predict the system’s future states but also learning or estimating the parameters of ordinary differential equations (ODE) or even the functional terms of the ODE describing the behavior of the UAV based on input and output data (e.g. velocity, flight path angle, angle of attack, pitch rate). Ongoing research focuses on methods for parameter estimation as well as uncertainty quantification (UQ) and residual approximation  in SysID using Bayesian methods. Team members learn about the mathematical foundations of SysID based on readings of foundational literature (e.g. scientific literature), and implement methods based on simulated data of an ordinary differential equation (ODE) of a second-order dynamical system (Mass-Damper) before moving on to 3-DOF aircraft modeling and real-world 6-DOF aircraft data from our industry partner Windracers US LLC and our own collected data using a downscaled version (Sportscub). The final SysID results will be used for the digital twin simulator, used in the fixed-wing drone competition and also for SysID of the industry partner Windracers US LLC. Potential collaboration with other industry partners is also possible (Skydio etc.).
Expected activities include: 1) Literature review 2) Mathematical Formulation of the Method 3) Simulation, Data Collection and Preprocessing 4) Model Implementation for Bayesian SySiD 5) Software engineering and vehicle config publishing (e.g. log-to-parameter pipeline), 6) Simulation and sensitivity analysis. 
Researchers also have the opportunity to learn how to fly fixed-wing drones, and collect flight data through the collaboration with PURT and SATT. 

### Vision-based Navigation and Runway Detection for Safe Landing
This subteam uses computer vision algorithms for runway detection and automated landing.
 The focus is on creating and improving computer vision and deep learning models towards this end. The process involves - data collection, training, evaluation, and testing to achieve realistic automated landing of a fixed-wing on a runway. 

Expected activities include: 1) Literature review 2) Data collection and scraping, 2) Model Development (building vision models, both classical and deep), 3) Evaluation and testing 

The team member’s tasks will be closely defined in collaboration with the industry partner Windracers LLC and other senior researchers within AIDA3.

### Large Language Models for safe UAV operations
This subteam supports research on conversational mission-planning and operations, jointly with PhD students and other researchers. It focuses specifically on the safety of language-based instructions and communications between operators, UAVs, and air-traffic-control. Formal methods such as propositional logic and signal temporal logic play an important role in this. In addition, formal grammars also support methods such as prompt-engineering and fine-tuning of LLMs. This team will focus its efforts on such formal methods and grammars in order to increase the safety of conversational path-planning. The mentors will introduce the team members to ATC communication, STL, and formal grammars. The goal is to build a ground-truth dataset as well as to implement human-subject experiments to evaluate an existing chatbot system, and also mine additional data to fuse ATC communication data with ADS-B and other geo-spatial information. F


Individual tasks 1) review of existing datasets 2) data mining and data fusion related to speech, text, ADS-B and other geospatial data 3) labeling of ATC communication datasets, 4) testing and evaluation of speech-to-text-to-speech communication, 5) user-interface design improvements, 6) publishing produced data as open data for other researchers 7) recruitment of pilots, ATC experts, and students into chatbot testing, and support of A-B testing and real-world testing. 


All activities will be performed in alignment with our industry partners, and other stakeholders such as ATC operators, pilots, etc. 
