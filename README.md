# IRoC-U Technical Survival Guide: The Unfiltered Senior Handover

> **"Look, if you're setting out to build a GPS-denied drone welcome to hell. But you're not alone. We burned cash, fried Flight Controllers, smoked Raspberry Pis, and lost countless nights of sleep so you don't have to repeat our dumb mistakes. Make good use of it."**



---

## The Mindset: Don't Be A Hero, Iterate

The number one mistake you guys are going to make: **"Bro, we'll straight up build a fully autonomous drone, we already wrote the ROS 2 nodes."**  
**Bullshit.** If you jump straight into autonomous flight, you will turn \$500 worth of hardware into scrap metal on day one.

Follow this exact sequence. No shortcuts:
1. **Fly Manually First:** At least one person on the team needs to know how to fly comfortably in manual mode (Stabilize). If you can't even fly it by hand, what the hell makes you think your code will fly it autonomously?
Make onep erson udnerstand all the modes and how does they work of the stack you using ;
2. **Get Your Simulation Right:** Everything must work flawlessly in SITL and Gazebo first. If your drone won't fly in sim, taking it to real hardware is an absolute crime.
3. **Fight Sim2Real:** Simulation is an ideal wonderland. Real hardware brings ground effect, air drag, motor vibrations, voltage drops, and dirty sensor noise. Sim2Real is a whole different beast.
4. **Verify One Task At A Time:** First stable hover -> then manual guided square -> then optical flow hold -> then visual odometry -> then autonomous waypoints. Test one single thing at a time.
Whatevr maybe the task ; one task at a time ;

---

## The Golden Rules (Swear By These)

1. **RECORD EVERY FUCKING THING:**  
   Every single flight test, every time you plug in a LiPo battery, after every crash, and after every tragedy—**pull out your phone and record videos and photos**. When hardware crashes or flips on takeoff, high-speed slow-mo video is the only thing that will tell you whether a prop shattered first or the autopilot disarmed.
2. **Get Yourself 2 Mechanical Guys (PLEASE):**  
   Computer engineering students always assume software can fix everything. When motor shafts don't fit your props, frame vibrations make the EKF freak out, and your 3D mounts snap like twigs, you will sit in the lab and cry. Keep two dedicated mechanical teammates to handle structural rigidity, motor mounts, and prop tolerances.
3. **Embrace AI Tools (Claude Code / Codex):**  
   I was honestly a non-AI person before this, and I realized my mistake way too late. Competitions run on a completely different speed and mindset. Get Claude Code or Codex—they are ridiculously good. Even the core ArduPilot and PX4 dev communities are heavily embracing AI tooling now. Use AI to scaffold boilerplate and debug nasty ROS 2 setups, but keep your brain turned on.
4. **The Discord Community Is Your Lifeline:**  
   When you're stuck, don't bang your head against the wall for 10 hours. Hop onto the ArduPilot Discord; the core devs there are insanely helpful:
   - `@RyanF` (The god of DDS / Micro-ROS)
   - `@RhysMainwaring` (Gazebo simulation and GROS)
   - `@PeterBarker` (General dev, core ArduPilot & SITL)
   - `@NateMailhot` (General dev & EKF wizard)
   - `@AndyPiper` (General dev & modern tooling)

---

## Survival Guide Index

The entire guide is split across these 6 modules. Go through them step-by-step:

1. [01 - SITL and Gazebo](./01-Simulation-First/01-sitl-and-gazebo.md)  
   *SITL + Gazebo setup, joystick RC hacks, gimbal panics, the GStreamer trap, and custom 3D worlds.*
2. [02 - ROS 2 vs PyMAVLink & DDS](./02-Software-Architecture/01-ros2-vs-pymavlink.md)  
   *PyMAVLink vs ROS 2, MAVROS vs DDS (the 1MB FC RAM limit trap), GCS picks, and hacking custom firmware binaries.*
3. [03 - Optical Flow and Rangefinder](./03-The-GPS-Denied-Transition/01-optical-flow-and-rangefinder.md)  
   *First steps in GPS-denied: replicating MTF-01 optical flow + rangefinder, tweaking EKF parameters, and why custom precision landing scripts are a waste of time.*
4. [04 - ORB-SLAM3 and ROS 2 TF](./03-The-GPS-Denied-Transition/02-orbslam3-and-tf.md)  
   *ORB-SLAM3 on Ubuntu 24.04 (hunting community forks), RealSense depth feeds, `/ap/tf`, tracking states, and the FRD vs FLU coordinate frame nightmare.*
5. [05 - Hardware Fails & Reality Checks](./04-Hardware-and-Reality-Checks/01-hardware-fails.md)  
   *Smoke stoppers, frying Pis via GPIO, brownouts, short motor shafts & 1045 self-locking props, the 3 CCW + 1 CW blunder, and our honest benchmarking disaster.*
6. [06 - Reading Dataflash Logs & Debugging](./05-Debugging/01-reading-the-matrix-logs.md)  
   *Learning to read Dataflash `.bin` logs, catching EKF innovation blowups, vibration clipping, and injecting custom debug prints directly into ArduPilot source code.*

Good luck. Break less hardware, fly more.
