# Ender-5-Plus-Klipper
Klipper firmware for Ender 5 plus running Ender Sprite Direct Drive Extruder and BTT SKR Mini E3. V2

I am uploading this if anyone runs a similar setup and needs the file. I have taken a file similar to my machine and changed the code to work with my setup. 

[include shell_command.cfg]
[include mainsail.cfg]
[include logger.cfg]
# !Ender-5 Plus
# printer_size: 350x350x400
# version: 3.6
# This file contains pin mappings for the Creality Ender 5 Plus.
# Board  BTT SKR Mini E3.
# To use this config, the firmware should be compiled for the AVR
# atmega2560.

# See docs/Config_Reference.md for a description of parameters.

###fluidd set
[virtual_sdcard]
path: /home/nick/printer_data/gcodes
path: ~/printer_data/gcodes
[display_status]
[pause_resume]



[display_status]


[gcode_macro PAUSE]
description: Pause the actual running print
rename_existing: PAUSE_BASE
# change this if you need more or less extrusion
variable_extrude: 1.0
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  ##### set park positon for x and y #####
  # default is your max posion from your printer.cfg
  {% set x_park = printer.toolhead.axis_maximum.x|float - 5.0 %}
  {% set y_park = printer.toolhead.axis_maximum.y|float - 5.0 %}
  ##### calculate save lift position #####
  {% set max_z = printer.toolhead.axis_maximum.z|float %}
  {% set act_z = printer.toolhead.position.z|float %}
  {% if act_z < (max_z - 2.0) %}
      {% set z_safe = 2.0 %}
  {% else %}
      {% set z_safe = max_z - act_z %}
  {% endif %}
  ##### end of definitions #####
  PAUSE_BASE
  G91
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G1 E-{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}
  {% if "xyz" in printer.toolhead.homed_axes %}
    G1 Z{z_safe} F900
    G90
    G1 X{x_park} Y{y_park} F6000
  {% else %}
    {action_respond_info("Printer not homed")}
  {% endif %} 

[gcode_macro RESUME]
description: Resume the actual running print
rename_existing: RESUME_BASE
gcode:
  ##### read E from pause macro #####
  {% set E = printer["gcode_macro PAUSE"].extrude|float %}
  #### get VELOCITY parameter if specified ####
  {% if 'VELOCITY' in params|upper %}
    {% set get_params = ('VELOCITY=' + params.VELOCITY)  %}
  {%else %}
    {% set get_params = "" %}
  {% endif %}
  ##### end of definitions #####
  {% if printer.extruder.can_extrude|lower == 'true' %}
    G91
    G1 E{E} F2100
  {% else %}
    {action_respond_info("Extruder not hot enough")}
  {% endif %}  
  RESUME_BASE {get_params}

[gcode_macro CANCEL_PRINT]
description: Cancel the actual running print
rename_existing: CANCEL_PRINT_BASE
gcode:
  TURN_OFF_HEATERS
  {% if "xyz" in printer.toolhead.homed_axes %}
    G91
    G1 Z4.5 F300
    G90
  {% else %}
    {action_respond_info("Printer not homed")}
  {% endif %}
    G28 X Y
  {% set y_park = printer.toolhead.axis_maximum.y|float - 5.0 %}
    G1 Y{y_park} F2000
    M84
  CANCEL_PRINT_BASE

[stepper_x]
step_pin: PB13
dir_pin: !PB12
enable_pin: !PB14
microsteps: 16
rotation_distance: 40
endstop_pin: ^PC0
position_endstop: 350
position_max: 350
homing_speed: 100

[tmc2209 stepper_x]
uart_pin: PC11
tx_pin: PC10
uart_address: 0
run_current: 0.580
hold_current: 0.500
stealthchop_threshold: 250


[stepper_y]
step_pin: PB10
dir_pin: !PB2
enable_pin: !PB11
microsteps: 16
rotation_distance: 40
endstop_pin: ^PC1
position_endstop: 350
#position_min:-20
position_max: 350
homing_speed: 100

[tmc2209 stepper_y]
uart_pin: PC11
tx_pin: PC10
uart_address: 2
interpolate: True
run_current: 0.580
hold_current: 0.500
stealthchop_threshold: 250


[stepper_z]
step_pin: PB0
dir_pin: !PC5
enable_pin: !PB1
microsteps: 16
rotation_distance: 4
endstop_pin: probe:z_virtual_endstop
position_max: 400
position_min: -4
homing_speed: 10.0

