---
title: Tumor localization
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 4 · Medical robotics</p>

# Tumor localization by bioimpedance palpation

<p class="lede">A surgical robot locates a tumor by palpation and recovers its shape with only 25–50% of the samples, at recall above 90%.</p>

<div class="chips"><span>dVRK PSM2</span><span>ROS</span><span>RViz</span><span>MATLAB</span><span>Gaussian Process</span><span>Electrical bioimpedance</span><span>RealSense</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Goal** | Locate a tumor beneath the tissue surface, where vision cannot see, for minimally invasive surgery |
| **Robot** | da Vinci Research Kit (dVRK), patient-side manipulator PSM2 |
| **Sensing** | Electrical bioimpedance (EBI) probe in direct tissue contact |
| **Planner** | Gaussian Process active area search (MATLAB) |
| **Result** | Tumor shape reconstructed with 25–50% of the samples, recall >90% against ground truth |
| **Context** | Visiting researcher, Altair Robotics Laboratory, Verona (Jan – Jul 2020) |

</div>

<div class="media">
<figure><img src="../../assets/img/tumor_setup_big.jpg" alt="dVRK PSM2 palpating a sponge phantom with fiducial markers"><figcaption>Experimental setup: dVRK PSM2 palpating a sponge phantom with fiducial markers</figcaption></figure>
</div>

## Requirements

- Find and outline a tumor by **touch**, using bioimpedance (abnormal tissue reads differently from healthy tissue).
- **Minimize the number of palpations** compared with an exhaustive sweep.
- **Validate the software before** any trial on the physical robot.
- Measure accuracy against an **exhaustive ground truth** and show the result in **3D**.

## Architecture

<div class="flow">
<div class="node blue">1 · 3D acquisition<small>ROI point cloud and optical markers</small></div>
<div class="arr">⟶</div>
<div class="node green">2 · Kinematics & simulation<small>dVRK PSM2 control in RViz / ROS</small></div>
<div class="arr">⟶</div>
<div class="node rust">3 · Robotic palpation<small>EBI probe in direct tissue contact</small></div>
<div class="arr">⟶</div>
<div class="node purple">4 · Active planner<small>Gaussian Process (MATLAB)</small></div>
<div class="arr">⟶</div>
<div class="node magenta">5 · Map & validation<small>ground truth vs estimate</small></div>
</div>

ROS nodes synchronize the sensor, the manipulator and the planner. The region of interest is discretized in a **20 × 20 grid (400 points)**; the planner decides which cells actually need palpation.

## Key design decisions

| Decision | Alternative | Rationale |
|---|---|---|
| **Gaussian Process active area search** picks the next point to palpate | Exhaustive 20 × 20 sweep | Reconstructs the tumor with **25–50% of the samples** at recall above 90%, so far fewer contacts with the tissue |
| **ROS-based EBI sensor emulator** and an RViz model of the arm | Develop directly on the hardware | The whole pipeline (kinematics, safe trajectories, workspace, planner) is validated before any physical trial |
| **Fiducial markers + RealSense point cloud** | 2D maps only | Calibrates the camera and overlays the estimate on the 3D surface for validation |

<div class="media two">
<figure><img src="../../assets/img/tumor_rviz_arm.jpg" alt="da Vinci arm model in RViz"><figcaption>Digital model of the da Vinci arm in RViz</figcaption></figure>
<figure><img src="../../assets/img/tumor_rviz_grid.jpg" alt="400-point grid with the simulated tumor in green"><figcaption>400-point grid with the simulated tumor (green cylinder)</figcaption></figure>
</div>

## Results

<div class="media">
<figure><img src="../../assets/img/tumor_gp.jpg" alt="Ground truth and active area search stiffness maps, with RViz and MATLAB views"><figcaption>Ground truth stiffness map vs active area search, with the RViz and MATLAB views</figcaption></figure>
</div>

<div class="media three">
<figure><img src="../../assets/img/tumor_surface.jpg" alt="3D surface map from a depth sensor"><figcaption>3D surface map: depth sensor, 4 fiducial markers, circular region of interest</figcaption></figure>
<figure><img src="../../assets/img/tumor_truth.jpg" alt="Ground truth impedance map over the 3D relief"><figcaption>Ground truth: exhaustive impedance map over the 3D relief</figcaption></figure>
<figure><img src="../../assets/img/tumor_estimate.jpg" alt="Planned estimate with reduced samples"><figcaption>Planned estimate: reduced samples, same position and shape</figcaption></figure>
</div>

- The estimate reproduces the **position and concentric shape** of the tumor core (the high-impedance red-yellow region), with **recall above 90%** against the exhaustive ground truth.
- Achieved with **25–50% of the samples** of a full sweep.
- Reported experiments use a **sponge phantom**, with liquid soap and water to create the impedance contrast.

<div class="nextlink"><a href="../tiago-pro/">← Previous: TIAGo Pro</a><a href="../formation-control/">Next: Formation control →</a></div>
