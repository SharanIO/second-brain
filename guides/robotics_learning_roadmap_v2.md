# Robotics Career Acceleration Roadmap (2026)

A curated, opinionated guide focused on **AI for robotics**, **firmware/embedded**, and **systems in highest demand** right now. Built for someone already past beginner level — assumes you know Python/C++, basic ROS, and have hardware-software bridging experience.

> **Strategic note:** The single biggest shift in 2025–26 is the rise of **Vision-Language-Action (VLA) models** (NVIDIA GR00T, π0, SmolVLA, RT-2 successors), **massively parallel sim training** (Isaac Lab), and **VLA evaluation pipelines** (Isaac Lab-Arena × Hugging Face LeRobot). If you only have time for one new area, pick this one — it's where research and industry are converging.

---

## 1. AI / ML for Robotics — the highest-leverage area

### 1.1 Vision-Language-Action models & Generalist Policies (priority #1)

| Resource | What it teaches | Cost | Why it matters |
|---|---|---|---|
| **Hugging Face LeRobot** ([huggingface.co/lerobot](https://huggingface.co/lerobot)) | End-to-end VLA training (ACT, Diffusion Policy, π0, SmolVLA, GR00T N) on real cheap hardware (SO-100/SO-101 arms ~$200) | Free | THE de facto open-source robotics learning stack in 2026. Build a portfolio piece here. |
| **HF Deep RL Course** ([huggingface.co/learn/deep-rl-course](https://huggingface.co/learn/deep-rl-course)) | RL from PPO to offline RL, with certificate | Free | Hands-on, cert-bearing, recent |
| **NVIDIA Physical AI Learning Path** ([nvidia.com/learn/learning-path/robotics](https://www.nvidia.com/en-us/learn/learning-path/robotics/)) | Isaac Sim → Isaac Lab (massively parallel RL) → Isaac ROS → OpenUSD | Free | Industry-standard sim stack; pairs with NVIDIA DLI certs |
| **NVIDIA Isaac GR00T N1.5 / N1.6** (GitHub: NVIDIA/Isaac-GR00T) | Open foundation model for humanoids, fine-tuning workflows | Free | Run end-to-end on Jetson Thor; resume gold |

### 1.2 Reinforcement Learning (the deep dive)

- **UC Berkeley CS285 — Deep RL** (Sergey Levine) — full lectures + assignments on YouTube. *The* serious RL course. Free.
- **DeepMind × UCL RL Course** (David Silver / newer Hado van Hasselt versions) — YouTube. Free.
- **OpenAI Spinning Up** — best practical PPO/SAC walkthrough. Free.
- **Sutton & Barto, *Reinforcement Learning: An Introduction*** — free PDF. The textbook.

### 1.3 Imitation Learning, Behavior Cloning & Diffusion Policy (deep dive)

This is *the* fastest-moving subarea of robot learning right now and the most directly relevant to your tactile sensing + manipulation work. Treat this as a self-contained track.

#### Foundations: courses, surveys, textbooks (all free)

- **Russ Tedrake — *Underactuated Robotics* Ch. 21: Imitation Learning** ([underactuated.mit.edu/imitation.html](https://underactuated.mit.edu/imitation.html)) — the canonical free chapter. Tedrake himself is a Diffusion Policy co-author. Walks BC → ACT → Diffusion Policy → VLA in one cohesive narrative. **Read this first.**
- **Hugging Face — *Robot Learning: A Tutorial*** (Cattaneo et al., arXiv 2510.12403, Oct 2025) — comprehensive modern tutorial paper with executable code in LeRobot. The companion code is the same library you'd use for your own projects.
- **UC Berkeley CS285 — Lectures 2 & 3** (Sergey Levine) — formal treatment of BC, DAgger, distribution shift, with assignment code. Free YouTube + assignments on the course page.
- **CMU 16-831 — Statistical Techniques in Robotics** (Drew Bagnell originated DAgger here) — slides + assignments online, free.
- **CMU 10-403 — Deep Reinforcement Learning** — covers IL/BC modules; Katerina Fragkiadaki teaches the manipulation-relevant material.
- **Pascal Poupart CS885 — Imitation Learning lecture** (Waterloo) — focused 30-min intro covering BC, GAIL, IRL. Free YouTube.
- **Osa et al. (2018) — *An Algorithmic Perspective on Imitation Learning*** (Foundations and Trends in Robotics) — *the* survey paper. Free PDF.
- **Urain et al. (2026) — *Deep Generative Models in Robotics: A Survey on Learning from Multimodal Demonstrations*** (IEEE T-RO) — most current survey; covers DP, VLAs, flow matching policies.

#### Behavior Cloning specifically

- **Pomerleau (1988) — ALVINN** — the original BC paper (autonomous driving). Read for context.
- **Bain & Sammut (1995) — *A framework for behavioural cloning*** — formal foundation.
- **Ross & Bagnell (2011) — DAgger** — the canonical fix for compounding errors / distribution shift.
- **Florence et al. (2022) — *Implicit Behavioral Cloning*** — energy-based formulation; intellectual precursor to Diffusion Policy. Code on Google Research GitHub.
- **Robomimic** ([robomimic.github.io](https://robomimic.github.io/)) — ARISE Initiative's benchmark suite + reference implementations of BC, BC-RNN, IRIS, GMM-MLP. Best place to run baselines.
- **robosuite** ([robosuite.ai](https://robosuite.ai/)) — companion MuJoCo manipulation environments.
- **`tsmatz/imitation-learning-tutorials` on GitHub** — clean from-scratch Python implementations of BC, DAgger, GAIL, AIRL with explanations.

#### Action Chunking Transformer (ACT) & ALOHA

- **Zhao et al. (RSS 2023) — *Learning Fine-Grained Bimanual Manipulation with Low-Cost Hardware*** — the ACT paper. Project page: [tonyzhaozh.github.io/aloha](https://tonyzhaozh.github.io/aloha/).
- **Mobile ALOHA paper** (Fu, Zhao et al.) — extension to wheeled base; full open-source hardware (~$32k DIY) and software.
- **Code:** [`MarkFzp/act-plus-plus`](https://github.com/MarkFzp/act-plus-plus) and [`Interbotix/act_plus_plus`](https://github.com/Interbotix/act_plus_plus) — both contain ACT, Diffusion Policy, and VINN in one repo with sim envs (Transfer Cube, Bimanual Insertion).
- **Cheaper hardware path:** SO-100 / SO-101 arm (~$200) via Hugging Face LeRobot — runs ACT and DP with the same pipeline.

#### Diffusion Policy (the deep dive — most relevant for your research)

- **Chi et al. (RSS 2023, IJRR 2024) — *Diffusion Policy: Visuomotor Policy Learning via Action Diffusion*** — the paper. arXiv 2303.04137.
- **Project page:** [diffusion-policy.cs.columbia.edu](https://diffusion-policy.cs.columbia.edu/) — videos, supplementary, all checkpoints publicly hosted.
- **Official code:** [`real-stanford/diffusion_policy`](https://github.com/real-stanford/diffusion_policy) — comes with **self-contained Colab notebooks** (state-based and vision-based). Easiest possible entry point — **start here.**
- **Step-by-step diffusion math:** Nakkiran et al. (2024) — *Step-by-Step Diffusion: An Elementary Tutorial* — for the underlying DDPM intuition.
- **Curated reading list:** [`mbreuss/diffusion-literature-for-robotics`](https://github.com/mbreuss/diffusion-literature-for-robotics) — 100+ papers on diffusion in robotics, kept current. **Bookmark this.**
- **Yutong Zhang's paper review on Medium** ([Toward Humanoids](https://medium.com/correll-lab/diffusion-policy-applying-diffusion-model-to-policy-learning-8ee16fc056bb)) — clear walkthrough with annotated equations.
- **3D Diffusion Policy (DP3)** and **3D Diffuser Actor** — point-cloud-conditioned variants; often outperform 2D DP on contact-rich tasks (directly relevant to tactile work).
- **Universal Manipulation Interface (UMI)** (Chi et al., 2024) — handheld gripper for in-the-wild data collection that feeds Diffusion Policy. Open hardware + paper.
- **Equivariant Diffusion Policy / SE(3)-DiffusionFields** (TU Darmstadt) — symmetry-aware variants; useful for grasping and 6-DoF tasks.

#### Adversarial / Inverse RL methods (less hot now but important context)

- **Ho & Ermon (2016) — GAIL** (Generative Adversarial Imitation Learning) — paper + clean PyTorch implementations widely available.
- **Fu, Luo & Levine (2018) — AIRL** (Adversarial IRL — recovers reward function).
- **Ziebart et al. (2008) — MaxEnt IRL** — foundational; understand this before reading anything modern.
- **Russell et al. (NIPS 2016) — *Cooperative Inverse Reinforcement Learning*** — assistance-game framing; intellectually important for safety-relevant IL.
- **TU Darmstadt (Jan Peters' IAS group)** ([ias.informatik.tu-darmstadt.de/Research/ImitationLearning](https://www.ias.informatik.tu-darmstadt.de/Research/ImitationLearning)) — surveys and recent work on diffusion-based IL/IRL.

#### Modern foundation IL models (VLAs you should know)

| Model | Lab | Notes |
|---|---|---|
| **π0 / π0.5 / π0-fast** | Physical Intelligence | Flow-matching action head; SOTA generalist; partial open weights via LeRobot |
| **OpenVLA** | Stanford/Berkeley | Fully open VLA (7B), fine-tunable on a single GPU |
| **SmolVLA** | Hugging Face | Small, efficient; runs on edge devices; native to LeRobot |
| **Octo** | Berkeley/Stanford | Pretrained on 800k Open X-Embodiment trajectories; JAX |
| **RT-1, RT-2, RT-X** | Google DeepMind | Original generalist policies; partial release |
| **GR00T N1.5 / N1.6** | NVIDIA | Humanoid-focused; fully open via LeRobot |
| **RDT-1B** | Tsinghua | Diffusion transformer for bimanual manipulation; open |

#### Key datasets & benchmarks (you'll need these)

- **Open X-Embodiment (OXE)** — 1M+ trajectories across 22 robot embodiments. *The* pretraining dataset.
- **DROID** — 76k diverse manipulation episodes from 564 scenes (Stanford-led, 2024).
- **LIBERO** — lifelong learning manipulation benchmark.
- **CALVIN** — long-horizon language-conditioned manipulation.
- **Robomimic datasets** — clean offline learning baselines.
- **LeRobot Hub** ([huggingface.co/lerobot](https://huggingface.co/lerobot)) — community-contributed datasets in standard format; easiest to consume.

#### Where to publish / follow research

- **CoRL (Conference on Robot Learning)** — most relevant venue for IL/manipulation work. **Workshops are very accessible for first papers** — strongly recommend this as your PhD-app submission target.
- **RSS, ICRA, IROS** — broader robotics venues.
- **NeurIPS / ICLR** — for the ML-side framing. Watch for *Workshop on Imitation Learning* and *Foundation Models for Decision Making*.

#### Suggested 6-week IL/DP sprint (with a tactile angle for your portfolio)

1. **Week 1:** Run the official Diffusion Policy Colab notebook end-to-end on Push-T. Read Tedrake Ch. 21 in parallel.
2. **Week 2:** Run ACT in sim (Transfer Cube). Read the original ACT and DP papers carefully. Reproduce reported numbers.
3. **Week 3:** Set up LeRobot with either an SO-101 (~$200) or in MuJoCo. Collect ~50 demos of a simple task.
4. **Week 4:** Train Diffusion Policy on your own demos. Get the full data → train → deploy loop working.
5. **Week 5:** Fine-tune SmolVLA or π0 on the same task; compare against your DP baseline.
6. **Week 6+:** Add a tactile observation modality (Nano17 F/T, or DIGIT/GelSight if available) to the DP input. **This is publishable** — tactile-conditioned diffusion policies remain an open subarea, and you have the slip-detection background to do it well. Aim for a CoRL or ICRA workshop submission. This single project would carry serious weight in your PhD applications.

> **Tactile-IL papers to track on Google Scholar alerts:** TouchInsight, Tactile-VLA, FeelAnyForce, RoboPack, T-DEX, See-Through-Your-Skin, AnySkin, NeuralFeels, MimicTouch.

### 1.4 Computer Vision & 3D

- **Stanford CS231n** — CNN/vision foundations. Free lectures.
- **First Principles of Computer Vision** (Shree Nayar, Columbia) — free YouTube series, brilliant for image formation, sensors (relevant to your EVS work).
- **3D Vision: NeRFs & Gaussian Splatting** — Jonathan Barron's tutorials, NerfStudio docs.
- **Cyrill Stachniss SLAM Course** (Bonn) — YouTube. The gold standard for SLAM.

### 1.5 Foundation models / LLMs background (you'll need this)

- **Stanford CS25 — Transformers United** — YouTube
- **Andrej Karpathy — "Neural Networks: Zero to Hero"** — YouTube. Build GPT from scratch.
- **DeepLearning.AI short courses** (Andrew Ng) — free, 1-hour modules on RAG, agents, function calling.

---

## 2. Firmware / Embedded / Low-Level Electronics

### 2.1 Bare-metal & ARM Cortex-M (the foundation)

- **UT Austin — Embedded Systems** (Jonathan Valvano) on edX — *the* legendary embedded course, TM4C123 board. Free to audit; ~$99 for cert. Two parts (Shape the World + Multi-Threaded Interfacing).
- **University of Colorado Boulder — Embedded Software & Hardware Architecture** Specialization (Coursera) — STM32-based, professional toolchain. ~$49/mo.
- **FastBit Embedded Brain Academy** (Udemy, by Kiran Nayak) — "Mastering Microcontroller" series on STM32. Excellent paid sequence for register-level work, RTOS porting, USB, CAN, Ethernet. ~$15–20 per course on sale.

### 2.2 RTOS

- **FreeRTOS official book + tutorials** ([freertos.org](https://www.freertos.org/)) — free, comprehensive. Run it on STM32.
- **Zephyr Project training** ([zephyrproject.org](https://www.zephyrproject.org/)) — Linux Foundation has a paid LFD419 cert (~$299). Zephyr is gaining hard in industrial/robotics; worth knowing.
- **Quantum Leaps / Miro Samek YouTube** ("Modern Embedded Systems Programming") — free, deep, by the author of QP/state machine framework.

### 2.3 Real-Time Linux (very relevant given your EtherCAT pipeline work)

- **OSADL Real-Time Linux training** — paid but the authoritative source.
- **PREEMPT_RT documentation** + Reghenzani et al. survey paper.
- **Xenomai docs** — for hard real-time on top of Linux.

### 2.4 Motor control, power electronics, BLDC/FOC (anticogging adjacent)

- **TI Motor Control Engineer training** (free, ti.com/motor-control)
- **Microchip Motor Control University** (free)
- **SimpleFOC** ([simplefoc.com](https://simplefoc.com/)) — open hardware + Arduino-friendly FOC. Build a custom BLDC driver as a portfolio piece.
- **Texas Instruments InstaSPIN-FOC** application notes.

### 2.5 PCB design (firmware engineers are 10x more valuable when they can spin a board)

- **KiCad EDA** — free, industry-respected. Phil's Lab YouTube channel for STM32 + KiCad.
- **Altium Academy** — free official tutorials if you have Altium access.
- **Robert Feranec YouTube** — high-speed design, signal integrity.

### 2.6 FPGA (high leverage for sensor fusion, EVS, real-time perception)

- **HDLBits** ([hdlbits.01xz.net](https://hdlbits.01xz.net/)) — free Verilog problem set, like LeetCode for FPGAs.
- **Adam Taylor — MicroZed Chronicles** — Hackster series, Xilinx/AMD focus.
- **Nand2Tetris** (Coursera) — build a CPU from logic gates. Foundational.
- **AMD/Xilinx Vitis HLS training** — free, certificate available.

---

## 3. Control Theory (the bedrock — and the bridge between firmware and AI)

> Control theory is what separates someone who can train a policy from someone who can deploy it on a real robot that doesn't break. Every serious robotics PhD program will probe your controls knowledge in interviews. Modern policy learning (DP, VLAs) does *not* replace classical control — it sits on top of it. Impedance controllers, MPC, and Kalman filters are still the inner loops underneath every "end-to-end" demo.

### 3.1 The conceptual map (what to actually learn, in order)

**Tier 1 — Classical / linear (must-have):**
- Continuous and discrete-time signals, Laplace and Z-transforms
- Transfer functions, block diagrams, frequency response (Bode, Nyquist)
- Stability margins (gain margin, phase margin), root locus
- **PID** — tuning methods (Ziegler-Nichols, relay autotune, Tyreus-Luyben), anti-windup, derivative filtering, integrator decay
- State-space representation, controllability, observability
- Pole placement, Ackermann's formula
- Lyapunov stability (direct method, linearization)

**Tier 2 — Optimal & estimation (the working set for robotics):**
- **LQR** (continuous and discrete) — Riccati equation, infinite-horizon, finite-horizon, time-varying LQR
- **LQG** = LQR + Kalman filter (separation principle)
- **Kalman filter** — derivation, prediction/update, tuning Q and R
- **EKF** and **UKF** — when each fails; sigma-point intuition
- **Particle filter** — for non-Gaussian / multimodal estimation
- **Complementary filter** — the practical IMU workhorse
- Sensor fusion patterns (loose vs. tight coupling)

**Tier 3 — Predictive & trajectory optimization (the modern core):**
- **MPC / receding-horizon control** — convex MPC, QP formulation, constraint handling, terminal cost/set, recursive feasibility, stability guarantees
- **Nonlinear MPC (NMPC)** — direct multiple shooting, direct collocation
- **iLQR / DDP** (Differential Dynamic Programming) — the foundation of most modern trajectory optimizers and the backbone of MuJoCo MPC
- **Sequential Quadratic Programming (SQP)** for trajectory optimization
- **Convex optimization** — KKT conditions, duality, why a QP solves in microseconds and an NLP doesn't
- Warm-starting, real-time iteration scheme

**Tier 4 — Nonlinear & robust (for serious robot work):**
- Feedback linearization (input-output and full-state)
- Sliding mode control
- Backstepping
- Lyapunov-based controller synthesis
- H-infinity / robust control basics
- Adaptive control (MRAC, self-tuning regulators)

**Tier 5 — Robotics-specific control (your wheelhouse):**
- Inverse dynamics / **computed torque control**
- **Operational space control** (Khatib formulation) — the "task-space" mental model
- **Impedance and admittance control** (Hogan) — *the* paradigm for contact-rich tactile manipulation; **directly relevant to your slip-detection and end-effector work**
- **Hybrid position/force control** (Raibert-Craig)
- Whole-body control via QP (used in every modern humanoid)
- Centroidal dynamics, ZMP, capture point (legged systems)

**Tier 6 — Modern / learning-based control (where research lives):**
- **Control Barrier Functions (CBFs)** — safe RL and safety filters
- **Differentiable simulation / differentiable MPC** (Amos et al., MuJoCo XLA, Brax)
- **Koopman operators** for data-driven linearization
- **Sum-of-squares (SOS)** programming for nonlinear analysis
- Neural ODEs and learned dynamics models for MPC (TD-MPC, Dreamer-style world models)
- Policy distillation: train a big NMPC offline, distill to a fast policy

### 3.2 Best courses (free unless noted)

- **Steve Brunton — *Control Bootcamp* on YouTube** — the single best intuition-builder for classical + modern control. Watch this *first*. Pairs with his *Data-Driven Science and Engineering* book (also free PDF) for Koopman, DMD, SINDy.
- **Brian Douglas — *Control Systems Lectures* on YouTube** — the gold standard for classical control intuition. Also see the MathWorks-produced *Understanding PID*, *Understanding Kalman Filters*, and *Understanding Model Predictive Control* video series (he hosts most of them).
- **Russ Tedrake — MIT 6.832 *Underactuated Robotics*** ([underactuated.mit.edu](https://underactuated.mit.edu/)) — *the* free book + lecture series. Covers LQR, trajectory optimization, sums of squares, and bridges into policy learning. **Required reading for any manipulation PhD.**
- **Russ Tedrake — MIT 6.4210 *Robotic Manipulation*** ([manipulation.csail.mit.edu](https://manipulation.csail.mit.edu/)) — covers task-space control, force/impedance, grasp planning. Uses Drake.
- **Zac Manchester — CMU 16-745 *Optimal Control and Reinforcement Learning*** — fully open-source: [GitHub org](https://github.com/Optimal-Control-16-745), lectures on YouTube. **The single best free MPC/iLQR/trajectory-optimization course on the internet.** Strong Julia code component.
- **Stephen Boyd — Stanford EE263 *Linear Dynamical Systems*** — the rigorous linear-algebra-first treatment of state space, controllability, observability. Free lectures.
- **Stephen Boyd — Stanford EE364A/B *Convex Optimization*** — required prerequisite for understanding MPC and trajectory optimization. Free lectures + free book PDF.
- **Jean-Jacques Slotine — MIT *Applied Nonlinear Control*** — Lyapunov, sliding mode, adaptive control. Lectures available; book is the reference.
- **Frank Park & Kevin Lynch — *Modern Robotics* (Northwestern)** — Coursera specialization + free book; the standard for kinematics/dynamics that controls is built on top of.
- **Sebastian Trimpe / Melanie Zeilinger — RWTH/ETH lecture videos on YouTube** for safe learning-based control and learning-augmented MPC (current research frontier).
- **Pieter Abbeel — Foundations of Deep RL (6-lecture series, YouTube)** — bridges control and RL conceptually.

### 3.3 Books (the canon)

- **Åström & Murray — *Feedback Systems: An Introduction for Scientists and Engineers*** — free PDF online. The best modern intro textbook.
- **Khalil — *Nonlinear Systems*** — *the* graduate-level reference. Buy this; it lasts a career.
- **Slotine & Li — *Applied Nonlinear Control*** — more practitioner-oriented than Khalil, easier entry.
- **Anderson & Moore — *Optimal Control: Linear Quadratic Methods*** — the LQR/LQG bible. Dover paperback, cheap.
- **Rawlings, Mayne & Diehl — *Model Predictive Control: Theory, Computation, and Design*** — free PDF on Rawlings's site. *The* MPC reference.
- **Boyd & Vandenberghe — *Convex Optimization*** — free PDF. Required.
- **Tedrake — *Underactuated Robotics*** — free online textbook.
- **Spong, Hutchinson & Vidyasagar — *Robot Modeling and Control*** — classic, balances modeling and control.
- **Siciliano, Sciavicco, Villani & Oriolo — *Robotics: Modelling, Planning and Control*** — comprehensive European-school reference.
- **Murray, Li & Sastry — *A Mathematical Introduction to Robotic Manipulation*** — free PDF; Lie-group treatment of manipulation.
- **Thrun, Burgard & Fox — *Probabilistic Robotics*** — for Kalman/EKF/particle filtering done right.
- **Simon — *Optimal State Estimation*** — best Kalman-filter book if you want a single reference.

### 3.4 Software / solvers (what working roboticists actually use)

| Tool | What it does | When to use |
|---|---|---|
| **Drake** ([drake.mit.edu](https://drake.mit.edu/)) | Tedrake's full robotics toolbox: dynamics, trajectory opt, simulation, MPC | Manipulation research, anything in his ecosystem |
| **CasADi** ([casadi.org](https://web.casadi.org/)) | Symbolic framework for nonlinear optimization | NMPC prototyping in Python/MATLAB |
| **acados** ([github.com/acados/acados](https://github.com/acados/acados)) | Real-time embedded NMPC; auto-generates C code | Production NMPC on real robots/MCUs |
| **OCS2** | Optimal Control for Switched Systems (ETH) | Legged robots, hybrid systems |
| **Crocoddyl** | DDP/iLQR for legged robots (LAAS-CNRS) | Trajectory optimization for humanoids/quadrupeds |
| **OSQP** ([osqp.org](https://osqp.org/)) | Operator splitting QP solver, embedded-friendly | Fast convex MPC, the de facto QP solver |
| **TinyMPC** ([tinympc.org](https://tinympc.org/)) | MPC for resource-constrained MCUs (Zac Manchester's group) | Running MPC on STM32/Cortex-M — relevant to your firmware side |
| **CVXPY / CVXGEN** | Disciplined convex programming | Prototyping convex problems quickly |
| **MuJoCo MPC (mjpc)** | DeepMind's interactive predictive control on MuJoCo | Real-time iLQR/predictive sampling for sim |
| **Pinocchio** | Fast rigid-body dynamics with analytical derivatives | Backbone for Crocoddyl, modern whole-body control |
| **Stable Baselines3 / RSL-RL / SKRL** | RL libraries | When the problem is bigger than what classical control handles |
| **MATLAB Control / Robotics / MPC Toolboxes** | Industry standard for system ID, control design | Job interviews still test Simulink fluency |
| **Julia: ControlSystems.jl, TrajectoryOptimization.jl** | Modern numerical control stack | Manchester's group uses these heavily |

### 3.5 Concrete project ideas to lock in the concepts

1. **Cart-pole swing-up** — classical baseline. Implement LQR (linearized), then iLQR (full nonlinear). Compare. Then train a PPO policy and compare again.
2. **Quadrotor hover and trajectory tracking** — implement linear MPC with OSQP. Add wind disturbance.
3. **Kalman filter for IMU + encoder fusion** — on a real STM32 + MPU6050. This connects your firmware track to controls.
4. **Impedance-controlled end-effector for slip mitigation** — *directly applicable to your Meta tactile work and your PhD pitch*. Implement a Cartesian impedance controller, modulate stiffness based on tactile slip signal. This is publishable.
5. **TinyMPC on Cortex-M** — port a real MPC controller to an STM32, profile it, write up the timing analysis. Combines firmware + controls; great portfolio piece.
6. **Differentiable MPC layer in a learned policy** — implement Amos & Kolter's differentiable LQR/MPC. This is the modern research direction.

### 3.6 Connection back to your other tracks

- **→ Firmware:** TinyMPC, embedded Kalman filters, real-time scheduling for control loops, FOC for BLDCs are all controls problems.
- **→ Imitation Learning:** ACT and Diffusion Policy output *action sequences* that get tracked by a low-level controller — usually a joint PD or impedance controller. Knowing the inner loop is what makes outer-loop policies actually work on hardware.
- **→ PhD applications:** "Learning-augmented control" / "safe RL" / "differentiable control" are some of the hottest areas at MIT (Tedrake, Slotine), CMU (Manchester, Shi), Stanford (Pavone, Sadigh), Berkeley (Tomlin, Sreenath), UPenn (Kumar, Pappas), Caltech (Ames, Yue). Showing controls fluency *and* learning fluency is a stronger profile than either alone.

---

## 4. Robotics Frameworks & Simulation

- **NVIDIA Isaac Sim + Isaac Lab** — covered above; the dominant sim platform.
- **MuJoCo** — DeepMind's physics engine, now free. Used by every top RL lab. Official tutorials are good.
- **The Construct** ([theconstructsim.com](https://www.theconstructsim.com/)) — best ROS 2 hands-on platform. Browser-based. Paid (~$30/mo) but worth it for a focused 1–2 month sprint.
- **Open Robotics ROS 2 tutorials** — free, official.
- **PyBullet & Genesis** — lighter-weight sim alternatives.

---

## 5. Math & CS Foundations (refresh as needed)

- **MIT 18.06 Linear Algebra** (Gilbert Strang) — free OCW.
- **3Blue1Brown — Essence of Linear Algebra / Calculus** — best intuition videos ever made. Free.
- **Stephen Boyd — Convex Optimization** (Stanford EE364A) — free lectures + book PDF. Foundational for control, MPC, planning.
- **Probabilistic Robotics** (Thrun, Burgard, Fox) — the textbook for Bayes filters, SLAM, particle filters.
- **Modern Robotics: Mechanics, Planning, and Control** (Lynch & Park, Northwestern) — free book + Coursera specialization. *Standard* text for kinematics/dynamics.

---

## 6. High-Value Certifications (in rough order of ROI)

| Cert | Provider | Cost | Worth it for |
|---|---|---|---|
| **NVIDIA-Certified Associate: Generative AI Multimodal / LLMs** | NVIDIA | ~$135 | Resume signal; aligns with VLA work |
| **NVIDIA OpenUSD Development Professional** | NVIDIA | ~$400 | Sim/digital twin work, Omniverse ecosystem |
| **NVIDIA Jetson AI Specialist / Ambassador** | NVIDIA DLI | Free–$90 | Edge robotics, deployable AI |
| **AWS Certified Machine Learning – Specialty** | AWS | $300 | Cloud ML pipelines; broad value |
| **Hugging Face Deep RL Course Certificate** | HF | Free | Hands-on RL portfolio proof |
| **Coursera — Modern Robotics Specialization** | Northwestern | ~$49/mo | Foundation cert, well-respected |
| **Coursera — Self-Driving Cars Specialization** | U. Toronto | ~$49/mo | If autonomy interests you |
| **edX — Embedded Systems (UT Austin)** | UT Austin | $99 each | Legitimate embedded credential |
| **Linux Foundation — Zephyr RTOS (LFD419)** | LF | ~$299 | Real industrial RTOS skill |
| **Certified ROS Developer** | The Construct | included w/ subscription | ROS 2 proficiency proof |
| **MathWorks Certified MATLAB/Simulink Associate** | MathWorks | $150–250 | Control, model-based design roles |
| **Barr Group — Embedded Software Boot Camp / CESP** | Barr | $$$ (employer-paid usually) | Senior firmware engineer signal |
| **AMD/Xilinx Vitis HLS / Vivado certs** | AMD | varies | FPGA roles |

> For PhD applications, **certifications matter much less than projects + papers**. For industry roles (Zoox, Apptronik, Figure, 1X, Boston Dynamics, Tesla Optimus, etc.), the NVIDIA certs and a strong GitHub matter far more than the rest.

---

## 7. High-Demand Specialized Tracks (pick 1–2)

### Humanoid robotics & whole-body control
- Isaac Lab humanoid environments (G1, H1, GR1)
- MIT 6.832 Underactuated Robotics (Russ Tedrake) — free book + lectures, *the* reference for legged/dynamic systems.
- Hwangbo et al. ANYmal papers; Marco Hutter group work.

### Manipulation & dexterous hands (your home turf)
- Russ Tedrake — **Robotic Manipulation** (MIT 6.4210) — free book + Drake software stack. Best modern resource.
- Stanford CS336 — Foundations of Robotic Learning.
- Track: TRI, Toyota Research, Google DeepMind robotics, Berkeley AUTOLab, MIT IRG.

### Event-based vision (matches your current research interest)
- **Davide Scaramuzza's group (UZH-RPG)** — papers, datasets (DSEC, MVSEC), and open-source code (rpg_dvs_ros, event_camera_py).
- Prophesee Metavision SDK tutorials.
- ESIM event simulator.

### Autonomous driving stack
- U. Toronto Self-Driving Cars Specialization (Coursera).
- MIT 6.S094 Deep Learning for Self-Driving Cars.
- CARLA + Apollo open-source studies.

### Drone autonomy
- UPenn Robotics Specialization (Coursera, Vijay Kumar) — aerial control modules.
- ETH Zurich + UZH-RPG drone papers.

---

## 8. Books Worth Buying (or pirating from a library)

- **Making Embedded Systems** — Elecia White (also her *Embedded.fm* podcast)
- **Real-Time Concepts for Embedded Systems** — Qing Li
- **A Mathematical Introduction to Robotic Manipulation** — Murray, Li, Sastry (free PDF)
- **Modern Robotics** — Lynch & Park (free PDF)
- **Probabilistic Robotics** — Thrun, Burgard, Fox
- **Deep Learning** — Goodfellow et al. (free online)
- **Mathematics for Machine Learning** — Deisenroth et al. (free PDF)
- **Robot Modeling and Control** — Spong, Hutchinson, Vidyasagar

---

## 9. Stay Current (the meta-skill)

- **Conferences to follow papers from:** ICRA, IROS, RSS, CoRL, NeurIPS, ICLR, RA-L (journal).
- **Newsletters:** *The Batch* (deeplearning.ai), *Import AI* (Jack Clark), *Robotics Tomorrow*, NVIDIA Robotics Newsletter.
- **YouTube:** Two Minute Papers, Yannic Kilcher, Lex Fridman (interviews), NVIDIA Developer.
- **Discord/Slack:** LeRobot, Isaac Sim, Zephyr Project, ROS Discourse.
- **arXiv-sanity-lite + Semantic Scholar alerts** — set up keyword alerts for "tactile sensing", "slip detection", "VLA", "diffusion policy".
- **Twitter/X follows:** Sergey Levine, Chelsea Finn, Russ Tedrake, Pieter Abbeel, Jim Fan, Lerrel Pinto, Vincent Vanhoucke, Karol Hausman.

---

## Suggested 6-month sequence (given your starting point)

1. **Month 1–2:** Run LeRobot end-to-end with SO-101 arm. Train a Diffusion Policy on a real task. Push to GitHub.
2. **Month 2–3:** Berkeley CS285 in parallel; reproduce one Isaac Lab manipulation environment with your own reward.
3. **Month 3–4:** Pick one VLA (π0 or GR00T N1.5), fine-tune on a tactile-augmented task. Write a short report — this is PhD-app material.
4. **Month 4–5:** Sharpen one weak area (FPGA for EVS pipelines, or motor control / FOC for end-effector design). Build a small board.
5. **Month 5–6:** NVIDIA OpenUSD or Jetson AI Specialist cert. Submit a workshop paper (CoRL workshops, ICRA workshops are accessible).

---

*Last updated: April 2026. Resource availability changes — verify current pricing/links before committing.*
