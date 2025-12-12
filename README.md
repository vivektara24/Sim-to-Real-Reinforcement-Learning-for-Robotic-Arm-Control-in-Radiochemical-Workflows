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

Nuclear medicine is the most rapidly expanding area of radionuclide use, driving an increased demand for efficient, reliable, and scalable methods for radiopharmecutical production. Automation of chemical workflows offers a powerful means to meet these demands and accelerate discoveries. Rapid advances in artificial intelligence (AI) have opened new avenues to for optimizing these workflows, enabling data-driven decision-making and shortening development cycles.

Robots were originally designed to assist or replace humans by performing repetitive, hazardous, or physically demanding, particularly in enviroments that impose limitations on human operations. A central goal of robotics research is to endow machines with human-like motor capabilites that enable smooth, adaptive, and natural interactions with their enviroment. Reinforcment learning (RL) provides a powerful framework for achieving this goal by enabling a robot to autonomously learn optimal behavior through trial-and-error interactions with its enviroment, driven by reward functions that encode task objectives.

Simulators offer virtually unlimited tiral-and-error chances to perform the exploartion neccesary for RL. However, the successful transfer of policies learned in simulation to real-world robotic systems critically depends on accurate modeling of both robots and the enviroments they interact with. Manufacturer supplied robot models offer a baseline, but often require significant tuning to be ready for sim-to-real. 

This study investigates the use of deep rienforcment learning to train a robotic manipulator to perform a multi-step radiochemical seperation workflow. As an initial step, the Soft Actor-Critic (SAC) algorithm is implemented to learn a policy for robust pick-and-place operations within NVIDIA's Isaac Lab simulation framework. To mitigate the sim-to-real gap, we extend prior work on parallized autotuning by integrating Bayesian optimization to directly minimze tracking error. Additionally, domain randomization is employed to improve policy robustness to agianst the inherent sim-to-real gap.

## Methods

#### _Simulation Enviroment_

Training reinforcement learning policies on physical robots is impractible due to time, wear, and saftey constraints. Isaac Lab offers a simulation framework 

The direct worklow in Isaac Lab is desinged to expose fine-grained control over simulation and learning pipleines

#### _Autotuned Robot Modeling_

To reduce the sim-to-real gap prior to learning a policy, an automated physics-parameter tuning framework based on Bayesian optimzation was developed. The Optuna hyperparameter optimzation library was employted to systematically calibrate key low-level physical parameters of the simulated Lite6 robot model. These parameters included joint friction coefficients, joint damping constants, and joint aramture values, which influence robot dynamics but were initially poorly speciifed by the manufacturer. Joint stiffness was set to zero to enable joint velocity control, consistent with recommendations in the Isaac Lab documentation.

The autotuning produceure begins by collecting joint-level trajectores including positions, velocities, and torques, from real-world motion dmeostrations performed on the physical Lite6 robot arm. For each optimization trial, the same control policy is executed in simulation using a candidate set of physics parameters. The mean square error (MSE) between real and simulated joint trajectories is computed and used as the optimization objective. Optuna iterativley proposes parameter updates to minimize this error, yielding a set of physics paramters that best reproduce the observed real-world behavior.

This approach enables fully automated identification of realistic simulation paramters, imporving correspondance between simulation and physical execution reducing the real-to-sim gap.

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

This study solves the problem of a Pick-and-Place using a the Soft Actor-Critic deep reinforcment learning algorithm.

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

## Acknowledgements

**This material is based upon work supported by the U.S Department of Energy, Office of Science, Isotope Program, under Award Number DE-SC0022550 through the Horizon-broadening Isotope Production Pipeline Opportunities (HIPPO) program.**

I would like to thank **Andrew Blades** for his contributions to this project and **Dr. Jayheon Do**, **Dr. Dohyun Kim**, and **Dr. Cathy Cutler** for the opportunity to contribute to this work.

## References
