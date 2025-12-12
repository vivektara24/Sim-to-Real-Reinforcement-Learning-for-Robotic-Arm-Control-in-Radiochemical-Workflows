# Sim-to-Real Reinforcement Learning for Robotic Arm Control in Radiochemical Workflows

## Introduction

Automation of chemical workflows has the potential to accelerate discoveries, but this requires.

Robots were originally designed to assist or replace humans by performing repetitive and/or dangerous tasks which humans usually prefer not to do, or unable to do because of physical limitations imposed by extreme enviroments. Endowing robots with human-like abilities to perform motor skills in a smooth and natural way is one of the important goals of robotics. Reinforcment learning enables a robot to autonomously discover an optimal behavior through trial-and-error interactions with its enviroment.

Simulators offer unlimited tiral-and-error chances to perform the exploartion neccesary for RL. However, whether the policies learned in simulation can be reliably transferred to the real world hinges on accurate modeling of both robots and enviroments. Manufacturer supplied robot models offer a baseline, but often require significant tuning to be ready for sim-to-real. We extend prior work on parallized autotuning () by leveraging Bayesian optimization techniques to direclty minizie trackign error, rather than selecting the best from pre-sampled candidates.

This study aims to train a robotic arm to perform a multip-step radiochemical seperation process using deep rienforcment learning. As an initial step, the Soft Actor-Critic (SAC) algorithm is implemented to train a policy for pick-and-place operations in simulation.

## Methods

#### _Simulation Enviroment Setup_

#### _Real-to-Sim Modeling_

#### _Problem Statement_

#### _Proposed Solution_



## Results

The approach yielded a 31.4% reduction in total joint position RMSE with improvments observed in 

<table align="center">
  <thead>
    <tr>
      <th>Column 1</th>
      <th>Column 2</th>
      <th>Column 3</th>
      <th>Column 4</th>
      <th>Column 5</th>
    </tr>
  </thead>
  <tbody>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
    <tr><td></td><td></td><td></td><td></td><td></td></tr>
  </tbody>
</table>


## Acknoledgments


## References
