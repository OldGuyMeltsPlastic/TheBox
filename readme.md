# Custom DoomCube-Trident 355³ AWD Build Log

This repository documents the engineering, CAD optimization, physical wiring architecture, and initial Klipper firmware configuration for a highly modified, heavy-duty, custom-sized **Voron Trident** variant integrated into a **DoomCube** enclosure ecosystem. The design targets a true cubic build volume of **355mm x 355mm x 355mm** with optimized toolhead overtravel for localized sub-systems.

---

## 📐 1. Frame Geometry & Structural Layout

Unlike standard 2020-based Voron builds, this frame scales up structural rigidity significantly by utilizing thick Euro-profile aluminum extrusions for the load-bearing pillars, establishing an asymmetric, inner-flush framework.

*   **Vertical Z-Axis Corner Pillars:** 4420 Euro-Profile Extrusions (40mm x 20mm), **880mm height**.
*   **Horizontal Perimeter Frame (X/Y Crossmembers):** 4420 Euro-Profile Extrusions, **520mm length** (top/bottom perimeters).
*   **Gantry & Bed Support Matrix:** Standard 2020 aluminum profile extrusions.
*   **Mounting Alignment:** All internal 2020 gantry and bed framework extrusions mount strictly **flush with the inside edge** of the 4420 Z-pillars. This layout preserves standard interior kinematic spacing while pushing the extra 20mm profile width outward to function as a rigid exterior chassis.

### 🗺️ Vertical Coordinate Map (Z-Axis Stack)
*   **Z = 0mm to 40mm:** Bottom 4420 horizontal frame profile baseline.
*   **Z = 40mm to 560mm:** Clear mechanical workspace gap (**520mm total height**).
*   **Z = 560mm to 580mm:** Fixed XY Gantry plane (2020 gantry extrusion location).
*   **Z = 580mm to 880mm:** Overhead workspace clearance (**300mm total height**) reserved for toolhead electronics umbilical loops, remote CPAP air hose path loops, and DoomCube Top Hat design clearance.

---

## 🏎️ 2. Kinematics & Motion System

The motion system is built for ultra-high acceleration and high-torque execution to counteract the mass penalty of heavy, all-metal CNC components.

### X/Y CoreXY Axis Configuration
*   **Gantry System:** LDO Monolith CNC Aluminum Gantry Kit.
*   **Drive Configuration:** 4-Motor All-Wheel Drive (AWD).
*   **Motors:** 4x High-Torque LDO-42STH48-2804AH Steppers.
*   **Belts & Pulleys:** True 9mm wide genuine Gates GT2 (2GT) EPDM belts. GT1.5 modifications were avoided to preserve high-tension torque transfer, prevent tooth shear during high-acceleration spikes, and maintain standard Monolith parallel tracking geometries.
*   **Spatial Clearance:** Because the gantry mounts flush to the inner edge of the frame, the AWD rear stepper pods have 100% mechanical clearance without risking any physical collision with the wider 40mm faces of the 4420 Z corner pillars during deep Y-travel moves.

### Z-Axis Configuration
*   **Kinematic Style:** True Trident 3-point kinematic bed leveling suspension.
*   **Z Steppers:** 3x LDO-42STH48-1684CL500E Motors with integrated **500mm Tr8x4 Lead Screws**.
*   **Linear Guides:** 3x 500mm Hiwin MGN9H Linear Rails.
*   **Layout Orientation:** Single Z-axis lead screw and rail centered at the **rear** of the frame (sharing space with the main vertical cable drag chain); dual Z-axis assemblies tucked into the **front-left** and **front-right** corners. This layout preserves a completely wide-open front path for standard DoomCube enclosure doors.
*   **Mounting Method:** CNC Aluminum Trident Z-axis mounts and linear rails are bolted directly into the internal-facing 20mm slot of the 4420 vertical extrusions to maintain strict linear parallelism.

---

## 🛏️ 3. Custom Bed Frame & Thermal Kinematics

To integrate a 4-point perimeter-mounted build plate onto a 3-point Trident moving Z-axis without modifying precision components, a custom structural H-frame was engineered.

*   **Build Plate:** Standard LDO Voron 2.4 355mm x 355mm Mic6 Aluminum Plate (lacks a center-rear mounting hole).
*   **Custom H-Frame Dimensions:** 
    *   *Front Extrusion:* 2020 profile, **520mm length**.
    *   *Rear Extrusion:* 2020 profile, **350mm length**.
    *   *Center Spine Cross-Piece:* 2020 profile, **320mm length**.
    *   *Joint Fasteners:* Blind joints secured via heavy-duty M6 x 16mm bolts.
