# HDMI5

<img src=img/HDMI5/HDMI5_Title.png width="600"/>

## **介绍**

<p>BIGTREETECH HDMI5 V1.0是由深圳市必趣科技有限公司3D打印团队开发的通用5英寸HDMI显示屏。</p>

## **主要特点**

- HDMI输入，可以与Raspberry Pi配合使用。
- 连接到电脑，它可以用作电脑显示器。
- 液晶显示器采用5英寸IPS电容式触摸屏，分辨率为800x480。
- 支持5点触控。
- 内置音频解码电路，支持3.5mm耳机插孔音频输出。
- 支持亮度和显示方向调整。

## **基本参数**

- 产品尺寸：121 x 76mm
- 安装尺寸：121 x 76mm。有关更多详细信息，请阅读：**尺寸**
- 电源输入：DC 5V
- 逻辑电压：DC 3.3V
- 屏幕尺寸：5英寸IPS显示屏
- 屏幕分辨率：800x480
- 屏幕视角：160°

## **尺寸**

<img src=img/HDMI5/HDMI5_Dimension.png width="600"/>

***您可以在此处阅读更多详细信息：**BTT HDMI5_V1.0_SIZE\**** 

## **接口示意图**

<img src=img/HDMI5/HDMI5_Interface.png width="600"/>

## **硬件配置**

### **连接到显示输出设备**

<p>1.使用C型数据电缆将HDMI5连接到显示输出设备（与Raspberry Pi/PC/其他支持HDMI显示输出的设备兼容）。当连接到电脑时，电脑会在正常情况下自动加载驱动程序。在加载驱动程序之后，可以识别触摸设备。</p>

<p>2.使用HDMI电缆将HDMI5连接到显示输出设备。通常，连接HDMI电缆后，LCD可以在5秒内正常显示。</p>

### **音频输出**

将3.5毫米耳机/扬声器插入AUDIO接口，实现音频输出。

<img src=img/HDMI5/HDMI5_Audio.png width="600"/>

### **屏幕亮度调整**

<p>BIGTREETECH HDMI5 V1.0支持亮度调节，可以通过Ks1按钮增加亮度，通过Ks3按钮降低亮度。</p>

<img src=img/HDMI5/HDMI5_Brightness.png width="600"/>

### **显示方向调整**

BIGTREETECH HDMI5 V1.0支持通过Ks2按钮进行水平显示方向调整。

<img src=img/HDMI5/HDMI5_Direction.png width="600"/>

### **指示灯**

当主板通电时：

电源指示灯D11（电源）红灯亮起，表示电源工作正常。

工作状态指示灯D12（状态）绿光闪烁，表示屏幕工作正常。

## **软件配置**

### **使用树莓派**

#### **HDMI显示输出**

1.在Raspberry Pi官方网站下载：

​	Raspberry Pi OS with desktop

​	Release date: April 4th 2022

​	System: 32-bit

​	Kernel version: 5.15

​	Debian version: 11 (bullseye)

 

2.将镜像写入TF卡，然后在config.txt中修改以下配置：

```
# uncomment to force a specific HDMI mode (this will force VGA)
hdmi_group=2
hdmi_mode=87
hdmi_cvt 800 480 60 6 0 0 0
# uncomment to force a HDMI mode rather than DVI. This can make audio work in
# DMT (computer monitor) modes
hdmi_drive=1
```

#### **HDMI音频输出**

1.树莓派系统版本:	

​	Raspberry Pi OS with desktop

​	Release date: April 4th 2022

​	System: 32-bit

​	Kernel version: 5.15

​	Debian version: 11 (bullseye)

 

2.进入系统桌面后，右键单击右上角的音频源图标，然后选择HDMI。

<img src=img/HDMI5/HDMI5_Desktop.png width="600"/>
