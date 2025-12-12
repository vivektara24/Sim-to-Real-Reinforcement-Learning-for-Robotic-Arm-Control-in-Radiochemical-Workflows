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

Training reinforcement learning policies on physical robots is impractible due to time, wear, and saftey constraints. Traditional CPU-based simulators often struggle to meet the compuational demands of high fidelity physics, complex sensor models, and large-sclae parallelization. Isaac Lab offers a .

The direct worklow in Isaac Lab is desinged to expose fine-grained control over simulation and learning pipleines

#### _Real-to-Sim Modeling_

Two strategies are employed to address the real-to-sim gap: A Bayseian Optimzation based robot autotunign module and domain randomizatoion.

<table align="center">
  <thead>
    <tr>
      <th>Domain Parameter</th>
      <th>Applied Noise Distribution(deg)</th>
    </tr>
  </thead>
  <tbody>
    <tr><td>Object Initial Position</td><td>17.33</td></tr>
    <tr><td>Object Position Observation Noise</td><td>9.71</td></tr>
    <tr><td>Joint Observation Noise</td><td>7.83</td</tr>
    <tr><td>Action Noise</td><td>4.76</td><td></tr>
  </tbody>
</table>


#### _Problem Statement_

This study solves the problem of Pick-and-Placed using the Soft Actor-Critic Deep Rienforcment Learning algorithm.

Pick and place is one of the high-level robot manipulation tasks, which involves the robot manipulator picking up a specified object and brining it to a target location. We use the UFACTORY Lite6, a lightwieght 6-axis robot with a reach of 440mm, a 600g payload capaocity, and 0.5mm repeatability. We equipp the Lite 6 Gripper, a two-finger gripper featuring a 16mm stroke range and maximum payload < 1kg.


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