*   **Kinematic Suspension:** Whopping Pochard / Lightweight Labware Voron 2.4 CNC Kinematic Bed Mounts Kit. 
    *   The long black aluminum cross-yoke plate connects to the rear two corner holes of the 2.4 plate, pivoting on a center dowel pin.
    *   The front two corner holes couple via leaf-spring tabs to magnetic pins on the 520mm front extrusion, eliminating thermal expansion warping ("taco effect").

---

## 🧠 4. Control System & Power Architecture

The brain and power infrastructure utilizes a split-bus strategy to safely mix high-performance high-voltage motion with delicate logic and auxiliary components.

### Hardware Component Matrix
*   **Host Computer:** Raspberry Pi 4B.
*   **Main MCU Controller:** BigTreeTech Octopus Max EZ.
*   **Stepper Drivers:** BTT EZ5160 Pro RGB Drivers (high-voltage external MOSFET versions).
*   **Triple-Bus Power Supply Array:**
    *   *System Logic & Pi:* Mean Well RS-25-5 (5V, 5A output).
    *   *Heaters, Fans, & Z-Axis:* Mean Well UHP-200-24 (24V, 8.4A fanless low-profile supply).
    *   *High-Performance AWD Stepper Rail:* Mean Well UHP-200-55 (55V, 3.6A fanless low-profile supply).

### Physical Wiring Topology & Terminal Isolation
Because the Octopus Max EZ organizes its high-power paths into discrete, isolated block sectors, power lines are routed into three completely separate input terminal clusters on the board:

[Mean Well RS-25-5]  ---> (5V Output)  --->  DCIN Terminal (Logic Bus)
[Mean Well UHP-200-24] ---> (24V Output) --->  MOTOR_M5-M8 Power Input Bank (Z-Axis & Extruder)
[Mean Well UHP-200-55] ---> (55V Output) --->  MOTOR_M1-M4 Power Input Bank (AWD Gantry Only)

*   **⚠️ Unified Grounding Protocol:** The Negative (V-) terminals of all three Mean Well power supplies are wired together with a heavy-gauge jumper wire to create a single, unified reference ground across the entire machine chassis. Leaving the supplies unbonded will drop the tmc5160 reference baseline, causing erratic positioning errors or hardware damage.
*   **Driver Jumper Mapping:** The Octopus Max EZ features pinless driver routing logic; no manual jumper caps are required beneath the EZ5160 drivers to activate SPI communication.
*   **Sensorless Homing DIAG Jumpers:** Jumper caps are inserted onto the physical DIAG header pins adjacent to driver sockets M1 and M3 to route the hardware StallGuard signal directly to the MCU processor.
*   **Voltage Isolation Constraints:** The Z-axis and Extruder stepper motors are strictly isolated on the 24V rail. Running them at 55V offers no performance benefit at low RPMs and introduces extreme thermal heat-soak risk into the compact aluminum toolhead/filament path.

---

## 🛠️ 5. Integrated Sub-Systems & Performance Thresholds

### Toolhead & Nozzle Ecosystem
*   **Hotend:** Chube Conduction High-Flow Hotend (rigid-mounted flat face directly to an aluminum cooling/mounting block).
*   **Extruder:** CNC Aluminum Sherpa Mini driven by a lightweight, high-temperature LDO-42STH20-1004ASH(IG7T) 20mm pancake motor.
*   **Nozzle Geometry:** Bozzle 0.5mm Tungsten Carbide Nozzle. Selected to deliver extreme thermal throughput with structural resistance against abrasive fiber-filled polymers (CF-ASA / GF-Nylon). Sliced extrusions baseline at a `0.55mm` nominal line width.
*   **Part Cooling System:** Remote high-pressure CPAP blower fan delivering compressed air via a flexible hose, stripping heavy cooling fans completely off the moving gantry mass.

### Toolhead & Bed Mesh Sensor
*   **Bed Mesh Sensor:** Cartographer 3D Eddy-Current Sensor.
    *   *Calibration Target:* Sensor coil positioned exactly **3.0mm higher than the nozzle tip**.
    *   *Keep-Out Zone:* Specialized printed or non-metallic mounts required to prevent the LDO Monolith CNC aluminum components from distorting the eddy current magnetic sensor field.

