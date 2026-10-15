# World-Model-Based Visual Humanoid Locomotion with Terrain-Oriented Training and Multi-Rate Deployment

Supplementary experimental videos and an overview of the locomotion framework presented in the paper.

## Experiment Videos

All supplementary videos are also available in the [Supplementary Video issue](https://github.com/shoukatng636-max/WMVHL/issues/1).

### 圆形台阶

https://github.com/user-attachments/assets/107611a6-2855-40a0-8b12-a8570d64543d

### 崎岖地形

https://github.com/user-attachments/assets/a6e987b6-b629-4b14-9a11-249e81b3f459

### sim2sim

https://github.com/user-attachments/assets/4dd0f047-8136-4739-a95f-99d8906d396c

<!-- Optional: add a public paper link here once it is available. -->

## Overview

This work presents a world-model-based visual locomotion framework for the **Unitree G1 humanoid robot**. A recurrent state-space model (RSSM) combines depth observations and proprioceptive information into a compact temporal representation. A PPO locomotion controller uses this representation together with proprioceptive feedback and velocity commands to traverse discontinuous and irregular terrain.

The framework combines three elements:

- **Recurrent visual perception:** depth history and proprioception are encoded into deterministic and stochastic RSSM states that condition the locomotion policy.
- **Terrain-oriented training:** feasible local terrain targets, posture adaptation, foot-clearance regulation, and edge-aware contact penalties support traversal of stairs, obstacles, and irregular terrain.
- **Multi-rate deployment:** the world-model representation is refreshed at **10 Hz**, while the locomotion policy runs at **50 Hz**. Between world-model updates, the actor reuses the latest latent representation and receives fresh proprioceptive feedback at every policy step.

The policy is trained in **Isaac Gym**, transferred to **MuJoCo without retraining or fine-tuning**, and deployed on a physical Unitree G1.

## Method

Depth history and current proprioception are encoded by separate observation branches and fused by the RSSM. Its deterministic and stochastic states provide the temporal representation for control.

An asymmetric PPO actor-critic is used during training. The actor receives deployable proprioception, proprioceptive history, velocity commands, and RSSM states; the critic additionally receives simulator-only privileged information. A stop-gradient interface separates PPO updates from world-model optimization.

Terrain-derived local targets and reward terms are used during training. Deployment uses the learned representation and deployable policy observations.

## Experimental Results

### Zero-Shot Transfer to MuJoCo

Policies are trained in Isaac Gym and evaluated directly in MuJoCo. Each terrain condition is evaluated over **10 trials**.

| Method | 10 cm stairs | 15 cm stairs | Rough terrain | Falls (total) | Edge slips (total) |
| --- | ---: | ---: | ---: | ---: | ---: |
| AMP | 8/10 | 2/10 | 8/10 | 12 | 15 |
| Ours | 9/10 | 7/10 | 9/10 | 5 | 8 |

The terrain columns report successful trials. Falls and edge slips are aggregate counts across the listed MuJoCo evaluation conditions.

### Real-World Evaluation on Unitree G1

The outdoor evaluation compares the manufacturer-provided **Base** controller, the proprioceptive **AMP** baseline, and the proposed complete system. Each terrain is evaluated over **10 trials**.

| Terrain | Base | AMP | Ours |
| --- | ---: | ---: | ---: |
| Low step, 12 cm | 7/10 | 9/10 | 9/10 |
| High step, 15 cm | 4/10 | 8/10 | 9/10 |
| Grass | 7/10 | 8/10 | 9/10 |
| Gravel | 6/10 | 7/10 | 8/10 |
| Slope, 15 degrees | 9/10 | 9/10 | 9/10 |
| **Total** | **33/50** | **41/50** | **44/50** |

Additional physical demonstrations include traversal of a **10 cm isolated obstacle** and indoor stair climbing on soft steps and rigid wooden stairs.

These results describe the tested conditions and trial counts. The comparison evaluates the complete system: visual input, recurrent representation, terrain-oriented objectives, and deployment structure change together, so the experiments do not isolate the contribution of any single component.

## Platform

| Component | Configuration |
| --- | --- |
| Robot | Unitree G1 |
| Depth camera | Intel RealSense D435i |
| Onboard compute | NVIDIA Jetson Orin NX, 16 GB |
| Middleware | ROS 1, with separate perception and locomotion-control nodes |
| Training simulator | Isaac Gym with PhysX |
| Transfer evaluation | MuJoCo |
| World-model refresh rate | 10 Hz |
| Locomotion policy rate | 50 Hz |

## Scope

The reported evaluation covers selected obstacles, stairs, grass, gravel, and slopes. Severe depth failures and more extreme unstructured terrain are outside the current evaluation. The deployed 10 Hz world-model refresh rate is an operating point; it has not been established as optimal.
