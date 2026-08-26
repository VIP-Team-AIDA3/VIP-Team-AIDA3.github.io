## Reinforcement Learning for Guidance, Navigation, and Control

This subteam explores learning-based policies for path-planning for fixed-wing UAVs using a Gymnasium environment. The research focus is Reinforcement Learning for guidance and control (policy learning, reward design, sim-to-real strategies) and how learned policies compare to classical baselines under disturbances and modeling error.

**Expected activities include:**

1. Literature review
2. Mathematical Problem Formulation
3. Coding a Gymnasium Environment for Dynamic path-planning (no-ROS) that is compatible with the Simulator used for the Fixed-wing drone competition
4. Integration of the RL path-planner with a PID control and State-Simulator
5. Training the RL model for a fixed-wing drone in the developed Gymnasium
6. Validating the path-planner in the Simulator of the Competition
7. Deploying it on a real fixed-wing vehicle to be flown in PURT.

---

### Past Work & Milestones

**Spring 2026**

* Formed the subteam and established our core research focus.
* Experimented with ROS 2 and foundational Reinforcement Learning concepts utilizing the *Grokking Deep Reinforcement Learning* textbook.
* Delivered a research talk at the Purdue Undergraduate Symposium, featuring a Q-Learning agent focused on navigation successfully flying a virtual Purdue UAS Research and Test Facility (PURT) course.

**Summer 2026**

* Created a series of mini-tutorials covering RL, Dynamic Programming (DP), Monte Carlo (MC), Temporal Difference (TD), SARSA, and Q-Learning in a demo environment.
* Deepened our theoretical foundation by studying from the Sutton & Barto textbook.
* Built a simplified demo environment to effectively test, validate, and compare all of our agents.
* Presented our ongoing progress with a research poster at the Summer Purdue Undergraduate Symposium.

---

### Future Goals (Fall 2026 & Beyond)

* **ICRA Submission:** Targeting a paper submission for the International Conference on Robotics and Automation (ICRA) in the upcoming weeks.
* **Pre-Planner Implementation:** Implementing our complete environment and agents for the path pre-planner.
* **Advanced Additions:** Planning future expansions to the system, including a dynamic planner to handle real-time disturbances and the integration of Signal Temporal Logic (STL) for formal trajectory specifications.