### Performance & Slicer Targeting
Given the 55V rail pushing a 9mm EPDM drive train, the motion boundaries are structurally unconstrained. Real-world speeds are throttled by the volumetric flow properties of the Chube+Bozzle cluster (~50 mm³/s).
*   **Daily Max Print Velocity:** **350 mm/s** for a standard `0.25mm` layer height baseline.
*   **Outer Perimeter Target:** **200 to 220 mm/s** for perfect surface finish and gloss distribution.
*   **Target Acceleration:** **15,000 to 20,000 mm/s²** for regular execution, peaking up to **25,000+ mm/s²** for travel transitions via Klipper Input Shaper tuning.
*   **Belt Tuning Metric:** 9mm EPDM belts are highly tensioned to a target frequency of **140 Hz to 160 Hz** on parallel runs to maximize input shaper structural data collection.

### Off-Plate Purge System
*   **Overtravel Budget:** Oversizing the inner frame to 520mm relative to the 355mm bed yields **165mm of extra physical width**. This enables full nozzle travel completely off the edges of the build volume on all four sides.
*   **Purge Hardware:** *nitjsefni's* Frame-Mounted Trident Purge Bucket & Nozzle Brush System (Remixed from Bloodpack's Decontaminator).
    *   *Material:* Printed entirely in **ASA** (5 walls, 40% Gyroid infill) to withstand extreme enclosure ambient temperatures.
    *   *Modifications:* The printed mounting arm is customized in CAD to adapt from a standard 2020 bracket to the specific spacing of the front-left 4420 extrusion pillar, and scaled vertically to compensate for the specific nozzle height of the Chube Conduction vs a standard Rapido UHF.

---

## 💾 6. Comprehensive Klipper `printer.cfg` Codebase