[tmc2209 stepper_z]
uart_pin: PC11
tx_pin: PC10
uart_address: 1
#interpolate: True
run_current: 0.580
hold_current: 0.500
stealthchop_threshold: 5

[bltouch]
sensor_pin: ^PC2
control_pin: PA1
x_offset: -47.5
y_offset: -10
#z_offset: 0
speed: 2.0
samples: 1
sample_retract_dist: 4.0
probe_with_touch_mode: True

[safe_z_home]
home_xy_position: 180, 180
speed: 100
z_hop: 10
z_hop_speed: 5

[bed_mesh]
speed: 150
horizontal_move_z: 8
mesh_min: 0, 0
mesh_max: 302.50, 340
probe_count: 5, 5

[extruder]
step_pin: PB3
dir_pin: !PB4
enable_pin: !PD2
microsteps: 16
gear_ratio: 1:1
rotation_distance: 7.452258965999069
nozzle_diameter: 0.400
filament_diameter: 1.750
heater_pin: PC8
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PA0
#control: pid
#pid_Kp: 21.527
#pid_Ki: 1.063
#pid_Kd: 108.982
min_temp: 0
max_temp: 300

[force_move]
#enable_force_move: True


[idle_timeout]
timeout: 172800

[bed_screws]
screw1:30,40
screw1_name:1
screw2:325,40
screw2_name:2
screw3:325,295
screw3_name:3
screw4:30,295
screw4_name:4

[heater_bed]
heater_pin: PC9
sensor_type: EPCOS 100K B57560G104F
sensor_pin: PC3
#control: pid
#pid_kp = 54.027
#pid_ki = 0.706
#pid_kd = 1536.277
min_temp: 0
max_temp: 130

[heater_fan nozzle_cooling_fan]
pin: PC7

[fan]
pin: PC6

[input_shaper]
shaper_type_x = mzv
shaper_freq_x = 80.8
shaper_type_y = mzv
shaper_freq_y = 77.2


[filament_switch_sensor filament_sensor]
pause_on_runout: true
switch_pin:PE4

[mcu]
serial:(your port here)


[printer]
kinematics: cartesian
max_velocity: 300
max_accel: 7000
#max_accel_to_decel: 7000
max_z_velocity: 5
max_z_accel: 1000
square_corner_velocity: 5.0

[exclude_object]


[gcode_macro G29]	
gcode:
  G28
  bed_mesh_calibrate
  G1 X0 Y0 Z10 F4200

[gcode_arcs]
#resolution: 1.0

[include moonraker_obico_macros.cfg]

# [mcu rpi]
# serial: /tmp/klipper_host_mcu

# [adxl345]
# cs_pin: rpi:None
# spi_speed: 2000000
# spi_bus: spidev2.0

# [resonance_tester]
# accel_chip: adxl345
# accel_per_hz: 70
# probe_points:
#     150,150,10





#[include timelapse.cfg]

#*# <---------------------- SAVE_CONFIG ---------------------->
#*# DO NOT EDIT THIS BLOCK OR BELOW. The contents are auto-generated.
#*#
#*# [bltouch]
#*# z_offset = 3.250
#*#
#*# [bed_mesh default]
#*# version = 1
#*# points =
#*# 	  -0.201250, -0.203750, -0.172500, -0.206250, -0.207500
#*# 	  -0.118750, -0.125000, -0.043750, -0.057500, -0.090000
#*# 	  -0.131250, -0.085000, 0.031250, -0.011250, -0.013750
#*# 	  -0.125000, -0.043750, 0.037500, -0.011250, -0.053750
#*# 	  -0.081250, -0.016250, 0.071250, -0.042500, -0.090000
#*# x_count = 5
#*# y_count = 5
#*# mesh_x_pps = 2
#*# mesh_y_pps = 2
#*# algo = lagrange
#*# tension = 0.2
#*# min_x = 0.0
#*# max_x = 302.48
#*# min_y = 0.0
#*# max_y = 340.0
#*#
#*# [extruder]
#*# control = pid
#*# pid_kp = 28.717
#*# pid_ki = 1.694
#*# pid_kd = 121.687
#*#
#*# [heater_bed]
#*# control = pid
#*# pid_kp = 53.307
#*# pid_ki = 0.480
#*# pid_kd = 1479.264
