# Using PWM tools

В этом документе описывается, как настроить лазер или шпиндель с ШИМ-управлением, используя `output_pin' и некоторые макросы.

## Как это работает?

Перенастроив ШИМ-выход вентилятора печатающей головки, вы можете управлять лазерами или шпинделями. Это полезно, если вы используете переключаемые печатающие головки, например, E3D устройство для смены инструмента или решение "Сделай сам". Обычно кулачковые инструменты, такие как LaserWeb, могут быть сконфигурированы для использования команд `M3-M5`, которые обозначают *скорость вращения шпинделя CW * (`M3 S[0-255]`), * скорость вращения шпинделя CCW* (`M4 S[0-255]`) и * остановка шпинделя* (`M5`).

** Внимание: ** При управлении лазером соблюдайте все возможные меры предосторожности! Диодные лазеры обычно инвертируются. Это означает, что при перезапуске MCU лазер будет * полностью включен * в течение времени, необходимого MCU для повторного запуска. Для достижения наилучших результатов рекомендуется * всегда * надевать соответствующие лазерные очки с нужной длиной волны, если лазер включен; и отключать лазер, когда он не нужен. Кроме того, вы должны настроить тайм-аут безопасности, чтобы, когда ваш хост или MCU столкнутся с ошибкой, инструмент остановится.

For an example configuration, see [config/sample-pwm-tool.cfg](/config/sample-pwm-tool.cfg).

## Текущие ограничения

There is a limitation of how frequent PWM updates may occur. While being very precise, a PWM update may only occur every 0.1 seconds, rendering it almost useless for raster engraving. However, there exists an [experimental branch](https://github.com/Cirromulus/klipper/tree/laser_tool) with its own tradeoffs. In long term, it is planned to add this functionality to main-line klipper.

## Commands

`M3/M4 S<value>` : Set PWM duty-cycle. Values between 0 and 255. `M5` : Stop PWM output to shutdown value.

## Laserweb Configuration

If you use Laserweb, a working configuration would be:

    GCODE START:
        M5            ; Disable Laser
        G21           ; Set units to mm
        G90           ; Absolute positioning
        G0 Z0 F7000   ; Set Non-Cutting speed
    
    GCODE END:
        M5            ; Disable Laser
        G91           ; relative
        G0 Z+20 F4000 ;
        G90           ; absolute
    
    GCODE HOMING:
        M5            ; Disable Laser
        G28           ; Home all axis
    
    TOOL ON:
        M3 $INTENSITY
    
    TOOL OFF:
        M5            ; Disable Laser
    
    LASER INTENSITY:
        S
