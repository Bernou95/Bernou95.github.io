---
title: TIAGo Pro
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 3 · Mobile manipulation</p>

# TIAGo Pro: a ROS 2 torque interface for both arms

<p class="lede">The TIAGo Pro did not expose torque control to users through ROS 2 topics. I built the ROS 2 interface to its arm controllers, with a state machine that allows only one controller (position, velocity or torque) per arm, and ran it on the real robot at 500 Hz.</p>

<div class="chips"><span>ROS 2</span><span>ros2_control</span><span>MoveIt 2</span><span>State machine</span><span>Gazebo Classic</span><span>Docker</span><span>C++</span><span>Python</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Motivation** | No torque interface was exposed to the user through ROS 2 topics, so neural and torque controllers could not be tried on the arms |
| **What I built** | A ROS 2 interface to the arm controllers, plus a state machine that allows **one** controller (position, velocity or torque) per arm at a time |
| **Safety** | Position hold by default, atomic mode switching, 0.5 s watchdog back to position hold |
| **Control rate** | Safe effort-based control at 500 Hz, tested on the real robot |
| **Planning** | MoveIt 2 for trajectory planning (control stays at joint level) |
| **Environment** | Docker, so the robot's own dependencies are never broken; Gazebo simulation first |

</div>

<div class="media">
<figure class="tall"><video src="../../assets/video/tiago.mp4" poster="../../assets/video/tiago.jpg" autoplay loop muted playsinline></video><figcaption>Torque control running on the real TIAGo Pro</figcaption></figure>
</div>

## 1. The problem: no torque interface for the user

The TIAGo Pro is a dual-arm mobile manipulator. For my research, I needed to send **torque commands** to its arms, for example from controllers that are learned rather than hand-designed. But the robot did **not expose a torque interface to the user through ROS 2 topics**: there was no ready-made way to publish efforts to the arm controllers.

So the work is not a small tweak: it is the **development of the ROS 2 interface itself**, together with the safety logic that makes it usable on a physical robot, where a wrong torque command can damage the hardware.

## 2. The solution: an interface and a state machine

<div class="flow">
<div class="node navy">User controller<small>publishes torques or joint commands</small></div>
<div class="arr">⟶<small>ROS 2 topics</small></div>
<div class="node teal">Motion controller interface<small>state machine, one mode per arm</small></div>
<div class="arr">⟶<small>ros2_control</small></div>
<div class="node slate">Arm controllers<small>position · velocity · effort</small></div>
<div class="arr">⟶</div>
<div class="node rust">TIAGo Pro arms<small>or Gazebo simulation</small></div>
</div>

Each arm has its own command topics and mode selection, named with an `arm_left_` / `arm_right_` prefix (for example `arm_left_effort_joint_controller/commands`). Behind them, a **state machine** applies one rule: **only one controller is active per arm** at any time.

<div class="cols3" markdown>
<div class="card c1" markdown>
<p class="ctitle">Position mode</p>

- **Default state at startup:** the arm holds its posture rigidly.
- Used for planning and as the safe fallback.
</div>
<div class="card c2" markdown>
<p class="ctitle">Velocity mode</p>

- One of the three exclusive controllers.
- Only active when the position and torque controllers are not.
</div>
<div class="card c3" markdown>
<p class="ctitle">Torque (effort) mode</p>

- Accepts torque commands from the user's topics.
- A **0.5 s watchdog** aborts to position hold if the command stream stops.
</div>
</div>

- **One controller per arm** prevents conflicting commands from reaching the same joints.
- The two arms are **independent**: one can be in torque mode while the other holds position.
- **Switching is requested through the interface,** not by hand-starting and stopping controllers, so the safety logic always runs.
- **MoveIt 2** plans trajectories, since control stays at joint level, and the robot's own controllers execute them.

## 3. A bug on the way: making mode switches safe

Implementing the state machine exposed an engineering problem I am proud of having solved. When you switch an arm from position control to torque control, any gap leaves the arm unsupported.

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
- **Position mode forced at startup** plus the **0.5 s watchdog** that falls back to position hold.
</div>
</div>

My first approach was a two-phase switch with an overlap, but in practice the overlap froze the simulated hardware. The plugin's stop loop did not clear only its own bit: it zeroed the **whole control-method register**, destroying the state of the concurrent effort controller. Batching both lists into a single call means the register is rewritten correctly within the same cycle, so the transition is imperceptible and physically stable. This is what the state machine's mode changes rely on.

## 4. Supporting software

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

## 5. Development environment

The whole environment ran in **Docker on ARM64 macOS**, which required working around a missing Gazebo Classic plugin binary for that architecture. Docker also keeps the robot's own dependencies untouched, which matters because the same dependencies serve all of the robot's controllers.

<div class="nextlink"><a href="../dual-actuation-arm/">← Previous: Dual-actuation arm</a><a href="../tumor-localization/">Next: Tumor localization →</a></div>
