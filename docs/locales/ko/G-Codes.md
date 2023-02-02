# G-Codes

이 문서는 Klipper가 지원하는 명령에 대해 설명합니다. OctoPrint Terminal 탭에 입력할 수 있는 명령입니다.

## G-Code 명령

Klipper는 다음과 같은 표준 G-Code 명령을 지원합니다:

- 이동 (G0 또는 G1): `G1 [X<위치>] [Y<위치>] [Z<위치>] [E<위치>] [F<속도>]`
- 지연(멈춤): `G4 P<밀리세컨드>`
- 원점으로 복귀: `G28 [X] [Y] [Z]`
- 모터 끄기: `M18` 또는 `M84`
- 현재 동작이 완료될 때까지 대기: `M400`
- 익스트루더 사출에 절대/상대 거리 사용: `M82`, `M83`
- 절대/상대 좌표 사용: `G90`, `G91`
- Set position: `G92 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>]`
- Set speed factor override percentage: `M220 S<백분율>`
- Set extrude factor override percentage: `M221 S<백분율>`
- 가속도 설정: `M204 S<값>` 또는 `M204 P<값> T<값>`
   - 참고: S를 지정하지 않고 P와 T를 모두 지정하면 가속도는 P와 T의 최소값으로 설정됩니다. P 또는 T 중 하나만 지정하면 명령이 적용되지 않습니다.
- 익스트루더(베드) 온도 가져오기: `M105`
- 익스트루더 온도 설정: `M104 [T<인덱스>] [S<온도>]`
- 익스트루더 온도 설정될때 까지 기다리기: `M109 [T<인덱스>] S<온도>`
   - 참고: M109는 항상 온도가 요청된 값으로 안정될 때까지 기다립니다.
- 베드 온도 설정: `M140 [S<온도>]`
- 베드 온도 설정될때 까지 기다리기: `M190 S<온도>`
   - 참고: M190는 항상 온도가 요청된 값으로 안정될 때까지 기다립니다.
- 팬 속도 설정: `M106 S<값>`
- 팬 끄기: `M107`
- 비상 정지: `M112`
- 현재 위치 가져오기: `M114`
- 펌웨어 버전 가져오기: `M115`

