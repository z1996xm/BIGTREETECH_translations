# 功能

Klipper 有幾個引人注目的功能：

* High precision stepper movement. Klipper utilizes an application processor (such as a low-cost Raspberry Pi) when calculating printer movements. The application processor determines when to step each stepper motor, it compresses those events, transmits them to the micro-controller, and then the micro-controller executes each event at the requested time. Each stepper event is scheduled with a precision of 25 micro-seconds or better. The software does not use kinematic estimations (such as the Bresenham algorithm) - instead it calculates precise step times based on the physics of acceleration and the physics of the machine kinematics. More precise stepper movement provides quieter and more stable printer operation.
* 同類項目中最佳的效能。 Klipper 能夠在新舊微控制器上實現高步進速率。即使是舊的 8 位微控制器也可以發送超過每秒 175K 步的速率。在較新的微控制器上，每秒數百萬步也可以實現。更高的步進速率可以實現更高的列印速度。步進事件計時即使在高速下也能保持精確，提高了整體穩定性。
* Klipper 支援帶有多個微控制器的印表機。例如，一個微控制器可以被用來控制擠出機，而另一個用來控制加熱器，並使用第三個來控制其他的印表機元件。Klipper 主機程式實現了時鐘同步，解決了微處理器之間的時鐘漂移。 啟用多個控制器只需要在配置檔案中新增幾行，不需要任何特殊程式碼。
* 通過簡單的配置檔案進行配置。修改設定不需要重新刷寫微控制器。Klipper 的所有配置都被儲存在一個易編輯的配置檔案中，大大減少了配置與維護硬體的難度。
* Klipper 支援「平滑提前壓力」--一種考慮了擠出機內壓力影響的機制。這項技術可以減少噴嘴溢料並改善轉角的列印質量。Klipper 的實現不會引入瞬間擠出機速度變化，改善了整體穩定性和穩健性。
* 支援使用「輸入整形」來減少振動對列印質量的影響。這項功能可以減少或消除列印件的「振紋(ringing)」（又名「ghosting」，「echoing」，或「rippling」）。在一些情況下它可以在保持列印質量的同時提高列印速度。
* Klipper 使用「迭代求解器」從簡單的運動學方程中計算精準的步進時間。這降低了移植Klipper到新的機械結構的難度並保證了精確的步進計時（而不需要「線段化」）。
* Klipper is hardware agnostic. One should get the same precise timing independent of the low-level electronics hardware. The Klipper micro-controller code is designed to faithfully follow the schedule provided by the Klipper host software (or prominently alert the user if it is unable to). This makes it easier to use available hardware, to upgrade to new hardware, and to have confidence in the hardware.
* 易移植的程式碼。Klipper 可以在 ARM，AVR，和PRU架構的微控制器上執行。現有的「reprap」類印表機不需要改動任何硬體就可以執行 Klipper，只需要加一個樹莓派。Klipper 的內部程式碼結構使它能夠被簡單的移植到其他架構。
* 簡潔的程式碼。大部分 Klipper 程式碼使用一個極高級程式語言（Python），這包括了運動演算法，G程式碼解析，加熱，溫度感測器演算法和其他，降低了開發新功能的難度。
* 自定義可程式設計指令碼。可以在印表機配置檔案中定義新的G程式碼命令（而不需要修改任何程式碼）。這些命令都是可程式設計的，可以能根據印表機的狀態做出不同的響應。
* 內建API伺服器。除了標準G程式碼介面，Klipper也支援富JSON API。使程式設計師能編寫對印表機進行精細控制的外接程式。

## 其他功能

Klipper 支援許多標準的 3d 印表機功能：

