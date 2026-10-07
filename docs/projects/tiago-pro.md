---
title: TIAGo Pro
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 3 · Mobile manipulation</p>

# TIAGo Pro: safe dual-arm torque control in ROS 2

<p class="lede">Safe position-to-torque switching and dual-arm control for the TIAGo Pro, running on the real robot at 500 Hz.</p>

<div class="chips"><span>ROS 2</span><span>ros2_control</span><span>MoveIt 2</span><span>Gazebo Classic</span><span>Docker</span><span>C++</span><span>Python</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Goal** | Torque controllers for both arms of a TIAGo Pro, tested on the real robot |
| **Control rate** | Safe effort-based control at 500 Hz |
| **Planning** | MoveIt 2 for trajectory planning (control stays at joint level) |
| **Mode switching** | `ros2_control`, from position to torque mode, with safety mechanisms |
| **Environment** | Docker, so the robot's own dependencies are never broken |
| **Highlight** | Root-caused a hardware-plugin bug by disassembling its binary |

</div>

<div class="media">
<figure class="tall"><video src="../../assets/video/tiago.mp4" poster="../../assets/video/tiago.jpg" autoplay loop muted playsinline></video><figcaption>Torque control running on the real TIAGo Pro</figcaption></figure>
</div>

## Not just an interface

This work is not only the development of the controller interface. It combines:

- **MoveIt 2** for trajectory planning, because we keep doing joint-level control.
- **Safety mechanisms** around torque operation.
- **Switching between control modes**, from position to torque, with the help of **`ros2_control`**.
- A lot of work behind the scenes in **simulation**, and the mandatory use of **Docker** so as not to damage dependencies across all the robot's controllers.

## Deep dive: why a controller switch froze the robot

This is the engineering problem I am proudest of in this project. When you switch an arm from position control to torque control, any gap leaves the arm unsupported.

<div class="cols3" markdown>
<div class="card c1" markdown>
<p class="ctitle">1 · Problem</p>

- Gazebo needs ~15 s to start; torque commands begin at zero, so **gravity collapses the arms**.
- A naive position → effort switch leaves **~200 ms of free fall**.
- A two-phase switch with a **300 ms overlap froze the hardware registers**.
</div>
<div class="card c2" markdown>
<p class="ctitle">2 · Root cause</p>

- Found by **disassembling the PAL hardware plugin** (`libgazebo_hardware_plugins.so`).
- With both controllers active: `joint_control_method = 0x01 | 0x04 = 0x05`.
- Stopping the position controller **cleared the whole register to `0x00`**, killing the effort controller too.
</div>
<div class="card c3" markdown>
<p class="ctitle">3 · Fix</p>

- **One atomic `switch_controller` call:** `activate=[effort]`, `deactivate=[position]`.
- The stop loop clears the register and the start loop rewrites `0x04` **in the same cycle**.
- **Position mode forced at startup** plus a **0.5 s watchdog** that falls back to position hold.
</div>
</div>

My first approach was a two-phase switch with an overlap, but in practice the overlap froze the simulated hardware. The plugin's stop loop did not clear only its own bit: it zeroed the **whole control-method register**, destroying the state of the concurrent effort controller. Batching both lists into a single call means the register is rewritten correctly within the same cycle, so the transition is imperceptible and physically stable. On top of that, the system forces position mode at startup, to survive the Gazebo initialization window, and the watchdog returns the arms to position hold if torque commands stop.

## Software engineering behind it

<div class="cols4" markdown>
<div class="card" markdown>
<p class="ctitle">ROS 1 → ROS 2 migration</p>
The trajectory-planning setup was developed in MoveIt with ROS 1. Migration to MoveIt 2 was needed to test different trajectories. MoveIt in ROS 2 requires a **secondary executor thread** (`rclpy.spin`) in the background. Planning groups and joints were renamed (`arm_right`, `arm_right_[1-7]_joint`).
<p class="cstack">rclpy · MoveIt 2</p>
</div>
<div class="card" markdown>
<p class="ctitle">Parametric multi-arm topics</p>
Hard-coded topics were replaced by **`arm_{side}_` prefixes** resolved with `declare_parameter` in the C++ nodes, so a single binary controls either arm, selected at launch **without recompiling**.
<p class="cstack">C++ · ROS 2 params</p>
</div>
<div class="card" markdown>
<p class="ctitle">4 scripts → 1 dispatcher</p>
Circle, infinity, square and target-reaching scripts were unified into **`dual_arm_trajectory.py`**, selected with `trajectory:=circle`. It solves **inverse kinematics for both arms** at once.
<p class="cstack">Python · IK</p>
</div>
<div class="card" markdown>
<p class="ctitle">Full-body activation</p>
`full_controllers:=true` brings up **head, torso and mobile base**: leaving the body inert caused physical drift from the arms' reaction forces. The joint-state broadcaster now **auto-discovers every joint in the URDF** instead of using a fixed list of 14.
<p class="cstack">ros2_control · URDF</p>
</div>
</div>

## Development environment

The whole environment ran in **Docker on ARM64 macOS**, which required working around a missing Gazebo Classic plugin binary for that architecture. Docker also keeps the robot's own dependencies untouched.

<div class="nextlink"><a href="../dual-actuation-arm/">← Previous: Dual-actuation arm</a><a href="../tumor-localization/">Next: Tumor localization →</a></div>
