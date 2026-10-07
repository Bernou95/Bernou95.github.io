---
title: Dual-actuation arm
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 2 · Robot software</p>

# Dual-actuation arm: remote high-level interface

<p class="lede">Software that lets neural controllers drive a research arm with muscle-like actuators from another computer, at 1 kHz.</p>

<div class="chips"><span>Rust</span><span>Zenoh</span><span>ROS 2</span><span>Raspberry Pi 4</span><span>1 kHz</span><span>Torque control</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Robot** | Research prototype with two actuators per joint (agonist and antagonist), designed from scratch |
| **Challenge** | The on-board controller is a Raspberry Pi 4 (no GPU), so neural controllers run off-board, driven over a remote link |
| **My role** | Software for the high-level interface in ROS 2, and the remote command link |
| **Stack** | Rust + Zenoh transport, ROS 2 interface |
| **Targets** | 1 kHz loop, torque control |
| **Context** | EPFL BioRobotics Laboratory visit (2025), with [KM-RoBoTa](https://km-robota.com/) on the hardware deployment |

</div>

## A research arm with two actuators per joint

Each joint has an **agonist and an antagonist actuator**, mimicking human muscles. Like muscles, a pair of opposing actuators allows dynamics such as **adjustable arm stiffness**. The robot was designed from scratch, and I joined at the stage where the software layer had to be built.

<div class="media two">
<figure><img src="../../assets/img/arm_alpha.jpg" alt="Alpha version of the dual-actuation arm"><figcaption>Alpha version</figcaption></figure>
<figure><img src="../../assets/img/arm_cad.jpg" alt="CAD of the new version"><figcaption>New version (CAD)</figcaption></figure>
</div>

## My part: a high-level interface over the network

The arm is controlled by a **Raspberry Pi 4** that handles low-level control. My objective was to develop software so **commands can be sent remotely**, which makes it possible to control the arm with **neural controllers** that need more compute than the on-board controller provides.

<div class="flow">
<div class="node navy">Remote PC<small>ROS 2 + neural controllers</small></div>
<div class="arr">⇄<small>Zenoh</small></div>
<div class="node teal">Raspberry Pi 4<small>low-level control, 1 kHz</small></div>
<div class="arr">⇄</div>
<div class="node slate">Dual-actuation arm<small>agonist / antagonist actuators</small></div>
</div>

I built the interface with **ROS 2**, keeping the target applications in mind:

- **Operating at 1 kHz.**
- **Torque control.**
- The **data that has to travel between computers**: forward and inverse kinematics, and commands.

## Stiffness in action

The two clips show the same arm **without stiffness** and **with stiffness**. Press play on either one to compare. It is a direct demonstration of the dynamics that the agonist/antagonist design makes possible, and of controlling the arm through the remote interface.

<div class="media two">
<figure class="tall"><video src="../../assets/video/arm_nostiff.mp4" poster="../../assets/video/arm_nostiff.jpg" controls preload="none" playsinline></video><figcaption>① Without stiffness</figcaption></figure>
<figure class="tall"><video src="../../assets/video/arm_stiff.mp4" poster="../../assets/video/arm_stiff.jpg" controls preload="none" playsinline></video><figcaption>② With stiffness</figcaption></figure>
</div>

<div class="nextlink"><a href="../cerebellar-snn/">← Previous: Cerebellar SNN</a><a href="../tiago-pro/">Next: TIAGo Pro →</a></div>
