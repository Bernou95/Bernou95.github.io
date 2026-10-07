---
title: Formation control
---

<p class="backlink"><a href="../">← All projects</a></p>
<p class="kicker-top">Project 5 · Multi-robot systems · Published</p>

# Formation control with obstacle avoidance

<p class="lede">Robots keep a formation while following a leader and avoid obstacles, with continuous velocities at all times. Tested on three real robots.</p>

<div class="chips"><span>ROS</span><span>ROS 2</span><span>Gazebo Sim (Harmonic)</span><span>Consensus control</span><span>OptiTrack</span><span>TurtleBot 2</span><span>Pioneer 3DX</span></div>

<div class="facts" markdown>

| | |
|---|---|
| **Paper** | J. B. Martinez, H. M. Becerra and D. Gomez-Gutierrez. *Formation tracking control and obstacle avoidance of unicycle-type robots guaranteeing continuous velocities.* **Sensors** 2021, 21(13), 4374 |
| **My role** | First author. Software, methodology, validation (per the paper's author contributions) |
| **Robots** | Pioneer 3DX (leader) + 2 TurtleBot 2 (followers), differential-drive |
| **Software** | ROS over WiFi, positions from an OptiTrack motion-capture system |
| **Later** | Full stack migrated to ROS 2 and Gazebo Sim (Harmonic), refactoring node architecture and communication patterns |

</div>

<p class="links">
<a href="https://www.mdpi.com/1424-8220/21/13/4374">📄 Read the paper</a>
<a href="https://drive.google.com/file/d/1RVNe52Gh6gmc2AtGyeuERfKY54dixHz-/view">▶ Video: linear trajectory</a>
<a href="https://drive.google.com/file/d/1bJZBO7dByTipDryC0XqHJurHE4f-d_i1/view?usp=sharing">▶ Video: circular trajectory</a>
</p>

## The core idea: two prioritized tasks

A group of mobile robots must **keep a formation** while a leader follows a trajectory, and **avoid obstacles**, including each other. The scheme solves both tasks simultaneously with a hierarchy:

<div class="flow">
<div class="col">
<div class="node rust">Obstacle avoidance<small>high priority, active only near an obstacle</small></div>
<div class="node navy">Formation tracking<small>low priority, consensus with the leader included</small></div>
</div>
<div class="arr">⟶</div>
<div class="node slate">Hierarchical task control<small>smooth transition function h(t)</small></div>
<div class="arr">⟶</div>
<div class="node teal">Continuous velocities<small>for every robot</small></div>
</div>

- **Obstacle avoidance** has the higher priority and only activates when a robot gets close to an obstacle, inside a security distance.
- **Formation tracking** has the lower priority and always runs. It is based on **consensus**, with the leader included as part of the formation.
- A **smooth transition function** between the two keeps every robot's velocity **continuous**, even when avoidance switches on or off, and **stability is proven** during the transition.
- It is **fully distributed**: each robot needs only relative information from its neighbors and local obstacle measurements, so **no global coordinate system or map** is required.
- It is **scalable** to many robots and **generic**: any desired formation can be defined.

The paper also evaluates the scheme in simulation with 4 and 10 agents and in environments with unknown polygonal obstacles, before the real-robot experiments below.

## Real-robot experiments

**Setup.** A Pioneer 3DX is the leader and two TurtleBot 2 are the followers, in a **triangular formation** at 0.7 m displacement from the leader. Each robot is connected via WiFi to a computer where the velocity commands are computed in a **distributed way using ROS**: each agent publishes its computed velocities and reads those of its neighbors. Robot and obstacle positions come from an **OptiTrack** system. The gains were γ = 0.8 (tracking), λ = 0.8 (evasion) and k = 0.12 (consensus).

<div class="cols2" markdown>
<div class="card" markdown>
<p class="ctitle">Linear trajectory (30 s)</p>

- The leader tracks a straight line with a **fixed obstacle** nearby (security distance 0.4 m).
- One follower **evades the obstacle at about 18 s**, which also moves the others, with only a small effect on the leader.
- The group **recovers the formation** and the virtual agents reach **consensus** at the final point.
- Computed velocities stay **continuous** throughout, thanks to the transition function.

<p class="links"><a href="https://drive.google.com/file/d/1RVNe52Gh6gmc2AtGyeuERfKY54dixHz-/view">▶ Watch the video</a></p>
</div>
<div class="card" markdown>
<p class="ctitle">Circular trajectory (radius 1.1 m, 70 s)</p>

- The leader tracks a circle that returns to its starting point, with a **fixed obstacle** (security distance 0.55 m).
- The robots **deviate around the obstacle between about 53 s and 65 s**, and then **return to the formation**.
- After the avoidance finishes, **tracking and consensus errors converge to zero**, as proven in the paper.
- Velocities again stay **continuous** throughout.

<p class="links"><a href="https://drive.google.com/file/d/1bJZBO7dByTipDryC0XqHJurHE4f-d_i1/view?usp=sharing">▶ Watch the video</a></p>
</div>
</div>

## Why it matters

Existing schemes rarely combine all of the following: **fully distributed**, **scalable**, generic in the formation, no need for a global frame or a map, and **continuous velocities** when obstacle avoidance activates. The hierarchical formulation covers all of them at once, and it is the first time, to the authors' knowledge, that distributed formation tracking is combined with obstacle avoidance in this way.

<div class="nextlink"><a href="../tumor-localization/">← Previous: Tumor localization</a><a href="../">All projects →</a></div>
