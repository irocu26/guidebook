# Title: Hardware Fails - Smoke, Brownouts, and Ground Reality

**The Concept:** 
Everything looks clean and effortless in simulation, but physical hardware is governed by brutal real-world constraints. Clean power distribution, motor shaft mechanics, electrical protection, and proper companion computer power budgeting are the only things standing between a flight-ready drone and an expensive pile of charred ashes on your lab bench.

**How We Messed Up:** 
- **NO SMOKE STOPPER, FRIED OUR FC:** The very second we plugged in our LiPo battery, thick white smoke poured out of the Flight Controller! Whether it was a short circuit or reverse polarity, we had no idea because the board was already destroyed. If we had spent ten bucks on a smoke stopper, the circuit would have tripped and saved our FC. Make this rule number one: never connect a battery to freshly soldered hardware without a smoke stopper.
- **Powering the Pi through GPIO (Fried it):** Raspberry Pi GPIO header pins have zero reverse-polarity or over-voltage protection. We fed 5V directly into a GPIO pin, and the Pi fried on the spot! Never power your companion computer through GPIO; always use a dedicated high-current UBEC. And if you're using a Raspberry Pi 5, there are specific config commands you need to run to handle USB power negotiation (`usb_max_current_enable=1`) so it runs at full throttle without throttling your peripherals.
- **Companion Computer Brownouts:** Every time we tried to take off, our companion computer randomly shut down and rebooted! We were using a cheap, thin USB power cable that couldn't handle the current draw. When the motors throttled up, the main bus voltage dipped, and the board browned out. Use thick, high-gauge wiring and do proper power budgeting.
- **Cheap Depth Camera Cables:** We tried running our RealSense depth camera using a generic cheap USB cable, which resulted in constant disconnects, dropped frames, and system crashes. Depth cameras strictly require expensive, high-grade, shielded USB 3.0 cables.
- **Cheap Plastic Frames vs Real Carbon Fiber:** We bought cheap knock-off frames that looked like carbon fiber but shattered on our very first minor crash. Invest in real, rigid carbon fiber frames. Also, 3D print 2 or 3 spare copies of every single mount, bracket, and landing leg in advance so a broken piece doesn't kill your entire testing day.
- **Short Motor Shafts and 1045 Props:** We ordered Holybro 2216 motors, but their shafts were too short for standard prop nuts. Regular props wouldn't fit or clamp down securely! For short-shaft motors like that, you need DJI-style self-locking props.
- **The 3 CCW + 1 CW Blunder:** Due to some complication we  ended up with three Counter-Clockwise (CCW) motors and only one Clockwise (CW) motor. A quadcopter literally cannot fly like that. Always make sure you have two matched CW and CCW pairs. (Its very hard for mechanically configured CW motor to use it as CCW mtoors)
- **Takeoff Disasters:** 
    Our drone flipped violently the moment we took off because our prop rotation directions were backward. 
    
    On another test, it drifted aggressively sideways on liftoff because the RC sticks weren't calibrated.

    If drone doesnt behave well on takeoff in any mode first check with those two issues and then check with esc replacement 


- **Blindly Swapping ESCs:** We looked at low frame rate video from a phone and decided an ESC was dead, buying a replacement without diagnosing the problem. Always stick to the sequence: `RC Calibration -> Motor rotation check -> ESC calibration -> only replace the ESC if it still fails`.

- **Benchmarking Failure :** We never did proper performance benchmarking. We blindly guessed on vibes and gut feeling, and bought the wrong companion board! We didn't know whether our tasks needed a 16GB Raspberry Pi or a Jetson Orin Nano. Don't make our mistake: run your ROS 2 nodes and SLAM on your laptop first, profile the exact CPU and RAM loads, learn proper benchmarking methods, and buy the right companion computer based on data.

- **Soldering and Mechanical Teammates:** Clean up your soldering technique early, keep connections modular, and keep an organized tackle box with spare JST-to-DuPont wires, extra screw kits, and zip bags. Find repair shops near you for expensive gear. And seriously: recruit two **mechanical** engineering teammates.

**The Jargon:** 
- **Smoke Stopper:** A resettable electronic fuse placed between the battery and the drone that instantly cuts power if it detects a short circuit.
- **UBEC (Universal Battery Elimination Circuit):** A dedicated DC-DC step-down buck regulator that converts high battery voltage (14.8V - 25.2V) into a clean, regulated 5V DC high-amp output.
- **Brownout:** A sudden microprocessor reset caused by a temporary voltage dip during high motor current draws.
- **Self-Locking Props:** Threaded propellers that naturally tighten themselves in the opposite direction of motor rotation (mandatory for short-shaft motors).
- **ESC Calibration:** Teaching your Electronic Speed Controllers the exact minimum (1000us) and maximum (2000us) PWM throttle endpoints from your autopilot.

**The Task (Do It Yourself):** 
- Step 1: Create a full Power Budgeting spreadsheet: peak motor current draw + ESCs + Flight Controller + Companion Computer (minimum 5V @ 3A-5A) + Depth Camera + Lidar. Check that your battery C-rating and UBEC can easily supply peak demand.
- Step 2: buy a smoke stopper, and always test continuity and resistance across the main XT60 battery pads using a multimeter before plugging in.
- Step 3: TAKE THE PROPS OFF (non-negotiable safety rule) and run this exact sequence: RC stick calibration -> motor rotation check (2 CW, 2 CCW) -> ESC endpoint calibration.

- Question for you: If your companion computer runs perfectly on your lab bench with a single camera, but immediately reboots and crashes the depth camera the exact second all four motors spin up to full throttle, is it a software bug or an electrical brownout? How would you verify this using a multimeter?

**Where to find the answers:** 
- ArduPilot ESC Calibration Guide: [ArduPilot ESC Calibration](https://ardupilot.org/copter/docs/esc-calibration.html)
- Raspberry Pi 5 Power Configuration: Official Raspberry Pi documentation for USB current limit override (`usb_max_current_enable=1`).
- Hardware Community: ArduPilot Discord `#hardware` channel.
