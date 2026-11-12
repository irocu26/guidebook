# Title: GPS-Denied Transition - Optical Flow, Rangefinders, and EKF Drama

**The Concept:** 
In a GPS-denied arena, your autopilot doesn't have satellite fixes to fall back on. The Flight Controller desperately needs alternative sensors to calculate its position and velocity. An Optical Flow sensor tracks ground pixel shifts to estimate horizontal velocity, while a Rangefinder (Lidar/Sonar) measures altitude, fusing together inside EKF3 to give you a drift-free hover.

**How Should you DO:** 
- Don't Blindly trust the IMU: We naively assumed that the IMU (accelerometer + gyro) could handle dead reckoning and let us fly in GPS-denied spaces. It completely failed! IMU double-integration drift is so bad that the drone won't even arm in position-dependent modes (like LOITER or GUIDED). You need another source ; use MTF 01 module (rec).
- Understand parameters:  You have to  tweak values yourself, and see what breaks and why. Try to make the  necessary changes for GPS Denied 
- Replicate the Hardware  you have in SITL : We tried adding an optical flow + rangefinder combo in SITL (trying to replicate an MTF-01 module) using SITL peripheral docs;

- Wasting time on custom Precision Landing: I tried being heroes and wrote our own precision landing PID algorithm from scratch, thinking it would give us more granular control. It was a complete waste of time! ArduPilot's native Precision Landing mode is already rock solid and heavily battle-tested. Never reinvent the wheel when the built-in mode already works reliably.

**The Jargon:** 
- **Optical Flow:** A downward-facing camera sensor that tracks ground feature displacement to calculate the drone's horizontal velocity $(\Delta x, \Delta y)$.
- **Rangefinder (ToF / Lidar):** A distance sensor measuring height above the ground; without it, optical flow cannot convert pixel movement into metric speed.
- **MTF-01:** Microlink's popular integrated Optical Flow + Lidar module commonly used for GPS-denied indoor drone navigation.
- **EKF3 (Extended Kalman Filter 3):** ArduPilot's multi-sensor fusion math engine that blends noisy IMU, optical flow, and rangefinder data into a stable position estimate.
- **Precision Land:** A native ArduPilot flight mode that locks onto visual targets or beacons to land autonomously with centimeter-level accuracy.

**The Task (Do It Yourself):** 
- Step 1: Configure optical flow and rangefinder peripherals in SITL and set your EKF parameters (`EK3_SRC1_POSXY = 0`, `EK3_SRC1_VELXY = 5` [OpticalFlow], `EK3_SRC1_POSZ = 2` [RangeFinder]).
- Step 2: Arm the drone in GPS-denied conditions inside LOITER mode and verify that it holds a stable hover without requiring manual stick corrections.
- Step 3: Skip writing custom landing logic and configure ArduPilot's native Precision Landing mode to touch down on a designated target pad.
- Question for you: If your rangefinder suddenly spikes to 0 or reads out-of-range (say, over a shiny or reflective floor), can EKF3 still trust your optical flow velocity data, and what happens to your altitude hold?

**Where to find the answers:** 
- ArduPilot Non-GPS Navigation: [ArduPilot Non-GPS Navigation Hub](https://ardupilot.org/copter/docs/common-non-gps-navigation-landing-page.html)
- Optical Flow Sensor Setup: [Optical Flow Configuration](https://ardupilot.org/copter/docs/common-optical-flow-sensor-setup.html)
- Precision Landing Documentation: [Precision Landing Docs](https://ardupilot.org/copter/docs/precision-landing-with-irlock.html)
- SITL peripherals , Add sensors to drone: [SITL peripherals](https://ardupilot.org/dev/docs/adding_simulated_devices.html)