```ini
#####################################################################
# BTT Octopus Max EZ Split-Voltage Firmware Baseline Configuration
# 55V Bus: Drivers M1-M4 (AWD CoreXY) | 24V Bus: Drivers M5-M8 (Z & Extruder)
#####################################################################

[mcu]
serial: /dev/serial/by-id/usb-Klipper_stm32h723xx_BTT-Octopus-Max-EZ-if00

[printer]
kinematics: corexy
max_velocity: 600
max_accel: 25000
max_z_velocity: 15
max_z_accel: 350
square_corner_velocity: 8.0

#####################################################################
# 🏎️ AWD HIGH-VOLTAGE MOTOR CONFIGURATION (55V BUS)
# Drivers: BTT EZ5160 Pro RGB | Motors: LDO-42STH48-2804AH
#####################################################################

# --- STEPPER X (Front-Left AWD Motor - Slot M1) ---
[stepper_x]
step_pin: PC6
dir_pin: !PA14       # Adjust direction inversion (!) during calibration checkout
enable_pin: !PC7
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200
endstop_pin: tmc5160_stepper_x:virtual_endstop
position_min: -40    # Generous negative travel headroom to swing out over the front-left corner bucket
position_endstop: 0
position_max: 360    # Configured for 520mm frame overtravel clearance

[tmc5160 stepper_x]
cs_pin: PD12
spi_software_sclk_pin: PE12
spi_software_mosi_pin: PE14
spi_software_miso_pin: PE13
sense_resistor: 0.050        # ⚠️ CRITICAL CORRECTED VALUE FOR BTT RGB PRO DRIVERS
run_current: 1.75
interpolate: False
stealthchop_threshold: 0     # SpreadCycle forced for maximum tracking torque
driver_SGT: 1                # StallGuard threshold for sensorless homing

# --- STEPPER X1 (Rear-Right AWD Motor - Slot M2) ---
[stepper_x1]
step_pin: PD3
dir_pin: !PD2
enable_pin: !PD4
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200

[tmc5160 stepper_x1]
cs_pin: PD5
spi_software_sclk_pin: PE12
spi_software_mosi_pin: PE14
spi_software_miso_pin: PE13
sense_resistor: 0.050
run_current: 1.75
interpolate: False
stealthchop_threshold: 0

# --- STEPPER Y (Front-Right AWD Motor - Slot M3) ---
[stepper_y]
step_pin: PB3
dir_pin: !PB2
enable_pin: !PA5
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200
endstop_pin: tmc5160_stepper_y:virtual_endstop
position_min: 0
position_endstop: 360
position_max: 360

[tmc5160 stepper_y]
cs_pin: PA6
spi_software_sclk_pin: PE12
spi_software_mosi_pin: PE14
spi_software_miso_pin: PE13
sense_resistor: 0.050
run_current: 1.75
interpolate: False
stealthchop_threshold: 0
driver_SGT: 1

# --- STEPPER Y1 (Rear-Left AWD Motor - Slot M4) ---
[stepper_y1]
step_pin: PD11
dir_pin: !PD10
enable_pin: !PD12
rotation_distance: 40
microsteps: 16
full_steps_per_rotation: 200

[tmc5160 stepper_y1]
cs_pin: PD13
spi_software_sclk_pin: PE12
spi_software_mosi_pin: PE14
spi_software_miso_pin: PE13
sense_resistor: 0.050
run_current: 1.75
interpolate: False
stealthchop_threshold: 0


#####################################################################
# 🛏️ TRIDENT Z-AXIS MOTOR CONFIGURATION (24V BUS)
# Drivers: Standard TMC5160 Stepsticks | Motors: LDO-1684CL500E
#####################################################################

# --- STEPPER Z (Front-Left Z Motor - Slot M5) ---
[stepper_z]
step_pin: PA10
dir_pin: PA9
enable_pin: !PA13
rotation_distance: 4
microsteps: 16
endstop_pin: probe:z_virtual_endstop # Pinless configuration utilizing Cartographer Touch
position_max: 355
position_min: -5

[tmc5160 stepper_z]
cs_pin: PA11
sense_resistor: 0.075 # Set to match standard stepstick parameters used for Z
run_current: 1.00
interpolate: False
stealthchop_threshold: 0

# --- STEPPER Z1 (Rear Center Z Motor - Slot M6) ---
[stepper_z1]
step_pin: PC9
dir_pin: PC8
enable_pin: !PC10
rotation_distance: 4
microsteps: 16

[tmc5160 stepper_z1]
cs_pin: PA8
sense_resistor: 0.075
run_current: 1.00
interpolate: False
stealthchop_threshold: 0

# --- STEPPER Z2 (Front-Right Z Motor - Slot M7) ---
[stepper_z2]
step_pin: PG7
dir_pin: PG6
enable_pin: !PC11
rotation_distance: 4
microsteps: 16

[tmc5160 stepper_z2]
cs_pin: PG8
sense_resistor: 0.075
run_current: 1.00
interpolate: False
stealthchop_threshold: 0


#####################################################################
# 🪶 EXTRUDER CONFIGURATION (24V BUS)
# Extruder: CNC Sherpa Mini | Motor: LDO-42STH20-1004ASH(IG7T)
# Driver: BTT EZ5160 Pro RGB (Slot M8)
#####################################################################

[extruder]
step_pin: PE6
dir_pin: !PA15          # Sherpa Mini typically requires mechanical direction inversion
enable_pin: !PE1
rotation_distance: 22.67895
gear_ratio: 50:10       # 50:10 internal drive cluster teeth count
microsteps: 16
full_steps_per_rotation: 200
nozzle_diameter: 0.500  # Updated for Bozzle 0.5mm configuration
filament_diameter: 1.750
max_extrude_only_distance: 1400.0
pressure_advance: 0.025 # Baseline starting value for Sherpa + Bozzle cluster

# --- Chube Conduction High-Temp Sensor ---
heater_pin: PA2
sensor_type: PT1000     # Native Chube precision high-temperature sensor RTD element
sensor_pin: PF4         # Tied directly to standard Octopus Analog Input channel
pullup_resistor: 4700
min_temp: 0
max_temp: 350

[tmc5160 extruder]
cs_pin: PC13
spi_software_sclk_pin: PE12
spi_software_mosi_pin: PE14
spi_software_miso_pin: PE13
sense_resistor: 0.050   # ⚠️ CRITICAL CORRECTED VALUE FOR BTT RGB PRO DRIVERS
run_current: 0.70       # Maintained at 70% thermal headroom ceiling to prevent heat creep
interpolate: False
stealthchop_threshold: 0


#####################################################################
# 🌬️ PART COOLING CPAP REMOTE BLOWER SYSTEM & SENSORS
#####################################################################

[fan]
pin: PE5                # Tied to high-current hardware PWM Fan Controller pin
hardware_pwm: True
cycle_time: 0.0001      # Specialized cycle time for smooth ESC pulse modulation
kick_start_time: 0.100

[cartographer]
serial: /dev/serial/by-id/usb-Cartographer_3d_sensor_here
speed: 40.
#   Z probing speed in mm/s
lift_speed: 5.
#   Z lifting speed after probing in mm/s
backlash_comp: 0.005
#   Backlash compensation value
x_offset: 0.0
#   X offset of the cartographer sensor relative to the nozzle
y_offset: 25.0
#   Y offset of the cartographer sensor relative to the nozzle
trigger_distance: 3.0
#   Cartographer trigger distance threshold
trigger_curve: default

[bed_mesh]
speed: 300
horizontal_move_z: 5
mesh_min: 25, 25
mesh_max: 330, 330
probe_count: 15, 15
algorithm: bicubic
```

