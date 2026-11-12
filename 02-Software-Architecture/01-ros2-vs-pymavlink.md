# Title: Software Architecture - ROS 2 vs PyMAVLink, DDS, and GCS Chaos

**The Concept:** 
You have two primary routes to get your companion computer talking to your Flight Controller (FC): the lightweight PyMAVLink path (sending raw MAVLink packets) or the modern full-blown robotics stack with ROS 2 (using the native DDS Micro-ROS bridge or legacy MAVROS). Picking the right setup comes down to your FC's available RAM and how much overhead your autonomy stack can tolerate.

**Keep things in Mind:** 
- The DDS vs MAVROS trap:  To run DDS, your FC needs more than 1MB of RAM/Flash (like STM32H7 chips). If your FC has under 1MB of RAM, the DDS binary isnt recommended by `@Ryanf` .
- Getting overwhelmed by Ground Stations: On Linux, everything technically runs—Mission Planner (running through the Mono runtime), QGroundControl (QGC), MAVProxy, and for quick browser-based setup, *mico config* (an online configurator) felt pretty convenient too.Make a simple rule early: use MAVProxy in SITL for fast scripting and terminal checks, but once you're tuning real hardware on the field, stick to Mission Planner for rock-solid telemetry and parameter handling.
- Flying completely blind on coordinates: When we sent position coordinate commands from our companion computer, we had zero clue what numbers were actually reaching the FC. I realized way too late that you can jump right into ArduPilot's source code, drop custom debug print statements, compile a custom binary, and flash it to watch the raw coordinates land live inside your GCS!
For fuck sake do this early on;

**The Jargon:** 
- **MAVLink:** A lightweight binary telemetry protocol that ArduPilot speaks natively.
- **PyMAVLink:** A Python wrapper library used to create and parse MAVLink messages directly (the lightweight alternative to MAVSDK and the deprecated DroneKit).
- **MAVROS:** A heavy ROS 1/2 node that acts as a translator, turning ROS topics into MAVLink packets over a serial or UDP port.
- **DDS (Micro-ROS Bridge):** Data Distribution Service middleware where ArduPilot natively publishes and subscribes to ROS 2 topics (`/ap/...`) with no translation node in between (strictly requires >1MB RAM FC like Pixhawk 6C/6X).
- **GCS (Ground Control Station):** Software (Mission Planner, QGC, MAVProxy) used to monitor drone telemetry, adjust parameters, and trigger flight modes.

**The Task (Do It Yourself):** 
- Step 1: Fly a basic autonomous navigation pattern in GUIDED mode: arm, take off, and fly a clean 5x5 meter square (first manually by typing coordinate commands into the MAVProxy terminal, then by writing an automated PyMAVLink script).
- Step 2: Clone the ArduPilot repository, navigate to `libraries/AP_DDS/`, and inject custom debug console messages (`gcs().send_text()`) into the incoming external navigation topic handlers.
- Step 3: Compile your custom binary (`./waf configure --board sitl` and `./waf copter`), launch SITL, and verify that the companion computer's raw coordinates are printing out live on your GCS console.
- Question for you: If you're stuck with a budget flight controller running an STM32F4 processor with low RAM, can you use the native DDS Micro-ROS architecture, or are you forced to fall back on MAVROS or PyMAVLink?

**Where to find the answers:** 
- ArduPilot DDS Official Guide: [ROS 2 with ArduPilot DDS](https://ardupilot.org/dev/docs/ros2.html)
- PyMAVLink Documentation: [PyMAVLink GitHub & Docs](https://mavlink.io/en/mavgen_python/)
- Code Location: `libraries/AP_DDS/` in the ArduPilot root directory.
- Discord Mentors: Hit up `@RyanF` on the ArduPilot Discord (`#ros`).
