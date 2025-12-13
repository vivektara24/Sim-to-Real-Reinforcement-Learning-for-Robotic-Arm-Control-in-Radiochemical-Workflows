# Sim-to-Real Reinforcement Learning for Robotic Arm Control in Radiochemical Workflows

<p align="center">
  <img src="images/bnl_logo.png" alt="HIPPO Logo" height="100"/>
  &nbsp;&nbsp;&nbsp;&nbsp;
  <img src="images/hippo.png" alt="UW Medical Physics Logo" height="100"/>
</p>

## Table of Contents
1. [Introduction](#introduction)
2. [Methods](#methods)
3. [Results](#results)
4. [Acknowledgements](#acknowledgements)
5. [References](#references)

## Introduction

Nuclear medicine is the most rapidly expanding area of radionuclide use, driving an increased demand for efficient, reliable, and scalable methods for radiochemical seperations and radiopharmecutical production. Automation of chemical workflows offers a powerful means to meet these demands and accelerate discoveries. Rapid advances in artificial intelligence (AI) have opened new avenues to for optimizing these workflows, enabling data-driven decision-making and shortening development cycles.

Robots were originally designed to assist or replace humans by performing repetitive, hazardous, or physically demanding tasks, particularly in enviroments that impose limitations on human operations. A central goal of robotics research is to endow machines with human-like motor capabilites that enable smooth, adaptive, and natural interactions with their enviroment. Reinforcment learning (RL) provides a powerful framework for achieving this goal by enabling a robot to autonomously learn optimal behavior through trial-and-error interactions with its enviroment, driven by reward functions that encode task objectives.

Simulators offer virtually unlimited trial-and-error chances to perform the exploartion neccesary for RL. However, the successful transfer of policies learned in simulation to real-world robotic systems critically depends on accurate modeling of both robots and the enviroments they interact with. Manufacturer supplied robot models offer a baseline, but often require significant tuning to be ready for sim-to-real. 

This study investigates the use of deep rienforcment learning to train a robotic manipulator to perform a multi-step radiochemical seperation workflow. As an initial step, the Soft Actor-Critic (SAC) algorithm is implemented to learn a policy for robust pick-and-place operations within NVIDIA's Isaac Lab simulation framework. To mitigate the sim-to-real gap, we extend prior work on parallized autotuning by integrating Bayesian optimization to directly minimze tracking error. Additionally, domain randomization is employed to improve policy robustness to agianst the inherent sim-to-real gap.

## Methods

#### _Isaac Lab Simulation Enviroment_

Training reinforcement learning policies on physical robots is impractible due to time, wear, and saftey constraints. To address these limitations, this work leverages Isaac Lab, an open source simulation framework built on NVIDIA Isaac Sim. Isaac Lab combines RTX rendering for photorealistic, scalable visuals with PhysX for high-fidelity physics simulation. It uses Universal Scene Description (USD) as the core data layer for structed world authoring. Together, these capabilities scale efficiently across multi-GPU and multi-node setups. At it's core, Isaac Lab designs a manager-based API that organizes enviorment design into reusable and composable components, allowing consistent workflows across diverse research projects. Key features include integration of non-linear actuator models, multi-frequency custom sensor simulation, interfaces for low-level controllers, and tools for procedural enviorment genration and domain randomization.

_USD for Robotics_

Open Universal Scene Description (OpenUSD) is an open-source format for robust and scalable authoring of complex 3D scenes composed of numerous elements. It provides a comprehensive set of tools and Python/C++ APIs to define, organize, and edit 3D data. USD represents a 3D scene as a hierarcal scene graph, with data arranged in namespaces of primitives. This scene graph is parsed into PhysX, which allocated GPU tensors representing the interal simulation state. OmniPhysics exposes these states through teh View APIs which allow read or write access to subsets of objects in the scene while simplifiying data mangenemtn and improving usability.

A key development from the Alliance for OpenUSD (AOUSD) organization is the USDPhysics Schema, which extends OpenUSD with a standardized way to describe physical properties such as rigid bodies, collisions, joints, and materials and interpret scenes consistently across different engines and workflows. By providing a common representation representation for physics simulation, it enables robotics and simulation tools to share and interpret scenes consistently across different engines and workflows. However, OpenUSD also allows straightforward extention with engine-specific schemas; the PhysxSchema provides paratmers used in NVIDIA PhysX.

Beyond physics, OpenUSD provides complementary schemas that enrich how scenes can be represented and eexhanged across domains. For example the semantics schema annotates prims with categorical lables, enabling tasks in perception and learning. Camera prims allow the description of virtual sensors directly with a scene, ensuring that viewpoints and sensor models are preserved consistently across tools. Similarly Material schemas caputer surface properties in a standardized way, briding the needs fo both rendering and physics. These schemas unify geometery, dynamics, semantics, sensing, and apperance within a single scene description. The integrated representation overcomes the limitations of existitng robotics formats such as MJCF (physics-focused, limits scene richness) and URDF (kinematics/dynamics with Gazebo-specific tags for sensors).

Isaac Lab provided USD converters for widley used formats including URDF, MJCF, and meshes (e.g., OBJ, DAE).

_Physics Simulation_

The NVIDIA PhysX SDK is an open-source, multi-physics simulation engine desinged to meet the demainding needs for robotics and industrial applications. It supports a wide range of simulation types, from rigid and articulated bodies to deformable bodies such as cloth, fluids, and soft bodies. These simulation objects can interact with each other throug htwo-way coupling between specialized solvers, enabling efficient and accurate mixed-physics interactions.

PhysX can run on both CPU and GPU, providing flexibility for different simulation senarios. PhysX's Direct-GPU API provides direct read and write access to simulation state and control data in GPU memory. The resulting CUDA tensors can then be processed efficiently using user-defined GPU kernels for downstream applications, such as computing observations for robotics learning workflows. This end-to-end GPU pipeline eliminates the performance bottlenecks from CPU-GPU data transfers seen in traditional simulators.

NVIDIA Omniverse Physics (OmniPhysics) serves as the integration layer for the PhysX simulation engine within NVIDIA Omniverse. It extends the OpenUSD framework by introducing user-defined data types under PhysxSchema, which enables representation and control of physics simulations. OmniPhysics parses these attributes from USD, maps them into PhysX, runs the simulation, and writes the simulation output back to USD. It also supports state updates by monitoring changes in USD and updating the active simulation accordingly.

Where high-performance and large-scale simulation scenes are required, the USD read-write cycle during simulation is often a performance bottleneck and is therefore bypassed. Instead, the simulation data is accessed through OmniPhysics Tensor APIs, which internally rely on PhysX Direct GPU APIs. For workflows that require rendering, such as vision-in-the-loop training, OmniPhysics also maintains efficient synchronization between the simulation state and the renderer.

_Assets_

Assets correspond to any physics-enabled object that can be added to the simulation. These include rigid objects, articulations, and deformable objects. Each asset provides a high-level interface that wraps around low-level USD and OmniPhysics View APIs. Although the underlying TensorAPI views expose direct read and write access to simulation data, Isaac Lab introduces the additional abstraction layer to ensure a unified and reliable way of interacting with the simulation.

The asset interface consolidates attributes that are not directly managed by TensorAPIs but are essential for robotics. 

Additionally the interface employs a lazy-update mechanism for simulation state retrieval. With the lazy-update approach, data for each attribute is fetched only upon its first access after a simulation step, and subsequent acceses within the same step use the cached values.

_Actuators_

In Isaac Lab, actuators are the interface between desired joint actions and articulation motion.

#### Robot Platform

We use the UFACTORY Lite 6, a 6-degree-of-freedom (DoF) lightweight robotic arm with fully actuated joints, a 440mm reach, 600g payuload, capacity, repeatability of ±0.5mm, and an ISO 9409-1 compliant parallel gripper.


#### _Autotuned Robot Modeling_

To reduce the sim-to-real gap prior to learning a policy, an automated physics-parameter tuning framework based on Bayesian optimzation was developed. The Optuna hyperparameter optimzation library was employted to systematically calibrate key low-level physical parameters of the simulated Lite6 robot model. These parameters included joint friction coefficients, joint damping constants, and joint aramture values, which influence robot dynamics but were initially poorly speciifed by the manufacturer. Joint stiffness was set to zero to enable joint velocity control, consistent with recommendations in the Isaac Lab documentation. 

The autotuning produceure begins by collecting joint-level trajectores including positions, velocities, and torques, from real-world motion dmeostrations performed on the physical Lite6 robot arm. For each optimization trial, the same control policy is executed in simulation using a candidate set of physics parameters. The mean square error (MSE) between real and simulated joint trajectories is computed and used as the optimization objective. Optuna iterativley proposes parameter updates to minimize this error, yielding a set of physics paramters that best reproduce the observed real-world behavior.

This approach enables fully automated identification of realistic simulation paramters, improving correspondance between simulation and physical execution reducing the real-to-sim gap.

#### Domain Randomization

To improve robustness and further facilitate sim-to-real transfer, domain randomization was applied throughout training. Variability was introduced across object properties, sensory observations, and control actions to expose the the policy to a diverse range of operating conditions. This strategy encourages the learned policy to generalize beyond a single nominal simulation configuration. The specified randomized parameters and their associated noise distributions are summarized in Table 3.

<table align="center">
  <thead>
    <tr>
      <th>Domain Parameter</th>
      <th>Applied Noise Distribution</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Object Initial Position</td><td>+𝒰(-0.02cm, 0.02cm)</td></tr>
    <tr><td>Object Position Observation Noise</td><td>+𝒰(0.00cm, 0.02cm)</td></tr>
    <tr><td>Joint Observation Noise</td><td>+𝒰(0.00°, 5.73°)</td</tr>
    <tr><td>Action Noise</td><td>+𝒰(0.00°, 5.73°)</td><td></tr>
  </tbody>
</table>


#### _Problem Statement_

This study solves the problem of a Pick-and-Place task using a deep reinforcment learning algorithm. Pick-and-Place is a high-level robot manipulation task, which involves the robot manipulator picking up a specific object and bringing it to a target location. The task is decomposed into a sequence of subtasks, with the reward function transitioning between stages upon completion of each preceding subtask. The enviroment is defined without obstacles. 

The Lite 6 robot model used in this work, lacks actuated finger joints, preventing the simulation of a full grasp-and-release cycle. As a result task success is defined by successful completion of the grasping subtask, operationalized as positioning the object within the gripper's effective grasp radius.

Isaac Lab's Seattle Lab Table serves as the primary workspace for all manipulation tasks. The manipulation object is a cube, modeled as a rigid body with a dimension of 27.8mm, mass of 600g, and coefficinet of static friction 1.0.The enviroment is defined without obstacles. At the start of each episode, both the position and orientation of the object are randomized to promote policy robustness.

#### _Proposed Solution_




## Results

## Results

The proposed autotuning approach resulted in a **31.4% reduction in total joint position RMSE**, with improvements observed across all six joints. Joint-wise RMSE values before and after tuning are summarized in Table 1.


<table align="center">
  <thead>
    <tr>
      <th>Joint</th>
      <th>Initial RMSE (deg)</th>
      <th>Tuned RMSE (deg)</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Joint 1</td><td>17.33</td><td>11.05</td></tr>
    <tr><td>Joint 2</td><td>9.71</td><td>8.78</td></tr>
    <tr><td>Joint 3</td><td>7.83</td><td>4.40</td></tr>
    <tr><td>Joint 4</td><td>4.76</td><td>4.72</td></tr>
    <tr><td>Joint 5</td><td>9.03</td><td>4.99</td></tr>
    <tr><td>Joint 6</td><td>4.50</td><td>2.50</td></tr>
    <tr><td>Total</td><td>53.16</td><td>36.45</td></tr>
  </tbody>
</table>

_**Table 1**. Joint position RMSE (degrees) pre- and post- tuning for each of the Lite6 joints._

Performance across four independent trials is summarized in Table 2. Each trial reports the number of successful outcomes out of ten attempts, where a succesfull outcome is defined as completion of the gripping subtask, along with the mean success rate across all trials.

<table align="center">
  <thead>
    <tr>
      <th>Trial 1</th>
      <th>Trial 2</th>
      <th>Trial 3</th>
      <th>Trial 4</th>
      <th>Average</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>8/10</td><td>10/10</td><td>9/10</td><td>8/10</td><td>8.75/10</td></tr>
  </tbody>
</table>

_**Table 2**. Grasping success rate across four independent trials._

Table 1. Success rate across four independent trials, reported as successful outcomes out of ten attempts per trial. The average success rate across all trials is also shown.

## Acknowledgements

**This material is based upon work supported by the U.S Department of Energy, Office of Science, Isotope Program, under Award Number DE-SC0022550 through the Horizon-broadening Isotope Production Pipeline Opportunities (HIPPO) program.**

I would like to thank **Andrew Blades** for his contributions to this project and **Dr. Jayheon Do**, **Dr. Dohyun Kim**, and Dr. Cathy Cutler for the opportunity to contribute to this work.

## References