위 명령어에 대한 자세한 내용은 [RepRap G-Code documentation](http://reprap.org/wiki/G-code) 를 참조하십시오.

Klipper's goal is to support the G-Code commands produced by common 3rd party software (eg, OctoPrint, Printrun, Slic3r, Cura, etc.) in their standard configurations. It is not a goal to support every possible G-Code command. Instead, Klipper prefers human readable ["extended G-Code commands"](#additional-commands). Similarly, the G-Code terminal output is only intended to be human readable - see the [API Server document](API_Server.md) if controlling Klipper from external software.

일반적인 G-Code 명령이 아닌 사용자 지정 [gcode_macro config section](Config_Reference.md#gcode_macro) 으로도 구현할 수 있습니다. 예를 들어, `G12`, `G29`, `G30`, `G31`, `M42`, `M80`, `M81`, `T1` 등을 구현하는 데 사용할 수 있습니다.

## Additional Commands

Klipper는 일반 구성 및 상태에 대해 "확장된" G-Code 명령을 사용합니다. 이러한 확장 명령은 모두 유사한 형식을 따릅니다. 명령 이름으로 시작하고 하나 이상의 매개변수가 뒤따를 수 있습니다. 예제: `SET_SERVO SERVO=myservo ANGLE=5.3`. 이 문서에서 명령과 매개변수는 대문자로 표시되지만 대소문자를 구분하지 않습니다. (따라서 "SET_SERVO"와 "set_servo"는 모두 동일한 명령을 실행합니다.)

This section is organized by Klipper module name, which generally follows the section names specified in the [printer configuration file](Config_Reference.md). Note that some modules are automatically loaded.

### [adxl345]

The following commands are available when an [adxl345 config section](Config_Reference.md#adxl345) is enabled.

#### ACCELEROMETER_MEASURE

`ACCELEROMETER_MEASURE [CHIP=<config_name>] [NAME=<value>]`: Starts accelerometer measurements at the requested number of samples per second. If CHIP is not specified it defaults to "adxl345". The command works in a start-stop mode: when executed for the first time, it starts the measurements, next execution stops them. The results of measurements are written to a file named `/tmp/adxl345-<chip>-<name>.csv` where `<chip>` is the name of the accelerometer chip (`my_chip_name` from `[adxl345 my_chip_name]`) and `<name>` is the optional NAME parameter. If NAME is not specified it defaults to the current time in "YYYYMMDD_HHMMSS" format. If the accelerometer does not have a name in its config section (simply `[adxl345]`) then `<chip>` part of the name is not generated.

#### ACCELEROMETER_QUERY

`ACCELEROMETER_QUERY [CHIP=<config_name>] [RATE=<value>]`: queries accelerometer for the current value. If CHIP is not specified it defaults to "adxl345". If RATE is not specified, the default value is used. This command is useful to test the connection to the ADXL345 accelerometer: one of the returned values should be a free-fall acceleration (+/- some noise of the chip).

#### ACCELEROMETER_DEBUG_READ

`ACCELEROMETER_DEBUG_READ [CHIP=<config_name>] REG=<register>`: queries ADXL345 register "register" (e.g. 44 or 0x2C). Can be useful for debugging purposes.

#### ACCELEROMETER_DEBUG_WRITE

`ACCELEROMETER_DEBUG_WRITE [CHIP=<config_name>] REG=<register> VAL=<value>`: Writes raw "value" into a register "register". Both "value" and "register" can be a decimal or a hexadecimal integer. Use with care, and refer to ADXL345 data sheet for the reference.

### [angle]

The following commands are available when an [angle config section](Config_Reference.md#angle) is enabled.

#### ANGLE_CALIBRATE

`ANGLE_CALIBRATE CHIP=<chip_name>`: Perform angle calibration on the given sensor (there must be an `[angle chip_name]` config section that has specified a `stepper` parameter). IMPORTANT - this tool will command the stepper motor to move without checking the normal kinematic boundary limits. Ideally the motor should be disconnected from any printer carriage before performing calibration. If the stepper can not be disconnected from the printer, make sure the carriage is near the center of its rail before starting calibration. (The stepper motor may move forwards or backwards two full rotations during this test.) After completing this test use the `SAVE_CONFIG` command to save the calibration data to the config file. In order to use this tool the Python "numpy" package must be installed (see the [measuring resonance document](Measuring_Resonances.md#software-installation) for more information).

#### ANGLE_DEBUG_READ

`ANGLE_DEBUG_READ CHIP=<config_name> REG=<register>`: Queries sensor register "register" (e.g. 44 or 0x2C). Can be useful for debugging purposes. This is only available for tle5012b chips.

#### ANGLE_DEBUG_WRITE

`ANGLE_DEBUG_WRITE CHIP=<config_name> REG=<register> VAL=<value>`: Writes raw "value" into register "register". Both "value" and "register" can be a decimal or a hexadecimal integer. Use with care, and refer to sensor data sheet for the reference. This is only available for tle5012b chips.

### [bed_mesh]

The following commands are available when the [bed_mesh config section](Config_Reference.md#bed_mesh) is enabled (also see the [bed mesh guide](Bed_Mesh.md)).

#### BED_MESH_CALIBRATE

`BED_MESH_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>] [<mesh_parameter>=<value>]`: 이 명령은 구성의 매개변수로 지정된 생성 지점을 사용하여 침대를 조사합니다. 프로빙 후 mesh 가 생성되고 메쉬에 따라 z-움직임이 조정됩니다. 선택적 프로브 매개변수에 대한 자세한 내용은 PROBE 명령을 참조하십시오. METHOD=manual이 지정되면 수동 프로빙 도구가 활성화됩니다. 이 도구가 활성화되어 있는 동안 사용할 수 있는 추가 명령에 대한 자세한 내용은 위의 MANUAL_PROBE 명령을 참조하십시오.

#### BED_MESH_OUTPUT

`BED_MESH_OUTPUT PGP=[<0:1>]`: This command outputs the current probed z values and current mesh values to the terminal. If PGP=1 is specified the X, Y coordinates generated by bed_mesh, along with their associated indices, will be output to the terminal.

#### BED_MESH_MAP

`BED_MESH_MAP`: BED_MESH_OUTPUT과 마찬가지로 이 명령은 메쉬의 현재 상태를 터미널에 출력합니다. 사람이 읽을 수 있는 형식으로 값을 인쇄하는 대신 상태가 json 형식으로 직렬화됩니다. 이를 통해 octoprint 플러그인은 데이터를 쉽게 캡처하고 침대 표면과 유사한 높이 맵을 생성할 수 있습니다.

#### BED_MESH_CLEAR

`BED_MESH_CLEAR`: 이 명령은 mesh를 지우고 모든 z 조정을 제거합니다. 이것을 end-gcode에 넣는 것이 좋습니다.

#### BED_MESH_PROFILE

`BED_MESH_PROFILE LOAD=<name> SAVE=<name> REMOVE=<name>`: This command provides profile management for mesh state. LOAD will restore the mesh state from the profile matching the supplied name. SAVE will save the current mesh state to a profile matching the supplied name. Remove will delete the profile matching the supplied name from persistent memory. Note that after SAVE or REMOVE operations have been run the SAVE_CONFIG gcode must be run to make the changes to persistent memory permanent.

#### BED_MESH_OFFSET

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`: 메쉬 조회에 X 및/또는 Y 오프셋을 적용합니다. 이것은 툴 교체 후 정확한 Z 조정을 생성하기 위해 오프셋이 필요하기 때문에 독립 익스트루더가 있는 프린터에 유용합니다.

### [bed_screws]

The following commands are available when the [bed_screws config section](Config_Reference.md#bed_screws) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws)).

#### BED_SCREWS_ADJUST

`BED_SCREWS_ADJUST`: 이 명령은 BED 나사 조정 도구를 호출합니다. 노즐을 다른 위치 (config 파일에 정의된 대로) 로 명령하고 BED 가 노즐에서 일정한 거리가 되도록 BED 나사를 조정할 수 있습니다

### [bed_tilt]

The following commands are available when the [bed_tilt config section](Config_Reference.md#bed_tilt) is enabled.

#### BED_TILT_CALIBRATE

`BED_TILT_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>]`: 이 명령은 구성에 지정된 지점을 조사한 다음 업데이트된 x 및 y 기울기 조정을 권장합니다. 선택적 프로브 매개변수에 대한 자세한 내용은 PROBE 명령을 참조하십시오. METHOD=manual이 지정되면 수동 프로빙 도구가 활성화됩니다. 이 도구가 활성화되어 있는 동안 사용할 수 있는 추가 명령에 대한 자세한 내용은 위의 MANUAL_PROBE 명령을 참조하십시오.

### [bltouch]

The following command is available when a [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [BL-Touch guide](BLTouch.md)).

#### BLTOUCH_DEBUG

`BLTOUCH_DEBUG COMMAND=<command>`: This sends a command to the BLTouch. It may be useful for debugging. Available commands are: `pin_down`, `touch_mode`, `pin_up`, `self_test`, `reset`. A BL-Touch V3.0 or V3.1 may also support `set_5V_output_mode`, `set_OD_output_mode`, `output_mode_store` commands.

#### BLTOUCH_STORE

`BLTOUCH_STORE MODE=<output_mode>`: 이것은 BLTouch V3.1의 EEPROM 에 출력 모드를 저장합니다. 사용 가능한 출력 모드는 '5V', 'OD'입니다.

### [configfile]

The configfile module is automatically loaded.

#### SAVE_CONFIG

`SAVE_CONFIG`: 이 명령은 기본 프린터 config 파일을 덮어쓰고 호스트 소프트웨어를 다시 시작합니다. 이 명령은 캘리브레이션 테스트 결과를 저장하기 위해 다른 캘리브레이션 명령과 함께 사용됩니다.

### [delayed_gcode]

The following command is enabled if a [delayed_gcode config section](Config_Reference.md#delayed_gcode) has been enabled (also see the [template guide](Command_Templates.md#delayed-gcodes)).

#### UPDATE_DELAYED_GCODE

`UPDATE_DELAYED_GCODE [ID=<name>] [DURATION=<seconds>]`: 식별된 [delayed_gcode]의 지연 시간을 업데이트하고 gcode 실행을 위한 타이머를 시작합니다. 값이 0이면 보류 중인 지연된 gcode 실행이 취소됩니다.

### [delta_calibrate]

The following commands are available when the [delta_calibrate config section](Config_Reference.md#linear-delta-kinematics) is enabled (also see the [delta calibrate guide](Delta_Calibrate.md)).

#### DELTA_CALIBRATE

`DELTA_CALIBRATE [METHOD=manual] [<probe_parameter>=<value>]`: 이 명령은 BED 의 7개 지점을 조사하고 업데이트된 엔드스톱 위치, 타워 각도 및 반경을 권장합니다. 선택적 프로브 매개변수에 대한 자세한 내용은 PROBE 명령을 참조하십시오. METHOD=manual이 지정되면 수동 프로빙 도구가 활성화됩니다. 이 도구가 활성화되어 있는 동안 사용할 수 있는 추가 명령에 대한 자세한 내용은 위의 MANUAL_PROBE 명령을 참조하십시오.

#### DELTA_ANALYZE

`DELTA_ANALYZE`: 이 명령은 향상된 델타 보정 중에 사용됩니다. 자세한 내용은 [Delta Calibrate](Delta_Calibrate.md)를 참조하십시오.

### [display]

The following command is available when a [display config section](Config_Reference.md#gcode_macro) is enabled.

#### SET_DISPLAY_GROUP

`SET_DISPLAY_GROUP [DISPLAY=<display>] GROUP=<group>`: LCD 디스플레이의 활성 디스플레이 그룹을 설정합니다. 이를 통해 구성에서 여러 디스플레이 데이터 그룹을 정의할 수 있습니다, 예를 들어 `[display_data <group> <elementname>]`을 선택하고 이 확장된 gcode 명령을 사용하여 둘 사이를 전환합니다. DISPLAY가 지정되지 않은 경우 기본값은 "display" (기본 디스플레이)입니다.

### [display_status]

The display_status module is automatically loaded if a [display config section](Config_Reference.md#display) is enabled. It provides the following standard G-Code commands:

- 메시지 표시: `M117 <message>`
- 출력 상태(%) 설정: `M73 P<percent>`

Also provided is the following extended G-Code command:

- `SET_DISPLAY_TEXT MSG=<message>`: Performs the equivalent of M117, setting the supplied `MSG` as the current display message. If `MSG` is omitted the display will be cleared.

### [dual_carriage]

The following command is available when the [dual_carriage config section](Config_Reference.md#dual_carriage) is enabled.

#### SET_DUAL_CARRIAGE

`SET_DUAL_CARRIAGE CARRIAGE=[0|1]`: 이 명령은 활성 캐리지를 설정합니다. 일반적으로 다중 압출기 구성의 activate_gcode 및 deactivate_gcode 필드에서 호출됩니다.

### [endstop_phase]

The following commands are available when an [endstop_phase config section](Config_Reference.md#endstop_phase) is enabled (also see the [endstop phase guide](Endstop_Phase.md)).

#### ENDSTOP_PHASE_CALIBRATE

`ENDSTOP_PHASE_CALIBRATE [STEPPER=<config_name>]`: STEPPER 매개변수가 제공되지 않으면 이 명령은 과거 원점 복귀 작업 동안 엔드스톱 스테퍼 단계에 대한 통계를 보고합니다. STEPPER 매개변수가 제공되면 지정된 endstop 단계 설정이 SAVE_CONFIG 명령과 함께 구성 파일에 기록되도록 정렬합니다.

### [exclude_object]

The following commands are available when an [exclude_object config section](Config_Reference.md#exclude_object) is enabled (also see the [exclude object guide](Exclude_Object.md)):

#### `EXCLUDE_OBJECT`

`EXCLUDE_OBJECT [NAME=object_name] [CURRENT=1] [RESET=1]`: With no parameters, this will return a list of all currently excluded objects.

When the `NAME` parameter is given, the named object will be excluded from printing.

When the `CURRENT` parameter is given, the current object will be excluded from printing.

When the `RESET` parameter is given, the list of excluded objects will be cleared. Additionally including `NAME` will only reset the named object. This **can** cause print failures, if layers were already skipped.

#### `EXCLUDE_OBJECT_DEFINE`

`EXCLUDE_OBJECT_DEFINE [NAME=object_name [CENTER=X,Y] [POLYGON=[[x,y],...]] [RESET=1] [JSON=1]`: Provides a summary of an object in the file.

With no parameters provided, this will list the defined objects known to Klipper. Returns a list of strings, unless the `JSON` parameter is given, when it will return object details in json format.

When the `NAME` parameter is included, this defines an object to be excluded.

- `NAME`: This parameter is required. It is the identifier used by other commands in this module.
- `CENTER`: An X,Y coordinate for the object.
- `POLYGON`: An array of X,Y coordinates that provide an outline for the object.

When the `RESET` parameter is provided, all defined objects will be cleared, and the `[exclude_object]` module will be reset.

#### `EXCLUDE_OBJECT_START`

`EXCLUDE_OBJECT_START NAME=object_name`: This command takes a `NAME` parameter and denotes the start of the gcode for an object on the current layer.

#### `EXCLUDE_OBJECT_END`

`EXCLUDE_OBJECT_END [NAME=object_name]`: Denotes the end of the object's gcode for the layer. It is paired with `EXCLUDE_OBJECT_START`. A `NAME` parameter is optional, and will only warn when the provided name does not match the current object.

### [extruder]

The following commands are available if an [extruder config section](Config_Reference.md#extruder) is enabled:

#### ACTIVATE_EXTRUDER

`ACTIVATE_EXTRUDER EXTRUDER=<config_name>`: In a printer with multiple [extruder](Config_Reference.md#extruder) config sections, this command changes the active hotend.

#### SET_PRESSURE_ADVANCE

`SET_PRESSURE_ADVANCE [EXTRUDER=<config_name>] [ADVANCE=<pressure_advance>] [SMOOTH_TIME=<pressure_advance_smooth_time>]`: Set pressure advance parameters of an extruder stepper (as defined in an [extruder](Config_Reference.md#extruder) or [extruder_stepper](Config_Reference.md#extruder_stepper) config section). If EXTRUDER is not specified, it defaults to the stepper defined in the active hotend.

#### SET_EXTRUDER_ROTATION_DISTANCE

`SET_EXTRUDER_ROTATION_DISTANCE EXTRUDER=<config_name> [DISTANCE=<distance>]`: Set a new value for the provided extruder stepper's "rotation distance" (as defined in an [extruder](Config_Reference.md#extruder) or [extruder_stepper](Config_Reference.md#extruder_stepper) config section). If the rotation distance is a negative number then the stepper motion will be inverted (relative to the stepper direction specified in the config file). Changed settings are not retained on Klipper reset. Use with caution as small changes can result in excessive pressure between extruder and hotend. Do proper calibration with filament before use. If 'DISTANCE' value is not provided then this command will return the current rotation distance.

#### SYNC_EXTRUDER_MOTION

`SYNC_EXTRUDER_MOTION EXTRUDER=<name> MOTION_QUEUE=<name>`: This command will cause the stepper specified by EXTRUDER (as defined in an [extruder](Config_Reference.md#extruder) or [extruder_stepper](Config_Reference.md#extruder_stepper) config section) to become synchronized to the movement of an extruder specified by MOTION_QUEUE (as defined in an [extruder](Config_Reference.md#extruder) config section). If MOTION_QUEUE is an empty string then the stepper will be desynchronized from all extruder movement.

#### SET_EXTRUDER_STEP_DISTANCE

This command is deprecated and will be removed in the near future.

#### SYNC_STEPPER_TO_EXTRUDER

This command is deprecated and will be removed in the near future.

### [fan_generic]

The following command is available when a [fan_generic config section](Config_Reference.md#fan_generic) is enabled.

#### SET_FAN_SPEED

`SET_FAN_SPEED FAN=config_name SPEED=<speed>` This command sets the speed of a fan. "speed" must be between 0.0 and 1.0.

### [filament_switch_sensor]

The following command is available when a [filament_switch_sensor](Config_Reference.md#filament_switch_sensor) or [filament_motion_sensor](Config_Reference.md#filament_motion_sensor) config section is enabled.

#### QUERY_FILAMENT_SENSOR

`QUERY_FILAMENT_SENSOR SENSOR=<sensor_name>`: Queries the current status of the filament sensor. The data displayed on the terminal will depend on the sensor type defined in the configuration.

#### SET_FILAMENT_SENSOR

`SET_FILAMENT_SENSOR SENSOR=<sensor_name> ENABLE=[0|1]`: 필라멘트 센서를 켜거나 끕니다. ENABLE이 0 으로 설정되면 필라멘트 센서가 비활성화되고 1 로 설정되면 활성화됩니다.

### [firmware_retraction]

The following standard G-Code commands are available when the [firmware_retraction config section](Config_Reference.md#firmware_retraction) is enabled. These commands allow you to utilize the firmware retraction feature available in many slicers, to reduce stringing during non-extrusion moves from one part of the print to another. Appropriately configuring pressure advance reduces the length of retraction required.

- `G10`: 현재 구성된 매개변수를 사용하여 익스트루더를 후퇴시킵니다.
- `G11`: 현재 구성된 매개변수를 사용하여 익스트루더를 수축 해제합니다.

The following additional commands are also available.

#### SET_RETRACTION

`SET_RETRACTION [RETRACT_LENGTH=<mm>] [RETRACT_SPEED=<mm/s>] [UNRETRACT_EXTRA_LENGTH=<mm>] [UNRETRACT_SPEED=<mm/s>]`: Firmware Retraction 에 사용되는 매개변수를 조정합니다. RETRACT_LENGTH 는 필라멘트를 수축 및 해제할 길이를 결정합니다. 후퇴 속도는 RETRACT_SPEED를 통해 조정되며 일반적으로 비교적 높게 설정됩니다. 철회 속도는 UNRETRACT_SPEED를 통해 조정되며, RETRACT_SPEED보다 낮은 경우가 많지만 특별히 중요하지는 않습니다. 어떤 경우에는 철회 시 약간의 추가 길이를 추가하는 것이 유용하며 이는 UNRETRACT_EXTRA_LENGTH를 통해 설정됩니다. SET_RETRACTION은 일반적으로 필라멘트별로 슬라이서 구성의 일부로 설정됩니다. 필라멘트마다 다른 매개변수 설정이 필요하기 때문입니다.

#### GET_RETRACTION

`GET_RETRACTION`: Firmware Retraction 에 사용되는 현재 매개변수를 쿼리하고 터미널에 표시합니다.

### [force_move]

The force_move module is automatically loaded, however some commands require setting `enable_force_move` in the [printer config](Config_Reference.md#force_move).

#### STEPPER_BUZZ

`STEPPER_BUZZ STEPPER=<config_name>`: 주어진 스테퍼를 앞으로 1mm 이동한 다음 뒤로 1mm 이동하고 10회 반복합니다. 이것은 스테퍼 연결을 확인하는 데 도움이 되는 진단 도구입니다.

#### FORCE_MOVE

`FORCE_MOVE STEPPER=<config_name> DISTANCE=<value> VELOCITY=<value> [ACCEL=<value>]`: 이 명령은 주어진 일정한 속도(mm/s)에서 주어진 거리(mm)로 주어진 스테퍼를 강제로 이동합니다. ACCEL이 지정되고 0보다 크면 지정된 가속도(mm/s^2)가 사용됩니다. 그렇지 않으면 가속이 수행되지 않습니다. 경계 검사는 수행되지 않습니다; 운동학적 업데이트가 수행되지 않습니다; 축의 다른 평행 스테퍼는 이동하지 않습니다. 잘못된 명령은 손상을 일으킬 수 있으므로 주의하십시오! 이 명령을 사용하면 낮은 수준의 운동학이 잘못된 상태에 놓이는 것이 거의 확실합니다; 이후에 G28을 발행하여 운동학을 재설정하십시오. 이 명령은 낮은 수준의 진단 및 디버깅을 위한 것입니다. `SET_KINEMATIC_POSITION [X=<value>] [Y=<value>] [Z=<value>]`: 툴 헤드가 지정된 직교 위치에 있다고 믿도록 저수준 운동학 코드를 강제 실행합니다. 이것은 진단 및 디버깅 명령입니다; 일반 축 변환에 SET_GCODE_OFFSET 및/또는 G92를 사용합니다. 축을 지정하지 않으면 헤드가 마지막으로 명령을 받은 위치로 기본 설정됩니다. 올바르지 않거나 잘못된 위치를 설정하면 내부 소프트웨어 오류가 발생할 수 있습니다. 이 명령은 향후 경계 검사를 무효화할 수 있습니다. 이후에 G28을 발행하여 운동학을 재설정하십시오.

#### SET_KINEMATIC_POSITION

`SET_KINEMATIC_POSITION [X=<value>] [Y=<value>] [Z=<value>]`: 툴 헤드가 지정된 직교 위치에 있다고 믿도록 저수준 운동학 코드를 강제 실행합니다. 이것은 진단 및 디버깅 명령입니다; 일반 축 변환에 SET_GCODE_OFFSET 및/또는 G92를 사용합니다. 축을 지정하지 않으면 헤드가 마지막으로 명령을 받은 위치로 기본 설정됩니다. 올바르지 않거나 잘못된 위치를 설정하면 내부 소프트웨어 오류가 발생할 수 있습니다. 이 명령은 향후 경계 검사를 무효화할 수 있습니다. 이후에 G28을 발행하여 운동학을 재설정하십시오.

### [gcode]

The gcode module is automatically loaded.

#### RESTART

`RESTART`: 호스트 소프트웨어가 구성을 다시 로드하고 내부 재설정을 수행합니다. 이 명령은 마이크로 컨트롤러에서 오류 상태를 지우지 않습니다. (FIRMWARE_RESTART 참조) 새 소프트웨어를 로드하지도 않습니다([FAQ](FAQ.md#how-do-i-upgrade-to-the-latest-software 참조)).

#### FIRMWARE_RESTART

`FIRMWARE_RESTART`: 이는 RESTART 명령과 유사하지만 마이크로 컨트롤러의 모든 오류 상태도 지웁니다

#### STATUS

`STATUS`: Klipper 호스트 소프트웨어 상태를 보고합니다.

#### HELP

`HELP`: 사용 가능한 확장 G 코드 명령 목록을 보고합니다.

### [gcode_arcs]

[gcode_arcs config section](Config_Reference.md#gcode_arcs)이 활성화된 경우 다음 표준 G-Code 명령을 사용할 수 있습니다.

- Arc Move Clockwise (G2), Arc Move Counter-clockwise (G3): `G2|G3 [X<pos>] [Y<pos>] [Z<pos>] [E<pos>] [F<speed>] I<value> J<value>|I<value> K<value>|J<value> K<value>`
- Arc Plane Select: G17 (XY plane), G18 (XZ plane), G19 (YZ plane)

### [gcode_macro]

The following command is available when a [gcode_macro config section](Config_Reference.md#gcode_macro) is enabled (also see the [command templates guide](Command_Templates.md)).

#### SET_GCODE_VARIABLE

`SET_GCODE_VARIABLE MACRO=<macro_name> VARIABLE=<name> VALUE=<value>`: 이 명령을 사용하면 런타임에 gcode_macro 변수의 값을 변경할 수 있습니다. 제공된 VALUE는 Python 리터럴로 구문 분석됩니다.

### [gcode_move]

The gcode_move module is automatically loaded.

#### GET_POSITION

`GET_POSITION`: Return information on the current location of the toolhead. See the developer documentation of [GET_POSITION output](Code_Overview.md#coordinate-systems) for more information.

#### SET_GCODE_OFFSET

`SET_GCODE_OFFSET [X=<pos>|X_ADJUST=<adjust>] [Y=<pos>|Y_ADJUST=<adjust>] [Z=<pos>|Z_ADJUST=<adjust>] [MOVE=1 [MOVE_SPEED=<speed>]]`: 향후 G-Code 명령에 적용할 위치 오프셋을 설정합니다. 이것은 일반적으로 Z 베드 오프셋을 가상으로 변경하거나 압출기를 전환할 때 노즐 XY 오프셋을 설정하는 데 사용됩니다. 예를 들어 "SET_GCODE_OFFSET Z=0.2"가 전송되면 향후 G-Code 이동은 Z 높이에 0.2mm가 추가됩니다.X_ADJUST 스타일 매개변수를 사용하는 경우 조정이 기존 오프셋에 추가됩니다. (예: "SET_GCODE_OFFSET Z = -0.2" 다음에 "SET_GCODE_OFFSET Z_ADJUST = 0.3"을 입력하면 총 Z 오프셋이 0.1이 됩니다.) "MOVE=1"이 지정되면 지정된 오프셋을 적용하기 위해 툴 헤드 이동이 실행됩니다. (그렇지 않으면 오프셋은 주어진 축을 지정하는 다음 절대 G 코드 이동에 적용됩니다.) "MOVE_SPEED"가 지정되면 지정된 속도(mm/s)로 툴 헤드 이동이 수행됩니다; 그렇지 않으면 툴 헤드 이동은 마지막으로 지정된 G 코드 속도를 사용합니다.

#### SAVE_GCODE_STATE

`SAVE_GCODE_STATE [NAME=<state_name>]`: 현재 g-code 좌표 구문 분석 상태를 저장합니다. g-code 상태를 저장하고 복원하는 것은 스크립트와 매크로에서 유용합니다. 이 명령은 현재 g-code 절대 좌표 모드(G90/G91), 절대 압출 모드(M82/M83), 원점(G92), 오프셋(SET_GCODE_OFFSET), 속도 오버라이드(M220), 압출기 오버라이드(M221), 현재 XYZ 위치 및 상대 압출기 "E" 위치 그리고 이동 속도를 저장합니다, NAME 이 제공되면 저장된 상태의 이름을 주어진 문자열로 지정할 수 있습니다. NAME 이 제공되지 않으면 기본값은 "default" 입니다.

#### RESTORE_GCODE_STATE

`RESTORE_GCODE_STATE [NAME=<state_name>] [MOVE=1 [MOVE_SPEED=<speed>]]`: SAVE_GCODE_STATE 를 통해 이전에 저장한 상태를 복원합니다. "MOVE=1"이 지정되면 이전 XYZ 위치로 다시 이동하기 위해 툴 헤드 이동이 실행됩니다. "MOVE_SPEED"가 지정되면 지정된 속도(mm/s)로 툴 헤드 이동이 수행됩니다; 그렇지 않으면 툴헤드 이동은 복원된 g-code 속도를 사용합니다.

### [hall_filament_width_sensor]

The following commands are available when the [tsl1401cl filament width sensor config section](Config_Reference.md#tsl1401cl_filament_width_sensor) or [hall filament width sensor config section](Config_Reference.md#hall_filament_width_sensor) is enabled (also see [TSLl401CL Filament Width Sensor](TSL1401CL_Filament_Width_Sensor.md) and [Hall Filament Width Sensor](Hall_Filament_Width_Sensor.md)):

#### QUERY_FILAMENT_WIDTH

`QUERY_FILAMENT_WIDTH`: Return the current measured filament width.

#### RESET_FILAMENT_WIDTH_SENSOR

`RESET_FILAMENT_WIDTH_SENSOR`: Clear all sensor readings. Helpful after filament change.

#### DISABLE_FILAMENT_WIDTH_SENSOR

`DISABLE_FILAMENT_WIDTH_SENSOR`: Turn off the filament width sensor and stop using it for flow control.

#### ENABLE_FILAMENT_WIDTH_SENSOR

`ENABLE_FILAMENT_WIDTH_SENSOR`: Turn on the filament width sensor and start using it for flow control.

#### QUERY_RAW_FILAMENT_WIDTH

`QUERY_RAW_FILAMENT_WIDTH`: Return the current ADC channel readings and RAW sensor value for calibration points.

#### ENABLE_FILAMENT_WIDTH_LOG

`ENABLE_FILAMENT_WIDTH_LOG`: Turn on diameter logging.

#### DISABLE_FILAMENT_WIDTH_LOG

`DISABLE_FILAMENT_WIDTH_LOG`: Turn off diameter logging.

### [heaters]

The heaters module is automatically loaded if a heater is defined in the config file.

#### TURN_OFF_HEATERS

`TURN_OFF_HEATERS`: 모든 히터를 끕니다.

#### TEMPERATURE_WAIT

`TEMPERATURE_WAIT SENSOR=<config_name> [MINIMUM=<target>] [MAXIMUM=<target>]`: 주어진 온도 센서가 제공된 최소값 이상 및/또는 제공된 최대값 이하일 때까지 기다립니다.

#### SET_HEATER_TEMPERATURE

`SET_HEATER_TEMPERATURE HEATER=<heater_name> [TARGET=<target_temperature>]`: 히터의 목표 온도를 설정합니다. 목표 온도가 제공되지 않으면 목표는 0입니다.

### [idle_timeout]

The idle_timeout module is automatically loaded.

#### SET_IDLE_TIMEOUT

`SET_IDLE_TIMEOUT [TIMEOUT=<timeout>]`: 사용자가 유휴 시간 초과(초)를 설정할 수 있습니다.

### [input_shaper]

The following command is enabled if an [input_shaper config section](Config_Reference.md#input_shaper) has been enabled (also see the [resonance compensation guide](Resonance_Compensation.md)).

#### SET_INPUT_SHAPER

`SET_INPUT_SHAPER [SHAPER_FREQ_X=<shaper_freq_x>] [SHAPER_FREQ_Y=<shaper_freq_y>] [DAMPING_RATIO_X=<damping_ratio_x>] [DAMPING_RATIO_Y=<damping_ratio_y>] [SHAPER_TYPE=<shaper>] [SHAPER_TYPE_X=<shaper_type_x>] [SHAPER_TYPE_Y=<shaper_type_y>]`: 입력 셰이퍼 매개변수를 수정합니다. SHAPER_TYPE 매개변수는 [input_shaper] 섹션에서 서로 다른 셰이퍼 유형이 구성된 경우에도 X 및 Y 축 모두에 대해 입력 셰이퍼를 재설정합니다. SHAPER_TYPE은 SHAPER_TYPE_X 및 SHAPER_TYPE_Y 매개변수와 함께 사용할 수 없습니다. 이러한 각 매개변수에 대한 자세한 내용은 [config reference](Config_Reference.md#input_shaper)를 참조하세요.

### [manual_probe]

The manual_probe module is automatically loaded.

#### MANUAL_PROBE

`MANUAL_PROBE [SPEED=<speed>]`: 주어진 위치에서 노즐 높이를 측정하는 데 유용한 도우미 스크립트를 실행합니다. SPEED가 지정되면 TESTZ 명령의 속도를 설정합니다(기본값은 5mm/s). 수동 프로브 중에 다음과 같은 추가 명령을 사용할 수 있습니다:

- `ACCEPT`: 이 명령은 현재 Z 위치를 받아들이고 수동 프로빙을 종료합니다.
- `ABORT`: 이 명령은 수동 프로빙을 종료합니다.
- `TESTZ Z=<value>`: 이 명령은 "value" 에 지정된 양만큼 노즐을 위 또는 아래로 이동합니다. 예를 들어 `TESTZ Z=-.1`은 노즐을 0.1mm 아래로 이동하고 `TESTZ Z=.1`은 노즐을 0.1mm 위로 이동합니다. 값은 또한 '+', '-', '++' 또는 '--'로 노즐을 이전 시도에 비해 위 또는 아래로 움직일 수 있습니다.

#### Z_ENDSTOP_CALIBRATE

`Z_ENDSTOP_CALIBRATE [SPEED=<speed>]`: Z position_endstop 구성 설정을 보정하는 데 유용한 도우미 스크립트를 실행합니다. 툴이 활성화되어 있는 동안 사용할 수 있는 매개변수 및 추가 명령에 대한 자세한 내용은 MANUAL_PROBE 명령을 참조하십시오.

#### Z_OFFSET_APPLY_ENDSTOP

`Z_OFFSET_APPLY_ENDSTOP`: 현재 Z Gcode 오프셋(일명 babystepping)을 가져 와서 stepper_z endstop_position에서 뺍니다.이것은 자주 사용하는 베이비 스테핑 값을 가져 와서 "영구화" 하는 역할을 합니다. 적용하려면 'SAVE_CONFIG'가 필요합니다.

### [manual_stepper]

The following command is available when a [manual_stepper config section](Config_Reference.md#manual_stepper) is enabled.

#### MANUAL_STEPPER

`MANUAL_STEPPER STEPPER=config_name [ENABLE=[0|1]] [SET_POSITION=<pos>] [SPEED=<speed>] [ACCEL=<accel>] [MOVE=<pos> [STOP_ON_ENDSTOP=[1|2|-1|-2]] [SYNC=0]]`: 이 명령은 스테퍼의 상태를 변경합니다. ENABLE 매개변수를 사용하여 스테퍼를 활성화/비활성화합니다. SET_POSITION 매개변수를 사용하여 스테퍼가 지정된 위치에 있다고 생각하도록 합니다. MOVE 매개변수를 사용하여 지정된 위치로 이동을 요청합니다. SPEED 및/또는 ACCEL이 지정되면 구성 파일에 지정된 기본값 대신 지정된 값이 사용됩니다. 0의 ACCEL이 지정되면 가속이 수행되지 않습니다. STOP_ON_ENDSTOP=1 이 지정되면 endstop 보고서가 트리거되면 이동이 일찍 종료됩니다 (엔드스톱이 트리거되지 않더라도 오류 없이 이동을 완료하려면 STOP_ON_ENDSTOP=2를 사용하고, 엔드스톱이 트리거되지 않았다고 보고할 때 중지하려면 -1 또는 -2를 사용). 일반적으로 향후 G-Code 명령은 스테퍼 이동이 완료된 후 실행되도록 예약되지만 수동 스테퍼 이동이 SYNC=0을 사용하는 경우 향후 G-Code 이동 명령은 스테퍼 이동과 병렬로 실행될 수 있습니다.

### [mcp4018]

The following command is available when a [mcp4018 config section](Config_Reference.md#mcp4018) is enabled.

#### SET_DIGIPOT

`SET_DIGIPOT DIGIPOT=config_name WIPER=<value>`: This command will change the current value of the digipot. This value should typically be between 0.0 and 1.0, unless a 'scale' is defined in the config. When 'scale' is defined, then this value should be between 0.0 and 'scale'.

### [led]

The following command is available when any of the [led config sections](Config_Reference.md#leds) are enabled.

#### SET_LED

`SET_LED LED=<config_name> RED=<value> GREEN=<value> BLUE=<value> WHITE=<value> [INDEX=<index>] [TRANSMIT=0] [SYNC=1]`: This sets the LED output. Each color `<value>` must be between 0.0 and 1.0. The WHITE option is only valid on RGBW LEDs. If the LED supports multiple chips in a daisy-chain then one may specify INDEX to alter the color of just the given chip (1 for the first chip, 2 for the second, etc.). If INDEX is not provided then all LEDs in the daisy-chain will be set to the provided color. If TRANSMIT=0 is specified then the color change will only be made on the next SET_LED command that does not specify TRANSMIT=0; this may be useful in combination with the INDEX parameter to batch multiple updates in a daisy-chain. By default, the SET_LED command will sync it's changes with other ongoing gcode commands. This can lead to undesirable behavior if LEDs are being set while the printer is not printing as it will reset the idle timeout. If careful timing is not needed, the optional SYNC=0 parameter can be specified to apply the changes without resetting the idle timeout.

#### SET_LED_TEMPLATE

`SET_LED_TEMPLATE LED=<led_name> TEMPLATE=<template_name> [<param_x>=<literal>] [INDEX=<index>]`: Assign a [display_template](Config_Reference.md#display_template) to a given [LED](Config_Reference.md#leds). For example, if one defined a `[display_template my_led_template]` config section then one could assign `TEMPLATE=my_led_template` here. The display_template should produce a comma separated string containing four floating point numbers corresponding to red, green, blue, and white color settings. The template will be continuously evaluated and the LED will be automatically set to the resulting colors. One may set display_template parameters to use during template evaluation (parameters will be parsed as Python literals). If INDEX is not specified then all chips in the LED's daisy-chain will be set to the template, otherwise only the chip with the given index will be updated. If TEMPLATE is an empty string then this command will clear any previous template assigned to the LED (one can then use `SET_LED` commands to manage the LED's color settings).

### [output_pin]

The following command is available when an [output_pin config section](Config_Reference.md#output_pin) is enabled.

#### SET_PIN

`SET_PIN PIN=config_name VALUE=<value> CYCLE_TIME=<cycle_time>`: Note - hardware PWM does not currently support the CYCLE_TIME parameter and will use the cycle time defined in the config.

### [palette2]

The following commands are available when the [palette2 config section](Config_Reference.md#palette2) is enabled.

팔레트 인쇄 작업은 특수 OCode (Omega Codes) 와 동작합니다:

- `O1`...`O32`: 이 코드는 GCode 스트림에서 읽고 이 모듈에 의해 처리되고 팔레트 2 장치로 전달됩니다.

The following additional commands are also available.

#### PALETTE_CONNECT

`PALETTE_CONNECT`: 이 명령은 팔레트 2와의 연결을 초기화합니다.

#### PALETTE_DISCONNECT

`PALETTE_DISCONNECT`: 이 명령은 팔레트2 에서 연결을 끊습니다.

#### PALETTE_CLEAR

`PALETTE_CLEAR`: 이 명령은 필라멘트의 모든 입력 및 출력 경로를 지우도록 팔레트2 에 지시합니다.

#### PALETTE_CUT

`PALETTE_CUT`: 이 명령은 현재 스플라이스 코어에 로드된 필라멘트를 절단하도록 팔레트2 에 지시합니다.

#### PALETTE_SMART_LOAD

`PALETTE_SMART_LOAD`: 이 명령은 팔레트 2에서 스마트 로드 시퀀스를 시작합니다. Filament는 프린터용 장치에서 보정된 거리만큼 밀어내어 자동으로 Load되며, Load가 완료되면 Palette 2에 지시합니다. 이 명령은 누르는 것과 동일합니다. 이 명령은 필라멘트 로드가 완료된 후 Palette 2 화면에서 직접 **Smart Load**를 누르는 것과 같습니다.

### [pid_calibrate]

The pid_calibrate module is automatically loaded if a heater is defined in the config file.

#### PID_CALIBRATE

`PID_CALIBRATE HEATER=<config_name> TARGET=<temperature> [WRITE_FILE=1]`: PID 캘리브레이션 테스트를 수행합니다. 지정된 히터는 지정된 목표 온도에 도달할 때까지 활성화되며, 그런 다음 히터가 여러 사이클 동안 꺼졌다 켜집니다. WRITE_FILE 매개변수가 활성화된 경우, /tmp/heattest.txt 파일이 테스트 중에 가져온 모든 온도 샘플의 로그와 함께 생성됩니다.

### [pause_resume]

[pause_resume config section](Config_Reference.md#pause_resume) 이 활성화된 경우 다음 명령을 사용할 수 있습니다:

#### PAUSE

`PAUSE`: 현재 인쇄를 일시 중지합니다. 재개시 복원을 위해 현재 위치가 캡처됩니다.

#### RESUME

`RESUME [VELOCITY=<value>]`: 일시 중지에서 인쇄를 다시 시작하고 이전에 캡처한 위치를 먼저 복원합니다. VELOCITY 매개변수는 도구가 원래 캡처된 위치로 돌아가야 하는 속도를 결정합니다.

#### CLEAR_PAUSE

`CLEAR_PAUSE`: 인쇄를 다시 시작하지 않고 현재 일시 중지된 상태를 지웁니다. PAUSE 후에 인쇄를 취소하기로 결정한 경우에 유용합니다. 각 인쇄에 대해 일시 중지된 상태가 최신인지 확인하려면 이를 시작 gcode에 추가하는 것이 좋습니다.

#### CANCEL_PRINT

`CANCEL_PRINT`: 현재 인쇄를 취소합니다.

### [print_stats]

The print_stats module is automatically loaded.

#### SET_PRINT_STATS_INFO

`SET_PRINT_STATS_INFO [TOTAL_LAYER=<total_layer_count>] [CURRENT_LAYER= <current_layer>]`: Pass slicer info like layer act and total to Klipper. Add `SET_PRINT_STATS_INFO [TOTAL_LAYER=<total_layer_count>]` to your slicer start gcode section and `SET_PRINT_STATS_INFO [CURRENT_LAYER= <current_layer>]` at the layer change gcode section to pass layer information from your slicer to Klipper.

### [probe]

The following commands are available when a [probe config section](Config_Reference.md#probe) or [bltouch config section](Config_Reference.md#bltouch) is enabled (also see the [probe calibrate guide](Probe_Calibrate.md)).

#### PROBE

`PROBE [PROBE_SPEED=<mm/s>] [LIFT_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>] [SAMPLES_TOLERANCE=<mm>] [SAMPLES_TOLERANCE_RETRIES=<count>] [SAMPLES_RESULT=median|average]`: 프로브가 트리거될 때까지 노즐을 아래로 이동합니다. 선택적 매개변수가 제공되면 [probe config section](Config_Reference.md#probe) 에서 해당 설정을 재정의합니다.

#### QUERY_PROBE

`QUERY_PROBE`: 프로브의 현재 상태를 보고합니다("triggered" 또는 "open").

#### PROBE_ACCURACY

`PROBE_ACCURACY [PROBE_SPEED=<mm/s>] [SAMPLES=<count>] [SAMPLE_RETRACT_DIST=<mm>]`: 여러 프로브 샘플의 최대, 최소, 평균, 중앙값 및 표준 편차를 계산합니다. 기본적으로 10개의 샘플이 사용됩니다. 그렇지 않으면 선택적 매개변수는 기본적으로 프로브 구성 섹션의 해당 설정으로 설정됩니다.

#### PROBE_CALIBRATE

`PROBE_CALIBRATE [SPEED=<speed>] [<probe_parameter>=<value>]`: 프로브의 z_offset 을 보정하는 데 유용한 도우미 스크립트를 실행합니다. 선택적 프로브 매개변수에 대한 자세한 내용은 PROBE 명령을 참조하십시오. SPEED 매개변수 및 도구가 활성화되어 있는 동안 사용할 수 있는 추가 명령에 대한 자세한 내용은 MANUAL_PROBE 명령을 참조하십시오. PROBE_CALIBRATE 명령은 속도 변수를 사용하여 Z뿐만 아니라 XY 방향으로 이동합니다.

#### Z_OFFSET_APPLY_PROBE

`Z_OFFSET_APPLY_PROBE`: Take the current Z Gcode offset (aka, babystepping), and subtract if from the probe's z_offset. 이것은 자주 사용하는 베이비 스테핑 값을 가져 와서 "영구화"하는 역할을 합니다. 적용하려면 'SAVE_CONFIG'가 필요합니다.

### [query_adc]

The query_adc module is automatically loaded.

#### QUERY_ADC

`QUERY_ADC [NAME=<config_name>] [PULLUP=<value>]`: 구성된 아날로그 핀에 대해 수신된 마지막 아날로그 값을 보고합니다. NAME 이 제공되지 않으면 사용 가능한 adc 이름 목록이 보고됩니다. PULLUP 이 제공되면 (Ohms 단위의 값으로) 해당 풀업이 제공된 등가 저항과 함께 원시 아날로그 값이 보고됩니다.

### [query_endstops]

The query_endstops module is automatically loaded. The following standard G-Code commands are currently available, but using them is not recommended:

- 엔드스탑 상태 가져오기: `M119` (대신 QUERY_ENDSTOPS 사용하세요)

#### QUERY_ENDSTOPS

`QUERY_ENDSTOPS`: 엔드스톱을 조사하고 "triggered" 되었는지 또는 "open" 상태인지 보고합니다. 이 명령은 일반적으로 엔드스톱이 올바르게 작동하는지 확인하는 데 사용됩니다.

### [resonance_tester]

The following commands are available when a [resonance_tester config section](Config_Reference.md#resonance_tester) is enabled (also see the [measuring resonances guide](Measuring_Resonances.md)).

#### MEASURE_AXES_NOISE

`MEASURE_AXES_NOISE`: 활성화된 모든 가속도계 칩의 모든 축에 대한 노이즈를 측정하고 출력합니다.

#### TEST_RESONANCES

`TEST_RESONANCES AXIS=<axis> OUTPUT=<resonances,raw_data> [NAME=<name>] [FREQ_START=<min_freq>] [FREQ_END=<max_freq>] [HZ_PER_SEC=<hz_per_sec>] [CHIPS=<adxl345_chip_name>] [POINT=x,y,z] [INPUT_SHAPING=[<0:1>]]`: Runs the resonance test in all configured probe points for the requested "axis" and measures the acceleration using the accelerometer chips configured for the respective axis. "axis" can either be X or Y, or specify an arbitrary direction as `AXIS=dx,dy`, where dx and dy are floating point numbers defining a direction vector (e.g. `AXIS=X`, `AXIS=Y`, or `AXIS=1,-1` to define a diagonal direction). Note that `AXIS=dx,dy` and `AXIS=-dx,-dy` is equivalent. `adxl345_chip_name` can be one or more configured adxl345 chip,delimited with comma, for example `CHIPS="adxl345, adxl345 rpi"`. Note that `adxl345` can be omitted from named adxl345 chips. If POINT is specified it will override the point(s) configured in `[resonance_tester]`. If `INPUT_SHAPING=0` or not set(default), disables input shaping for the resonance testing, because it is not valid to run the resonance testing with the input shaper enabled. `OUTPUT` parameter is a comma-separated list of which outputs will be written. If `raw_data` is requested, then the raw accelerometer data is written into a file or a series of files `/tmp/raw_data_<axis>_[<chip_name>_][<point>_]<name>.csv` with (`<point>_` part of the name generated only if more than 1 probe point is configured or POINT is specified). If `resonances` is specified, the frequency response is calculated (across all probe points) and written into `/tmp/resonances_<axis>_<name>.csv` file. If unset, OUTPUT defaults to `resonances`, and NAME defaults to the current time in "YYYYMMDD_HHMMSS" format.

#### SHAPER_CALIBRATE

`SHAPER_CALIBRATE [AXIS=<axis>] [NAME=<name>] [FREQ_START=<min_freq>] [FREQ_END=<max_freq>] [HZ_PER_SEC=<hz_per_sec>] [MAX_SMOOTHING=<max_smoothing>]`: `TEST_RESONANCES`와 유사하게 구성된 대로 공진 테스트를 실행하고 요청된 축(또는 `AXIS` 매개변수가 설정되지 않은 경우 X축과 Y축 모두)에 대한 입력 셰이퍼에 대한 최적의 매개변수를 찾으려고 시도합니다. `MAX_SMOOTHING`이 설정되지 않은 경우 해당 값은 `[resonance_tester]` 섹션에서 가져오고 기본값은 설정되지 않습니다. 이 기능의 사용에 대한 자세한 내용은 공진 측정 가이드의 [Max smoothing](Measuring_Resonances.md#max-smoothing) 를 참조하세요. 조정 결과는 콘솔에 인쇄되고 주파수 응답과 다양한 입력 셰이퍼 값은 CSV 파일 `/tmp/calibration_data_<axis>_<name>.csv`에 기록됩니다. 지정하지 않는 한 NAME의 기본값은 "YYYYMMDD_HHMMSS" 형식의 현재 시간입니다. 제안된 입력 셰이퍼 매개변수는 'SAVE_CONFIG' 명령을 실행하여 구성에 유지될 수 있습니다.

### [respond]

The following standard G-Code commands are available when the [respond config section](Config_Reference.md#respond) is enabled:

- `M118 <message>`: 구성된 기본 접두사가 추가된 메시지를 echo 합니다 (또는 접두사가 구성되지 않은 경우 `echo: `).

The following additional commands are also available.

#### RESPOND

- `RESPOND MSG="<message>"`: 구성된 기본 접두사가 추가된 메시지를 echo합니다(또는 접두사가 구성되지 않은 경우 `echo: `).
- `RESPOND TYPE=echo MSG="<message>"`: `echo:`가 추가된 메시지를 echo 합니다.
- `RESPOND TYPE=echo_no_space MSG="<message>"`: echo the message prepended with `echo:` without a space between prefix and message, helpful for compatibility with some octoprint plugins that expect very specific formatting.
- `RESPOND TYPE=command MSG="<message>"`: echo the message prepended with `// `. OctoPrint can be configured to respond to these messages (e.g. `RESPOND TYPE=command MSG=action:pause`).
- `RESPOND TYPE=error MSG="<message>"`: `!! `가 추가된 메시지를 echo 합니다.
- `RESPOND PREFIX=<prefix> MSG="<message>"`: `<prefix>`가 추가된 메시지를 echo 합니다. ('PREFIX' 매개변수는 'TYPE' 매개변수보다 우선합니다.)

### [save_variables]

The following command is enabled if a [save_variables config section](Config_Reference.md#save_variables) has been enabled.

#### SAVE_VARIABLE

`SAVE_VARIABLE VARIABLE=<name> VALUE=<value>`: 다시 시작할 때 사용할 수 있도록 변수를 디스크에 저장합니다. 저장된 모든 변수는 시작 시 `printer.save_variables.variables` 사전에 로드되며 gcode 매크로에서 사용할 수 있습니다. 제공된 VALUE는 Python 리터럴로 구문 분석됩니다.

### [screws_tilt_adjust]

The following commands are available when the [screws_tilt_adjust config section](Config_Reference.md#screws_tilt_adjust) is enabled (also see the [manual level guide](Manual_Level.md#adjusting-bed-leveling-screws-using-the-bed-probe)).

#### SCREWS_TILT_CALCULATE

`SCREWS_TILT_CALCULATE [DIRECTION=CW|CCW] [MAX_DEVIATION=<value>] [<probe_parameter>=<value>]`: This command will invoke the bed screws adjustment tool. It will command the nozzle to different locations (as defined in the config file) probing the z height and calculate the number of knob turns to adjust the bed level. If DIRECTION is specified, the knob turns will all be in the same direction, clockwise (CW) or counterclockwise (CCW). See the PROBE command for details on the optional probe parameters. IMPORTANT: You MUST always do a G28 before using this command. If MAX_DEVIATION is specified, the command will raise a gcode error if any difference in the screw height relative to the base screw height is greater than the value provided.

### [sdcard_loop]

When the [sdcard_loop config section](Config_Reference.md#sdcard_loop) is enabled, the following extended commands are available.

#### SDCARD_LOOP_BEGIN

`SDCARD_LOOP_BEGIN COUNT=<count>`: Begin a looped section in the SD print. A count of 0 indicates that the section should be looped indefinitely.

#### SDCARD_LOOP_END

`SDCARD_LOOP_END`: SD 인쇄에서 루프 섹션을 종료합니다.

#### SDCARD_LOOP_DESIST

`SDCARD_LOOP_DESIST`: 추가 반복 없이 기존 루프를 완료합니다.

### [servo]

The following commands are available when a [servo config section](Config_Reference.md#servo) is enabled.

#### SET_SERVO

`SET_SERVO SERVO=config_name [ANGLE=<degrees> | WIDTH=<seconds>]`: 서보 위치를 주어진 각도(도 단위) 또는 펄스 폭(초 단위)으로 설정합니다. 서보 출력을 비활성화하려면 `WIDTH=0`을 사용하십시오.

### [skew_correction]

The following commands are available when the [skew_correction config section](Config_Reference.md#skew_correction) is enabled (also see the [Skew Correction](Skew_Correction.md) guide).

#### SET_SKEW

`SET_SKEW [XY=<ac_length,bd_length,ad_length>] [XZ=<ac,bd,ad>] [YZ=<ac,bd,ad>] [CLEAR=<0|1>]`: 보정 인쇄에서 가져온 측정값(mm)으로 [skew_correction] 모듈을 구성합니다. 평면 조합에 대한 측정값을 입력할 수 있으며 입력하지 않은 평면은 현재 값을 유지합니다. 'CLEAR=1'을 입력하면 모든 기울기 보정이 비활성화됩니다.

#### GET_CURRENT_SKEW

`GET_CURRENT_SKEW`: 각 평면에 대한 현재 프린터 왜곡을 라디안과 도 단위로 보고합니다. 왜곡은 `SET_SKEW` gcode를 통해 제공된 매개변수를 기반으로 계산됩니다.

#### CALC_MEASURED_SKEW

`CALC_MEASURED_SKEW [AC=<ac_length>] [BD=<bd_length>] [AD=<ad_length>]`: Calculates and reports the skew (in radians and degrees) based on a measured print. This can be useful for determining the printer's current skew after correction has been applied. It may also be useful before correction is applied to determine if skew correction is necessary. See [Skew Correction](Skew_Correction.md) for details on skew calibration objects and measurements.

#### SKEW_PROFILE

`SKEW_PROFILE [LOAD=<name>] [SAVE=<name>] [REMOVE=<name>]`: Profile management for skew_correction. LOAD will restore skew state from the profile matching the supplied name. SAVE will save the current skew state to a profile matching the supplied name. Remove will delete the profile matching the supplied name from persistent memory. Note that after SAVE or REMOVE operations have been run the SAVE_CONFIG gcode must be run to make the changes to persistent memory permanent.

### [smart_effector]

Several commands are available when a [smart_effector config section](Config_Reference.md#smart_effector) is enabled. Be sure to check the official documentation for the Smart Effector on the [Duet3D Wiki](https://duet3d.dozuki.com/Wiki/Smart_effector_and_carriage_adapters_for_delta_printer) before changing the Smart Effector parameters. Also check the [probe calibration guide](Probe_Calibrate.md).

#### SET_SMART_EFFECTOR

`SET_SMART_EFFECTOR [SENSITIVITY=<sensitivity>] [ACCEL=<accel>] [RECOVERY_TIME=<time>]`: Set the Smart Effector parameters. When `SENSITIVITY` is specified, the respective value is written to the SmartEffector EEPROM (requires `control_pin` to be provided). Acceptable `<sensitivity>` values are 0..255, the default is 50. Lower values require less nozzle contact force to trigger (but there is a higher risk of false triggering due to vibrations during probing), and higher values reduce false triggering (but require larger contact force to trigger). Since the sensitivity is written to EEPROM, it is preserved after the shutdown, and so it does not need to be configured on every printer startup. `ACCEL` and `RECOVERY_TIME` allow to override the corresponding parameters at run-time, see the [config section](Config_Reference.md#smart_effector) of Smart Effector for more info on those parameters.

#### RESET_SMART_EFFECTOR

`RESET_SMART_EFFECTOR`: Resets Smart Effector sensitivity to its factory settings. Requires `control_pin` to be provided in the config section.

### [stepper_enable]

The stepper_enable module is automatically loaded.

#### SET_STEPPER_ENABLE

`SET_STEPPER_ENABLE STEPPER=<config_name> ENABLE=[0|1]`: 지정된 스테퍼만 활성화 또는 비활성화합니다. 이것은 진단 및 디버깅 도구이며 주의해서 사용해야 합니다. 축 모터를 비활성화해도 원점 복귀 정보는 재설정되지 않습니다. 비활성화된 스테퍼를 수동으로 이동하면 기계가 안전 한계를 벗어나 모터를 작동할 수 있습니다. 이로 인해 축 구성 요소, 핫 엔드 및 인쇄 표면이 손상될 수 있습니다.

### [temperature_fan]

The following command is available when a [temperature_fan config section](Config_Reference.md#temperature_fan) is enabled.

#### SET_TEMPERATURE_FAN_TARGET

`SET_TEMPERATURE_FAN_TARGET temperature_fan=<temperature_fan_name> [target=<target_temperature>] [min_speed=<min_speed>] [max_speed=<max_speed>]`: Sets the target temperature for a temperature_fan. If a target is not supplied, it is set to the specified temperature in the config file. If speeds are not supplied, no change is applied.

### [tmcXXXX]

The following commands are available when any of the [tmcXXXX config sections](Config_Reference.md#tmc-stepper-driver-configuration) are enabled.

#### DUMP_TMC

`DUMP_TMC STEPPER=<name>`: 이 명령은 TMC 드라이버 레지스터를 읽고 값을 보고합니다.

#### INIT_TMC

`INIT_TMC STEPPER=<name>`: This command will initialize the TMC registers. Needed to re-enable the driver if power to the chip is turned off then back on.

#### SET_TMC_CURRENT

`SET_TMC_CURRENT STEPPER=<name> CURRENT=<amps> HOLDCURRENT=<amps>`: 이것은 TMC 드라이버의 실행 및 유지 전류를 조정합니다. (HOLDCURRENT는 tmc2660 드라이버에는 적용되지 않습니다.)

#### SET_TMC_FIELD

`SET_TMC_FIELD STEPPER=<name> FIELD=<field> VALUE=<value>`: 이렇게 하면 TMC 드라이버의 지정된 레지스터 필드 값이 변경됩니다. 이 명령은 런타임 중에 필드를 변경하면 프린터의 바람직하지 않고 잠재적으로 위험한 동작으로 이어질 수 있기 때문에 낮은 수준의 진단 및 디버깅만을 위한 것입니다. config 파일을 사용하여 영구적으로 변경해야 합니다. 주어진 값에 대해 sanity check 가 수행되지 않습니다.

### [toolhead]

The toolhead module is automatically loaded.

#### SET_VELOCITY_LIMIT

`SET_VELOCITY_LIMIT [VELOCITY=<value>] [ACCEL=<value>] [ACCEL_TO_DECEL=<value>] [SQUARE_CORNER_VELOCITY=<value>]`: 프린터의 속도 제한을 수정합니다.

### [tuning_tower]

The tuning_tower module is automatically loaded.

#### TUNING_TOWER

`TUNING_TOWER COMMAND=<command> PARAMETER=<name> START=<value> [SKIP=<value>] [FACTOR=<value> [BAND=<value>]] | [STEP_DELTA=<value> STEP_HEIGHT=<value>]`: A tool for tuning a parameter on each Z height during a print. The tool will run the given `COMMAND` with the given `PARAMETER` assigned to a value that varies with `Z` according to a formula. Use `FACTOR` if you will use a ruler or calipers to measure the Z height of the optimum value, or `STEP_DELTA` and `STEP_HEIGHT` if the tuning tower model has bands of discrete values as is common with temperature towers. If `SKIP=<value>` is specified, the tuning process doesn't begin until Z height `<value>` is reached, and below that the value will be set to `START`; in this case, the `z_height` used in the formulas below is actually `max(z - skip, 0)`. There are three possible combinations of options:

- `FACTOR`: The value changes at a rate of `factor` per millimeter. The formula used is: `value = start + factor * z_height`. You can plug the optimum Z height directly into the formula to determine the optimum parameter value.
- `FACTOR` and `BAND`: The value changes at an average rate of `factor` per millimeter, but in discrete bands where the adjustment will only be made every `BAND` millimeters of Z height. The formula used is: `value = start + factor * ((floor(z_height / band) + .5) * band)`.
- `STEP_DELTA` and `STEP_HEIGHT`: The value changes by `STEP_DELTA` every `STEP_HEIGHT` millimeters. The formula used is: `value = start + step_delta * floor(z_height / step_height)`. You can simply count bands or read tuning tower labels to determine the optimum value.

### [virtual_sdcard]

Klipper supports the following standard G-Code commands if the [virtual_sdcard config section](Config_Reference.md#virtual_sdcard) is enabled:

- SD 카드 목록 보기: `M20`
- SD 카드 초기화: `M21`
- SD 파일 선택: `M23 <filename>`
- SD 프린트 시작/재시작: `M24`
- SD 프린트 잠시멈춤: `M25`
- SD 위치 설정: `M26 S<offset>`
- SD 프린트 상태 리포트: `M27`

In addition, the following extended commands are available when the "virtual_sdcard" config section is enabled.

#### SDCARD_PRINT_FILE

`SDCARD_PRINT_FILE FILENAME=<filename>`: Load a file and start SD print.

#### SDCARD_RESET_FILE

`SDCARD_RESET_FILE`: Unload file and clear SD state.

### [z_thermal_adjust]

The following commands are available when the [z_thermal_adjust config section](Config_Reference.md#z_thermal_adjust) is enabled.

#### SET_Z_THERMAL_ADJUST

`SET_Z_THERMAL_ADJUST [ENABLE=<0:1>] [TEMP_COEFF=<value>] [REF_TEMP=<value>]`: Enable or disable the Z thermal adjustment with `ENABLE`. Disabling does not remove any adjustment already applied, but will freeze the current adjustment value - this prevents potentially unsafe downward Z movement. Re-enabling can potentially cause upward tool movement as the adjustment is updated and applied. `TEMP_COEFF` allows run-time tuning of the adjustment temperature coefficient (i.e. the `TEMP_COEFF` config parameter). `TEMP_COEFF` values are not saved to the config. `REF_TEMP` manually overrides the reference temperature typically set during homing (for use in e.g. non-standard homing routines) - will be reset automatically upon homing.

### [z_tilt]

The following commands are available when the [z_tilt config section](Config_Reference.md#z_tilt) is enabled.

#### Z_TILT_ADJUST

`Z_TILT_ADJUST [<probe_parameter>=<value>]`: 이 명령은 구성에 지정된 지점을 조사한 다음 기울기를 보정하기 위해 각 Z 스테퍼를 독립적으로 조정합니다. 선택적 프로브 매개변수에 대한 자세한 내용은 PROBE 명령을 참조하십시오.
