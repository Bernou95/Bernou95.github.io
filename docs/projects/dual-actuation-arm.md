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
| **Challenge** | Its Raspberry Pi 4 only supports low-level control, so neural controllers cannot run on board |
| **My role** | Software for the high-level interface in ROS 2, and the remote command link |
| **Stack** | Rust + Zenoh transport, ROS 2 interface |
| **Targets** | 1 kHz loop, torque control |
| **Context** | EPFL BioRobotics Laboratory visit (2025), with KM-RoBoTa on the hardware deployment |

</div>

## A research arm with two actuators per joint

Each joint has an **agonist and an antagonist actuator**, trying to imitate our muscles. Like muscles, a pair of opposing actuators allows dynamics such as **adjustable arm stiffness**. The robot was designed from scratch, and the project went through two hardware iterations.

<div class="media two">
<figure><img src="../../assets/img/arm_alpha.jpg" alt="Alpha version of the dual-actuation arm"><figcaption>Alpha version</figcaption></figure>
<figure><img src="../../assets/img/arm_cad.jpg" alt="CAD of the new version"><figcaption>New version (CAD)</figcaption></figure>
</div>

## My part: a high-level interface over the network

The arm is controlled by a **Raspberry Pi 4**, which only allows controlling it at low level. My objective was to develop software so **commands can be sent remotely**, which makes it possible to control the arm with **neural controllers** that cannot be implemented on the on-board controller because of hardware limits.

<div class="flow">
<div class="node navy">Remote PC<small>ROS 2 + neural controllers</small></div>
<div class="arr">⇄<small>Zenoh</small></div>
<div class="node teal">Raspberry Pi 4<small>low-level control, 1 kHz</small></div>
<div class="arr">⇄</div>
<div class="node slate">Dual-actuation arm<small>agonist / antagonist actuators</small></div>
</div>

I built the interface with **Zenoh in Rust**, keeping the target applications in mind:

- **Operating at 1 kHz.**
- **Torque control.**
- The **data that has to travel between computers**: forward and inverse kinematics, commands and so on.

## Stiffness in action

The clip shows the same trajectory performed twice: first **without stiffness**, then **with stiffness**. It is a direct demonstration of the dynamics that the agonist/antagonist design makes possible, and of controlling the arm through the remote interface.

<div class="media">
<figure class="tall"><video src="../../assets/video/stiffness_demo.mp4" poster="../../assets/video/stiffness_demo.jpg" controls loop muted playsinline></video><figcaption>Same trajectory: ① without stiffness, then ② with stiffness</figcaption></figure>
</div>

<div class="nextlink"><a href="../cerebellar-snn/">← Previous: Cerebellar SNN</a><a href="../tiago-pro/">Next: TIAGo Pro →</a></div>
