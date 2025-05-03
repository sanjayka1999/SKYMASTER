# SKYMASTER
Advanced Drone Building Guide

# Advanced Drone Building Using Single Board Computer

This repository contains all the necessary resources, code, and guidelines to build an advanced drone with features like **follow-me**, **obstacle avoidance**, **gesture control**, and **voice command**. The project leverages cutting-edge technologies, including the **Orange Pi 5 Pro**, **reCamera**, **ReSpeaker USB Mic Array**, **TFMini Plus LiDAR**, and **Pixhawk 6X**.

---

## 📌 Goals

The primary objective is to design and implement a fully functional drone that integrates AI-based features with robust hardware capabilities. The drone will support:

- **Follow-Me**: Automatically follow a target based on visual data.
- **Obstacle Avoidance**: Detect and avoid obstacles in real time.
- **Gesture Control**: Respond to specific hand gestures for control.
- **Voice Command**: Execute commands based on voice inputs (optional).

---

## 📂 Hardware Components

| **Component**           | **Purpose**                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **[Orange Pi 5 Pro](https://www.amazon.com/Orange-Pi-Pro-Frequency-Bluetooth/dp/B0CSJVDL5G)** | Onboard computer for AI models and sensor processing               |
| **[reCamera](https://www.seeedstudio.com/reCamera-2002w-64GB-p-6249.html)**         | Captures video for follow-me and gesture control                   |
| **[ReSpeaker USB Mic Array](https://www.seeedstudio.com/ReSpeaker-USB-Mic-Array-p-4247.html)** | Captures voice commands (optional)                                 |
| **[TFMini Plus LiDAR](https://www.amazon.com/Stemedu-0-1m-12m-Distance-Detection-Waterproof/dp/B07L8B2FVK)** | Measures distance for obstacle avoidance                           |
| **[Pixhawk 6X](https://holybro.com/collections/x500-kits/products/px4-development-kit-x500-v2)** | Flight controller for navigation                                   |

---

## 🛠️ Hardware Connections

### **Orange Pi 5 Pro Connections**
1. **reCamera**:
   - Connect to **USB 3.0** or **MIPI CSI** interface.
   - Mount with an unobstructed view.
2. **ReSpeaker USB Mic Array (Optional)**:
   - Connect to **USB 3.0** port.
3. **TFMini Plus LiDAR**:
   - Connect to **UART** (e.g., `/dev/ttyS0`) or **I2C**.
   - Mount facing forward for obstacle detection.
4. **Pixhawk 6X**:
   - Connect via **UART** (e.g., `/dev/ttyAMA0`) for MAVLink communication.
   - Use **Telemetry Radio (SiK Radio)** for wireless communication.
5. **Power Supply**:
   - Use a 5V power module from the **Power Distribution Board (PDB)**.

---

## 💻 Software Setup (MacOS & Orange Pi 5 Pro)

### **1. Software for MacOS (Ground Station Setup)**

#### **[QGroundControl](https://docs.qgroundcontrol.com/master/en/qgc-user-guide/getting_started/download_and_install.html)**  
- **Purpose**: For configuring and controlling the Pixhawk 6X flight controller.
- **Download**: [QGroundControl.dmg](https://docs.qgroundcontrol.com/master/en/getting_started/download_and_install.html)
- **Installation**:
   ```bash
   hdiutil attach ~/Downloads/QGroundControl.dmg
   cp -r /Volumes/QGroundControl/QGroundControl.app /Applications/
   ```

#### **[Mission Planner](https://ardupilot.org/planner/docs/mission-planner-installation.html) (Alternative, Windows Only)**  
- **Purpose**: For configuring autonomous vehicles.
- **Note**: Not required for MacOS.

---

### **2. Orange Pi 5 Pro Setup (Ubuntu OS)**

#### **Step 1: Flash Ubuntu on Orange Pi 5 Pro**
1. **Download Ubuntu Image**:  
   Visit the [Orange Pi 5 Pro Downloads](http://www.orangepi.org/html/hardWare/computerAndMicrocontrollers/service-and-support/Orange-Pi-5-Pro.html).
2. **Flash Image**:  
   Use [BalenaEtcher](https://www.balena.io/etcher/) to flash the image to an SD card.
3. **First Boot & Login**:
   - Insert the SD card, connect HDMI, keyboard, and power.
   - Default credentials:
     - **Username**: orangepi  
     - **Password**: orangepi  

#### **Step 2: Install Dependencies**
Run the following commands in the Orange Pi terminal:
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Python & Libraries
sudo apt install python3-pip git -y
pip3 install numpy opencv-python pyserial pyaudio tensorflow pymavlink

# Install MAVLink tools
sudo apt install ros-humble-mavros ros-humble-mavros-extras -y
pip3 install PyYAML mavproxy dronekit
```

---

### **3. Configure Sensors and Features**

#### **1. reCamera Setup**
```bash
# Test camera
sudo apt install v4l-utils -y
v4l2-ctl --list-devices
```
**Python Script**:
```python
import cv2
cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    cv2.imshow("Frame", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```

#### **2. ReSpeaker USB Mic Array (Optional)**
```bash
# Install dependencies
sudo apt install libportaudio2 -y
pip3 install pyaudio
```
**Python Script**:
```python
import pyaudio
p = pyaudio.PyAudio()
for i in range(p.get_device_count()):
    print(p.get_device_info_by_index(i))
```

#### **3. TFMini Plus LiDAR**
```bash
# Enable UART
sudo raspi-config  # Enable serial port (disable login shell)
```
**Python Script**:
```python
import serial
ser = serial.Serial('/dev/ttyS0', 115200)
while True:
    data = ser.read(9)
    if data[0] == 0x59 and data[1] == 0x59:
        distance = data[2] + data[3] * 256
        print(f"Distance: {distance} cm")
```

#### **4. Pixhawk 6X MAVLink Connection**
```bash
# Install MAVProxy
pip3 install mavproxy
```
**Connect to Pixhawk via UART**:
```bash
mavproxy.py --master=/dev/ttyAMA0 --baudrate=57600 --out=udp:127.0.0.1:14550
```
In **QGroundControl**:
- Go to **Comm Links** → Add UDP Connection (Port: `14550`).

---

## 🚀 Workflow for Advanced Features

1. **Follow-Me Mode**:
   - Use OpenCV for object tracking.
   - Send MAVLink commands to the Pixhawk.

2. **Obstacle Avoidance**:
   - Use TFMini LiDAR for real-time distance measurement.
   - Adjust flight path using MAVLink.

3. **Gesture Control**:
   - Train a CNN model for gesture recognition.
   - Map gestures to drone commands.

4. **Voice Command (Optional)**:
   - Use the SpeechRecognition library.
   - Convert voice to MAVLink commands.

---

## 💡 Challenges & Solutions

| **Challenge**           | **Solution**                                 |
|--------------------------|---------------------------------------------|
| **Power Consumption**    | Use a high-capacity LiPo battery.           |
| **Thermal Management**   | Install a heatsink/fan for the Orange Pi.   |
| **Real-Time Performance**| Optimize AI models for low latency.         |

---

## 🏆 Conclusion

This guide provides a step-by-step process to build an AI-powered drone with **follow-me**, **obstacle avoidance**, **gesture control**, and **voice command** capabilities using the Orange Pi 5 Pro and Pixhawk 6X.

---

## 🤝 Contribution & Contact

Feel free to contribute to this project! Open a pull request or contact me for questions:
- **Email**: [sanjaykathula7@gmail.com](mailto:sanjaykathula7@gmail.com)
- **GitHub**: [github.com/sanjayka1999](https://github.com/sanjayka1999)
