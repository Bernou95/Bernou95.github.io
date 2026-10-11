---
title: TIAGo Pro
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 3 · Mobile manipulation</p>

# TIAGo Pro: a ROS 2 torque interface for both arms

<p class="lede">Safe torque control of both TIAGo Pro arms over ROS 2 at 500 Hz, validated on the real robot.</p>

<div class="chips"><span>ROS 2</span><span>ros2_control</span><span>MoveIt 2</span><span>State machine</span><span>Gazebo Classic</span><span>Docker</span><span>C++</span><span>Python</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Outcome** | External controllers (e.g. neural controllers) can command joint torques on both arms through ROS 2 topics |
| **What I built** | A ROS 2 mode-manager node on top of ros2_control: one active controller per arm (position, velocity or effort), selected by a state machine |
| **Control rate** | 500 Hz effort control, tested on the real robot |
| **Safety** | Position hold at startup, 0.5 s command watchdog that falls back to position hold |
| **Stack** | C++ and Python nodes, ros2_control, MoveIt 2, Gazebo Classic, Docker |

</div>

<div class="media">
<figure class="tall"><video src="../../assets/video/tiago.mp4" poster="../../assets/video/tiago.jpg" autoplay loop muted playsinline></video><figcaption>Torque control running on the real TIAGo Pro</figcaption></figure>
</div>

## Requirements

- Stream **joint torques** from an external controller over **ROS 2 topics at 500 Hz**.
- **Exactly one controller owns each arm** at any time; the two arms are independent.
- **Fail safe:** if the command stream stops, the arm holds its position.
- Leave the **vendor software stack unmodified**.
- A **reproducible** development and deployment environment.

## Architecture

<div class="flow">
<div class="node navy">User controller<small>publishes torques or joint commands</small></div>
<div class="arr">⟶<small>ROS 2 topics</small></div>
<div class="node teal">Mode manager<small>state machine, one mode per arm</small></div>
<div class="arr">⟶<small>controller_manager</small></div>
<div class="node slate">Arm controllers<small>position · velocity · effort</small></div>
<div class="arr">⟶</div>
<div class="node rust">TIAGo Pro arms<small>or Gazebo simulation</small></div>
</div>

Users publish commands on per-arm topics (for example `arm_left_effort_joint_controller/commands`) and request a mode through the interface. The mode manager runs one state machine per arm and asks `controller_manager` to activate the matching ros2_control controller. All three controllers can be loaded and waiting, but only the active one claims the arm's joint interfaces.

## Key design decisions

| Decision | Alternative | Rationale |
|---|---|---|
| Reuse ros2_control controllers through `controller_manager` | Write a custom hardware interface | Keeps the vendor's tested hardware layer untouched; the work stays in user space |
| Per-arm **state machine** decides which controller is active | Users start and stop controllers by hand | Several controllers can be loaded and waiting, but only one can claim an arm's joint interfaces. The state machine makes ownership explicit and every switch goes through the same safety logic |
| **Topic streaming** for commands | Services or actions | Continuous 500 Hz command stream; any external controller can publish without depending on my code |
| **Position hold** as the startup and fallback state | Start in effort mode | The arm is supported before the first torque command arrives |
| **0.5 s command watchdog** back to position hold | No timeout, or a very short one | Tolerates transport delays without false triggers, but is short enough to stop the arm before it can fall or self-collide. Tuned empirically on the robot |
| `arm_{side}` **parameterized** nodes (`declare_parameter`) | One copy of the code per arm | A single binary controls either arm, selected at launch without recompiling |
| **Docker** environment | Native install on the robot PC | Reproducible setup; the robot's own dependencies, shared by all its controllers, stay untouched |

## Verification

- Developed and tested first in **Gazebo**, then deployed on the **real robot**.
- Effort control running at **500 Hz** on the real robot (video above). The arms are independent: one can run in torque mode while the other holds position.

## Supporting work

- **ROS 1 → ROS 2 / MoveIt 2 migration.** The TIAGo Pro has no ROS 1 support, and the controllers needed test trajectories, so I ported the trajectory-planning code to MoveIt 2 (with a background `rclpy` executor thread, as MoveIt 2 requires).
- **Trajectory dispatcher.** Four scripts (circle, infinity, square, target reaching) unified into `dual_arm_trajectory.py`, selected with `trajectory:=circle`, solving inverse kinematics for both arms.
- **Full-body controllers.** `full_controllers:=true` also brings up head, torso and mobile base, which prevents drift from the arms' reaction forces; the joint-state broadcaster discovers every joint from the URDF.

<div class="nextlink"><a href="../dual-actuation-arm/">← Previous: Dual-actuation arm</a><a href="../tumor-localization/">Next: Tumor localization →</a></div>