🚀 7. Core Automated Calibration & Print Start MacrosAdd these custom automation macros directly to your macros.cfg file or append them to the end of printer.cfg. They are scaled explicitly to handle your large frame parameters, manage Cartographer Touch homing logic, execute the localized context mesh over your printed parts, and execute the front-left off-bed purge sequence.```ini#####################################################################🦾 SYSTEM PRINT CONTROL WORKFLOWS#####################################################################[gcode_macro START_PRINT]gcode:# --- Establish Configuration Parameters ---{% set BED_TEMP = params.BED|default(100)|float %}{% set EXTRUDER_TEMP = params.EXTRUDER|default(240)|float %}# --- Begin Safety Warm-Up Profile ---M117 Heating Bed to {BED_TEMP}C...M140 S{BED_TEMP}             # Begin heating bed plateG90                          # Use absolute coordinatesM104 S150                    # Preheat extruder nozzle safely below ooze thresholds (150C)G28                          # Initial global homing sequence# --- Thermal Saturation Soak ---M190 S{BED_TEMP}             # Wait for Mic6 bed to fully stabilize at heat targetM117 Executing Z-Tilt Alignment...Z_TILT_ADJUST                # Perform 3-point independent Z-axis tramming profileG28 Z                        # Re-home absolute Z axis center line# --- Localized Adaptive Mesh ---M117 Generating Cartographer Mesh...BED_MESH_CALIBRATE ADAPTIVE=1 METHOD=rapid_scan # Scans only the model's actual print layout space# --- Final Melt Execution ---G0 X-30 Y10 Z30 F18000       # Move out to the front-left overtravel safety zoneM117 Finalizing Hotend Temp...M109 S{EXTRUDER_TEMP}        # Heat Chube Conduction to functional melt point# --- Purge and Scrub Run ---CLEAN_NOZZLE_AND_PURGE       # Execute cleaning macro sequence before dropping down to the bedM117 Printing...[gcode_macro CLEAN_NOZZLE_AND_PURGE]gcode:SAVE_GCODE_STATE NAME=clean_nozzle_stateG90M117 Purging Filament...# --- Position over the nitjsefni Left Purge Bucket ---G0 X-35 Y15 Z15 F24000       # Align nozzle tip directly inside the custom ASA arm receptacleG92 E0                       # Reset extrusion metricG1 E25 F150                  # Extrude a clean 25mm purge blob to fill the melt chamberG4 P2000                     # Dwell 2 seconds to allow internal stringing pressure to clearG92 E0G1 E-1.5 F1500               # Small retract to prevent nozzle tip stringing over the frame# --- High-Acceleration Nozzle Scrub Sequence ---M117 Scrubbing Nozzle Tip...# Rapidly sweep back and forth across the front-left silicone/brass brush assembly{% for i in range(6) %}G0 X-40 Y30 F36000G0 X-20 Y30 F36000{% endfor %}# --- Exit to Print Matrix ---G0 Z5 F3000                  # Lift clear of the bucket rim wallsRESTORE_GCODE_STATE NAME=clean_nozzle_state[gcode_macro END_PRINT]gcode:# --- Safety Engine Shutdown ---M140 S0                      # Turn off heated bedM104 S0                      # Turn off Chube hotend heaterM107                         # Kill remote CPAP part cooling blower# --- Retract and Clear Space ---G91                          # Relative coordinatesG1 E-5 F300                  # Retract remaining molten filament safely away from the cold-zoneG1 Z10 F3000                 # Instantly drop the bed 10mm down to prevent nozzle scarringG90                          # Absolute coordinatesG0 X5 Y350 F18000            # Park the gantry at the rear of the frame out of the wayM117 Print Complete.M84                          # Disable all motor current lines (prevents frame holding heat)```

⚠️ 8. Critical CAD Constraints & Assembly Verification ChecklistZ-Axis Top Clearance: When the bed frame drives to maximum Z-height (nozzle touching the bed surface), ensure the top tips of the unconstrained 500mm lead screws (ending at ~Z=540mm) safely clear the underside of the moving Monolith AWD gantry blocks.DoomCube Panel Offsets: Because the 4420 frame steps outward by an extra 20mm relative to standard Voron 2020 frame profiles, the outer DoomCube panel corner brackets and hinge blocks must be customized in CAD to accommodate the extra 40mm mounting face.Klipper Cartesian Boundaries: Do not copy-paste standard 350mm printer.cfg travel limits. Custom macro bounds for clean_nozzle and MOVE_TO_BUCKET must be calibrated manually to prevent high-torque AWD steppers from over-traveling into the 520mm framework limits.
