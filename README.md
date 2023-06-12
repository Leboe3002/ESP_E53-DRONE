# ESP_E53-DRONE 
# ESP_E58-Drone Upgrades 

Controlling an E58 drone with an ESP8266 uC

The idea of this project is to use a really cheap drone as **plattform for experiements with GPS, compass, optical flow, and other sensors and finally to build an autonomous drone**. To achive that, first I had to find a way for simply adding new sensors and control software, if possible without reengineering the the whole onboard-software. The approach is, to add an additional ESP8266 controller onboard and let it interface with the drone's native controller via WiFi. Also add some upgrades from Autel Robotics EVO II Dual 640T V3 .  I am not turning E58 Into EVO ,I am just doing it better drone from normal E58 drones 

## The Drone

The Eachine E58 Upgrades (https://www.eachine.com/de/EACHINE-E58-WIFI-FPV-With-108MP-Wide-Angle-Camera-High resolution-Hold-Mode-Foldable-RC-Drone-Quadcopter-RTF-p-1045.html) is a small (< 100g)  drone (< 800€) that looks more reasonable price for mods used. It has separate RC and WiFi(!) control, reasonable flight characteristics and and upgraded camera high resolution  somewhat better image quality.also it has GPS added an optical flow sensor for stabilization. No need to worry be done about  wind outside.can with stand little bit harsh weather but not extremely 

The basic controller and the protocols  are quite similar  to other ESP E53 drones. They all use similar smartphone apps for video and control, which all seem to be variants from the same OEM build. So the results of this project should be portable to many similar drones.

## The Hardware Mod

To save weight I tried to use the drones battery also as power supply for the additional hardware. This seems to work at least for an ESP8266 and a compass modul. 

First I opened the drone, just 6 screws and you can remove the upper body part:

<img src="https://raw.githubusercontent.com/martin-ger/ESP_E58-Drone/main/IMG_20201025_112509899_HDR_s.jpg">

Now I soldered 2 wires to the down side of the battery/switch PCB - just plain B+ and B-:

<img src="https://raw.githubusercontent.com/martin-ger/ESP_E58-Drone/main/IMG_20201025_113207185_HDR_s.jpg">

Made a small hole in the body behind the left front arm to get the wire out:

<img src="https://raw.githubusercontent.com/martin-ger/ESP_E58-Drone/main/IMG_20201025_121400787_HDR_s.jpg">

Mounted a Wemos D1 mini on the back of the drone and connected battery power via a small switch to 5V and GND - in the front you see a GY-511 LSM303DLHC compass modul connected via I2C to the ESP:

<img src="https://raw.githubusercontent.com/martin-ger/ESP_E58-Drone/main/IMG_20201025_151930113_HDR_s.jpg">

This is now my platform for further experiments...

## Results

The drone can fly via the App as normal, while it is finally controlled by the intermediate ESP8266. The E58Contol.ino sketch will connect to the drone (change STASSID for your drone) and offers a WiFi SSID "E58drone". When you connect your JYUfo App to this SSID "E58drone", you can fly the drone as normal. When the App turns off (or loses signal to the ESP's WiFi network) the ESP will continue to send control commands to the drone. Deploy Rapidly

Deploy in under a minute. The  E58 with the upgrades of EVO II 640T V3 can go from its case to the air in 45 seconds.

No Fly Zones*E58 does not have any no fly zones and will not prevent the pilot from taking off.

## What I have learned so far:

- The drone can lift at least an GPS-Modul, its Antenna, and a WEMOS D1 mini (22g)

- The battery is powerful enough to supply the drone and the additional ESP8266 (around 70mA plus)

- The GY-511 LSM303DLHC compass modul seems to work on the drone

- Tests with the data of an ublox NEO-6M GPS receiver and http://freenmea.net/ were successful, best GPS accuracy with 10 satellites is less than 1m 

- The Eachine E58 drone itself has a Lewei LW9809 camera/WiFi modul (http://www.le-wei.com/eacp_view.asp?id=66)

- It opens an access point and has a DHCP server, that can connect at least two WiFi clients at the same time

- An ESP8266 can connect to the drone without any problems (even in parallel to the JYUfo App)

- The E58 drone can be operated via WiFi and the JYUfo App without major problems via an intermediate ESP8266 router (https://github.com/martin-ger/esp_wifi_repeater)

- The Eachine E58 drone uses the WiFi protocol described here: https://blog.horner.tj/hacking-chinese-drones-for-fun-and-no-profit/

```

    1st byte – Header: 66

    2nd byte – Left/right movement (0-254, with 128 being neutral)

    3rd byte – Forward/backward movement (0-254, with 128 being neutral)

    4th byte – Throttle (elevation) (0-254, with 128 being neutral)

    5th byte – Turning movement (0-254, with 128 being neutral)

    6th byte – Reserved for commands (0 = no command)

    7th byte – Checksum (XOR of bytes 2, 3, 4, and 5)

    8th byte – Footer: 99

Command Bits (can be ORed):

    01 – Auto Take-Off

    02 - Land

    04 - Emergency Stop

    08 - 360 Deg Roll

    10 - Headless mode

    20 - Lock

    40 – Unlock Motor

    80 – Calibrate Gyro

 ```

- There is a one-way serial connection from the camera/WiFi controller to the flight controller (1 wire, see https://www.youtube.com/watch?v=HoZUKzStchg 9:55). This is a UART protocol at 19200 bps using the same protocol as above.

- Indedendent of the WiFi IP address of smartphone, the JYUfo App always wants to connect to IP 192.168.0.1

- As soon as the drone gets no more messages on UDP port 50000, the drone stops its motors immediately

- The control messages to UDP port 50000 are send about every 50ms

- The take off command (byte 5: 01) is sent 22 times in a row (about 1.1 s)

- The JYUfo App sends, as soon as the contol interface is on a UDP message to port 40000, content seems to be these 7 bytes: 63 63 XX 00 00 00 00, where XX is 01 or 02 (first connect/re-connect?). No real idea about its meaning.

- If there is no drone response, it repeats this message to port 40000 about once per second.

## Further Ideas:

- Would it be possible to add a GPS controller via WiFi (or serial connection)?

- The non-intrusive idea for a GPS controller via WiFi would be, to use an intermediate ESP between the App and the drone (as router). This router could forward all move commands from the controller, thus you can fly the drone normally. As soon as the controller sends no more movements, the GPS takes over an tries to hold the position. Orientation of the drone could either be deduced from the last movements or with an additional compass.

## Other Programs:

- The E58.ino sketch simply starts the drone by sending the UDP command to port 50000 (CAUTION: no further control - keep it in your hand or it will crash!)


# Table of contents
1. [Installation and Build Instructions](#installation-and-build-instructions)
  * [NOTICE](#notice)
  * [Prerequisities](#prerequisties)
    * [Install NDK](#install-ndk)
    * [Install Android SDK](#install-android-sdk)
    * [Install java](#install-java)
    * [Install GStreamer SDK](#install-gstreamer-sdk)
  * [Command line build instructions](#command-line-build-instructions)
  * [Installation](#installation)
1. [Usage](#usage)

# Installation and Build Instructions

## NOTICE

This code uses the GStreamer SDK and links with the stlport_static library. Please review the respective licenses
for these tools as all users of this code are responsible for ensuring that their usage of the project is 
compatible with the various project licenses.

## Prerequisites 

### Install NDK
  
  Install the Android NDK version r10e ie **android-ndk-r10e**
  1. Download the ndk android-ndk-r10e from [here](https://developer.android.com/ndk/downloads/older_releases.html)
    1. Download URL: https://dl.google.com/android/repository/android-ndk-r10e-linux-x86_64.zip
  1. unzip the file to a folder.

### Install Android SDK.

  ```
    Note:  This is only needed for command line builds.
  ```
  1. Download the Android SDK command line tools from [here](https://developer.android.com/studio/index.html#downloads)
    1. For linux variant, download the following file: tools_r25.2.3-linux.zip
  1. Unzip the file to a folder.
  1. Run the "android" application from the unzipped folder.
    1. **NOTE**: Running the "android" application needs GUI.  Running natively on the linux host is recommended or have X11 forwarding.
    1. **NOTE**: When running the "android" app keep note of the SDK path.  This will be needed later.
  1. Install the following packages/tools:
    1. Android SDK tools( 25.2.3)
    1. Android SDK Platform Tools( 25.0.3)
    1. Android SDK Build Tools( 25.0.2 )
    1. Android SDK Build Tools( 21.1.2 )
    1. Android 5.0.1( API 21 )
      1. SDK Platform

### Install Java

  Install the latest version of java, if not already installed.

### Install Gstreamer SDK 
  
  Install Gstreamer SDK from http://gstreamer.freedesktop.org/download/

  Download URL: https://gstreamer.freedesktop.org/data/pkg/android/1.6.0/gstreamer-1.0-android-armv7-1.6.0.tar.bz2

  ```
  gstreamer-1.0-android-armv7-1.6.0.tar.bz2
  ```

  1. Once downloaded unzip the file to a folder.

  ```
  %>tar xvfj gstreamer-1.0-android-armv7-1.6.0.tar.bz2
  ```

## Command line build instructions

  1. Clone repository

  ```
  %>git clone https://github.com/ATLFlight/drone-controller.git
  ```
  1. Set up the environment, and install gradle if necessary:
  
  ```
  $ source env-setup.sh
  ```
  1. Create the **local.properties** file at the top-level of the dronecontroller git root.  This allows gradle to know where the Android NDK, Android SDK
     and Gstreamer SDK are located.
     Example:

     ```
      gst.dir=<path_to_gstreamer_1.6.0_unpacked_path>
      ndk.dir=<path_to_ndk_r10e_unpacked_path>
      sdk.dir=<path_to_installed_android_sdk_path>
    ```
    **NOTE**: For the "sdk.dir" use the "SDK path" as displayed when running the "android" app as part of the "Install Android SDK" section.

  1. Tell gradle to build the app
  
  ```
  $ gradle assemble
  ```
  
  1. The generated apk file will be located at:

  ```
  <drone_controller_git_root>/app/build/outputs/apk/app-debug.apk
  ```

## Installation

  Use the adb install command to install the apk on the Android Tablet

  ```
  %>adb install <path_to_dronecontroller_apk>/app-debug.apk
  ```

# Usage

Refer to [UserGuide](UserGuide.md) for DroneController usage information