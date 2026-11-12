# Title: Visual SLAM & Frames - ORB-SLAM3, ROS 2 TF, and Coordinate Hell

**The Concept:** 
Optical flow only works across flat surfaces for 2D velocity; true 3D spatial navigation in complex GPS-denied arenas requires Visual SLAM (like ORB-SLAM3). A depth camera (like an Intel RealSense RGB-D) computes a 3D pose estimate that your companion computer feeds into ArduPilot's EKF3 over the ROS 2 topic `/ap/tf` as external visual odometry.

**How You SHould Proceed:** 
- The Ubuntu 24.04 (Noble) build mismatch: The official ORB-SLAM3 repo does not build out-of-the-box on Ubuntu 24.04! C++ standards, Pangolin, and OpenCV versions are completely broken. We had to dig through community forks—one developer had forked ORB-SLAM3 core for 24.04, and someone else had created a separate fork for the ROS 2 wrapper! If a repo won't compile on your OS, don't give up: dive into GitHub issues and PRs, and grab a working community fork.
- Ignoring camera calibration & intrinsics: If you have an Intel RealSense depth camera, you get factory-calibrated intrinsics out of the box. But if you use any other camera, you must print a checkerboard pattern and calibrate the lens to extract the intrinsic matrix. If you skip this, your SLAM will lose tracking every 5 seconds.
- Ignoring Tracking States: We didn't bother reading the ORB-SLAM3 docs properly. We were blindly feeding coordinates to the FC without checking whether the tracking state was `OK`, `RECENTLY_LOST`, or completely `LOST`. When SLAM loses tracking and you feed old or garbage coordinates to the FC, the drone twitches violently and crashes.
- Frame calibration and the FRD vs FLU nightmare: Our biggest disaster was coordinate frames. ROS 2 defaults to the FLU (Forward-Left-Up) frame, but ArduPilot's FC strictly expects coordinates in FRD (Forward-Right-Down: forward = +X, right = +Y, down = +Z). We wasted hours trying random time delays and axis flips based on blind guesswork. The real fix was diving into ArduPilot's `libraries/AP_DDS/` (external_nav) source, adding coordinate print logs, and watching what numbers actually hit the FC.
- Note on alternatives: ORB-SLAM3 isn't the only game in town. Many teams use OpenVINS or other visual odometry packages, and those work great too.

**The Jargon:** 
- **ORB-SLAM3:** A real-time visual SLAM library that tracks visual features to calculate a camera's 6-DoF trajectory and map its environment.
- **TF / TF2:** ROS 2's coordinate transform library that keeps track of spatial relationships between sensor frames (`camera_link`, `base_link`, `odom`, `map`).
- **FRD vs FLU:** Forward-Right-Down (standard aeronautical body frame used by ArduPilot) vs Forward-Left-Up (standard robotics body frame used by ROS).
- **Tracking State:** ORB-SLAM3's internal health status (`SYSTEM_NOT_READY`, `NOT_INITIALIZED`, `OK`, `RECENTLY_LOST`, `LOST`).
- **Visual Odometry (VO):** Estimating the drone's position and orientation changes by tracking feature movements across camera frames.

**The Task (Do It Yourself):** 
- Step 1: Attach an RGB-D / RealSense camera model to the Iris gimbal inside your Gazebo simulation, and verify that the `/camera/depth` and `/camera/color` image topics are publishing by changing gazebo models in `ardupilot_gz_bringup` package.
- Step 2: Build ORB-SLAM3 core and its ROS 2 wrapper (using working community forks on Ubuntu 24.04), and publish the estimated camera pose to the `/ap/tf` topic.
- Step 3: Go into `libraries/AP_DDS/` in the ArduPilot repository, add debug print statements inside the external navigation handler, and verify that flying forward registers as +X, flying right registers as +Y, and moving downward registers as +Z inside the FC.
- Question for you: If your camera's optical frame (+Z forward, +X right, +Y down) is fed directly into ArduPilot's FRD frame without applying a rotation transform, what will happen when you command the drone to hold altitude?

**Where to find the answers:** 
- ArduPilot Source: `libraries/AP_DDS/` (specifically `ap_dds/external_nav`) in the ArduPilot repository.
- External Navigation Docs: [Vision-Positioning with ROS / DDS](https://ardupilot.org/copter/docs/common-external-position-estimation.html)
- ORB-SLAM3 Documentation: Official ORB-SLAM3 guides covering tracking states and camera models.
- Adding Cameras : [Depth Camera Ardupilot Disucssio Forum](https://discuss.ardupilot.org/t/integrating-depth-camera-with-ardupilot-gazebo-for-obstacle-avoidance/112332/2)
- Discord Mentors: Ping `@RyanF` on the ArduPilot Discord (`#ros`).