* Several web interfaces available. Works with Mainsail, Fluidd, OctoPrint and others. This allows the printer to be controlled using a regular web-browser. The same Raspberry Pi that runs Klipper can also run the web interface.
* 標準 G 程式碼支援。支援由常見「切片軟體」（SuperSlicer、Cura、PrusaSlicer 等）產生的通用 G 程式碼命令。
* 支援多擠出機。包括對共享熱端的擠出機（多進一出）和多頭（IDEX）的支援。
* Support for cartesian, delta, corexy, corexz, hybrid-corexy, hybrid-corexz, deltesian, rotary delta, polar, and cable winch style printers.
* 自動床面平整支援。Klipper可以被配置為基本的床身傾斜檢測或網床調平。如果床鋪使用多個Z步進器，那麼Klipper也可以通過獨立操縱Z步進器來調平。支援大多數Z高度探頭，包括BL-Touch探頭和伺服啟用的探頭。
* 支援自動delta校準。校準工具可以進行基本的高度校準，以及增強的X和Y尺寸校準。校準可以用Z型高度探頭或通過手動探測來完成。
* Run-time "exclude object" support. When configured, this module may facilitate canceling of just one object in a multi-part print.
* 支援常見的溫度感測器（例如，常見的熱敏電阻、AD595、AD597、AD849x、PT100、PT1000、MAX6675、MAX31855、MAX31856、MAX31865、BME280、HTU21D、DS18B20和LM75）。還可以配置自定義熱敏電阻和自定義模擬溫度感測器。還可以監測微控制器和 Raspberry Pi 內部的溫度感測器。
* 預設啟用基本加熱器保護。
* 支援標準風扇、噴嘴風扇和溫控風扇。不需要在印表機閑置時保持風扇運轉。可以在帶有轉速錶的風扇上監測風扇速度。
* Support for run-time configuration of TMC2130, TMC2208/TMC2224, TMC2209, TMC2660, and TMC5160 stepper motor drivers. There is also support for current control of traditional stepper drivers via AD5206, DAC084S085, MCP4451, MCP4728, MCP4018, and PWM pins.
* 支援直接連線到印表機的普通LCD顯示器。還提供了一個預設的菜單。顯示器和菜單的內容可以通過配置檔案完全定製。
* 恒定加速和「look-ahead」（前瞻）支援。所有印表機移動將從靜止逐漸加速到巡航速度，然後減速回到靜止。對傳入的 G 程式碼移動命令流進行排隊和分析 - 將優化類似方向上的移動之間的加速度，以減少列印停頓並改善整體列印時間。
* Klipper 實現了一種「步進相位限位」演算法，可以提高典型限位開關的精度。如果調整得當，它可以提高列印件首層和列印床的附著力。
* 支援列印絲存在感測器、列印絲運動感測器和列印絲寬度感測器。
* Support for measuring and recording acceleration using an adxl345, mpu9250, and mpu6050 accelerometers.
* 支援限制短距離「之」字形移動的最高速度，以減少印表機的振動和噪音。更多資訊見[運動學](Kinematics.md)文件。
* 許多常見的印表機都有樣本配置檔案。檢視[配置資料夾](../config/)中的列表。

要開始使用Klipper，請閱讀[安裝](Installation.md)指南。

## 步速基準測試

下面是步進效能測試的結果。顯示的數字代表了微控制器上每秒的總步數。

| 微控制器 | 1個活躍步進電機 | 3個步進器活躍 |
| --- | --- | --- |
| 16Mhz AVR | 157K | 99K |
| 20Mhz AVR | 196K | 123K |
| SAMD21 | 686K | 471K |
| STM32F042 | 814K | 578K |
| Beaglebone 可程式設計實時單元 | 866K | 708K |
| STM32G0B1 | 1103K | 790K |
| STM32F103 | 1180K | 818K |
| SAM3X8E | 1273K | 981K |
| SAM4S8C | 1690K | 1385K |
| LPC1768 | 1923K | 1351K |
| LPC1769 | 2353K | 1622K |
| RP2040 | 2400K | 1636K |
| SAM4E8E | 2500K | 1674K |
| SAMD51 | 3077K | 1885K |
| STM32F407 | 3652K | 2459K |
| STM32F446 | 3913K | 2634K |
| STM32H743 | 9091K | 6061K |

如果不確定特定板上的微控制器，請找到適當的 [配置文件](../config/)，並在該文件頂部的註釋中查找微控制器名稱。

關於基準測試的更多詳細資訊可在[基準測試文件](Benchmarks.md)中找到。
