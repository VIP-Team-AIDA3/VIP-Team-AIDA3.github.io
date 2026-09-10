## The Tailsitter

In this project, the team will build, test, and fly a tailsitter VTOL UAV: a fixed-wing aircraft that takes off and lands on its tail, hovers, and pitches over to cruise on the wing, using the same rotors and elevons for every flight phase. Requirements will be given to the team for a model which is able to fly in the Purdue UAS research and test facility. Scope runs from comparing requirements and a trade study against given kits and published recipes through fabrication (foam and/or lightweight 3D print) and propulsion, piloted flight test and telemetry data collection and aerodynamics analysis. Later the project will incorporate sensor integration,  the ArduPilot/PX4 sim-to-real pipeline including hardware-in-the-loop, as well as autonomous waypoint flight with perception on a companion microcontroller rather than a Linux computer, and a closing stress-test campaign that converts crashes and near-misses into documented operating limits.

A tailsitter rests on its tail for takeoff and landing, then rotates approximately 90 degrees to fly as a conventional wing-borne aircraft. The arrangement offers vertical takeoff without a runway while retaining the potential range and efficiency of fixed-wing cruise. In many designs, the same motors and propellers provide lift in hover and thrust in cruise, avoiding separate lift motors or mechanically complex tilt mechanisms.
<div align="center">
<img width="512" height="384" alt="image" src="https://github.com/user-attachments/assets/4590bd48-5aa6-4af2-9012-016708ea2d93" />
</div>

The architecture has gained renewed attention in modeling, research, and commercial surveying. PX4 and ArduPilot both support tailsitter configurations; open research platforms such as Phoenix provide reusable CAD, BOM, and simulation resources; and commercial aircraft use tailsitter layouts for mapping, multispectral imaging, and LiDAR missions. 


