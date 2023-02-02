# Example configurations

Dieses Dokument enthält Richtlinien für das Einbringen einer Klipper-Beispielkonfiguration in das Klipper-Github-Repository (zu finden im [config-Verzeichnis](../config/)).

Beachten Sie, dass der [Klipper Community Discourse Server](https://community.klipper3d.org) auch eine nützliche Ressource ist, um Konfigurationsdateien zu finden und zu teilen.

## Guidelines

1. Select the appropriate config filename prefix:
   1. The `printer` prefix is used for stock printers sold by a mainstream manufacturer.
   1. The `generic` prefix is used for a 3d printer board that may be used in many different types of printers.
   1. The `kit` prefix is for 3d printers that are assembled according to a widely used specification. These "kit" printers are generally distinct from normal "printers" in that they are not sold by a manufacturer.
   1. The `sample` prefix is used for config "snippets" that one may copy-and-paste into the main config file.
   1. The `example` prefix is used to describe printer kinematics. This type of config is typically only added along with code for a new type of printer kinematics.
1. All configuration files must end in a `.cfg` suffix. The `printer` config files must end in a year followed by `.cfg` (eg, `-2019.cfg`). In this case, the year is an approximate year the given printer was sold.
1. Do not use spaces or special characters in the config filename. The filename should contain only characters `A-Z`, `a-z`, `0-9`, `-`, and `.`.
1. Klipper must be able to start `printer`, `generic`, and `kit` example config file without error. These config files should be added to the [test/klippy/printers.test](../test/klippy/printers.test) regression test case. Add new config files to that test case in the appropriate section and in alphabetical order within that section.
1. The example configuration should be for the "stock" configuration of the printer. (There are too many "customized" configurations to track in the main Klipper repository.) Similarly, we only add example config files for printers, kits, and boards that have mainstream popularity (eg, there should be at least a 100 of them in active use). Consider using the [Klipper Community Discourse server](https://community.klipper3d.org) for other configs.
1. Only specify those devices present on the given printer or board. Do not specify settings specific to your particular setup.
   1. For `generic` config files, only those devices on the mainboard should be described. For example, it would not make sense to add a display config section to a "generic" config as there is no way to know if the board will be attached to that type of display. If the board has a specific hardware port to facilitate an optional peripheral (eg, a bltouch port) then one can add a "commented out" config section for the given device.
   1. Do not specify `pressure_advance` in an example config, as that value is specific to the filament, not the printer hardware. Similarly, do not specify `max_extrude_only_velocity` nor `max_extrude_only_accel` settings.
   1. Do not specify a config section containing a host path or host hardware. For example, do not specify `[virtual_sdcard]` nor `[temperature_host]` config sections.
   1. Only define macros that utilize functionality specific to the given printer or to define g-codes that are commonly emitted by slicers configured for the given printer.
1. Where possible, it is best to use the same wording, phrasing, indentation, and section ordering as the existing config files.
   1. The top of each config file should list the type of micro-controller the user should select during "make menuconfig". It should also have a reference to "docs/Config_Reference.md".
   1. Do not copy the field documentation into the example config files. (Doing so creates a maintenance burden as an update to the documentation would then require changing it in many places.)
   1. Example config files should not contain a "SAVE_CONFIG" section. If necessary, copy the relevant fields from the SAVE_CONFIG section to the appropriate section in the main config area.
   1. Use `field: value` syntax instead of `field=value`.
   1. When adding an extruder `rotation_distance` it is preferable to specify a `gear_ratio` if the extruder has a gearing mechanism. We expect the rotation_distance in the example configs to correlate with the circumference of the hobbed gear in the extruder - it is normally in the range of 20 to 35mm. When specifying a `gear_ratio` it is preferable to specify the actual gears on the mechanism (eg, prefer `gear_ratio: 80:20` over `gear_ratio: 4:1`). See the [rotation distance document](Rotation_Distance.md#using-a-gear_ratio) for more information.
   1. Avoid defining field values that are set to their default value. For example, one should not specify `min_extrude_temp: 170` as that is already the default value.
   1. Where possible, lines should not exceed 80 columns.
   1. Avoid adding attribution or revision messages to the config files. (For example, avoid adding lines like "this file was created by ...".) Place attribution and change history in the git commit message.
1. Do not use any deprecated features in the example config file.
1. Do not disable a default safety system in an example config file. For example, a config should not specify a custom `max_extrude_cross_section`. Do not enable debugging features. For example there should not be a `force_move` config section.
1. All known boards that Klipper supports can use the default serial baud rate of 250000. Do not recommend a different baud rate in an example config file.

Example config files are submitted by creating a github "pull request". Please also follow the directions in the [contributing document](CONTRIBUTING.md).
