# SKYMASTER - Advanced AI Drone with Orange Pi 5 Pro

A comprehensive guide to building a fully autonomous drone with advanced features such as **Follow-Me**, **Obstacle Avoidance**, **Gesture Control**, and **Voice Commands**. This project demonstrates the integration of AI, computer vision, and sensor technologies with the **Orange Pi 5 Pro** and **Pixhawk 6X**, creating an advanced drone for research, learning, or real-world applications.

---

## 📌 Overview

This project transforms an **Orange Pi 5 Pro** into the brain of an advanced AI-powered drone, integrating:

- ✅ **Follow-Me Mode**: Automatically follow a target using computer vision tracking.
- ✅ **Obstacle Avoidance**: Detect and avoid obstacles in real time utilizing LiDAR and AI algorithms.
- ✅ **Gesture Control**: Control the drone through hand-tracking gestures.
- ✅ **Voice Commands**: Execute voice-based commands (optional).

With this guide, you'll learn how to set up the hardware, configure the software, and implement AI features to create a robust autonomous drone.

This guide is divided into two sections to accommodate both **MacOS** and **Windows** users.

---

## 📂 Key Hardware Components

| **Component**           | **Purpose**                                                                 |
|--------------------------|-----------------------------------------------------------------------------|
| **[Orange Pi 5 Pro](https://www.amazon.com/Orange-Pi-Pro-Frequency-Bluetooth/dp/B0CSJVDL5G)** | Onboard computer for AI models and sensor processing               |
| **[reCamera](https://www.seeedstudio.com/reCamera-2002w-64GB-p-6249.html)**         | Captures video for follow-me and gesture control                   |
| **[ReSpeaker USB Mic Array](https://www.seeedstudio.com/ReSpeaker-USB-Mic-Array-p-4247.html)** | Captures voice commands (optional)                                 |
| **[TFMini Plus LiDAR](https://www.amazon.com/Stemedu-0-1m-12m-Distance-Detection-Waterproof/dp/B07L8B2FVK)** | Measures distance for obstacle avoidance                           |
| **[Pixhawk 6X](https://holybro.com/collections/x500-kits/products/px4-development-kit-x500-v2)** | Flight controller for navigation                                   |

---

## 🛠️ Hardware Setup

### **1. Orange Pi 5 Pro Connections**

| **Component**   | **Connection**        | **Notes**                                                                 |
|------------------|-----------------------|---------------------------------------------------------------------------|
| **reCamera**    | USB 3.0 or MIPI CSI   | Test with `v4l2-ctl --list-devices`.                                      |
| **ReSpeaker Mic**| USB 3.0              | Verify with `pyaudio`.                                                   |
| **TFMini LiDAR** | UART (`/dev/ttyS0`)   | Enable UART in `raspi-config`.                                           |
| **Pixhawk 6X**   | UART (`/dev/ttyAMA0`) | Baudrate: 57600.                                                         |

### **2. Pixhawk 6X Wiring**

| **Pixhawk Port** | **Orange Pi Port** |
|-------------------|--------------------|
| TELEM2 TX         | RX (Orange Pi)     |
| TELEM2 RX         | TX (Orange Pi)     |
| GND               | GND                |

---

## 💻 Software Installation

### For MacOS Users

#### **1. Ground Station Setup with QGroundControl**

- **Purpose**: Configure and control the Pixhawk 6X flight controller.
- **Download**: [QGroundControl.dmg](https://docs.qgroundcontrol.com/master/en/getting_started/download_and_install.html)
- **Installation**:
   ```bash
   wget https://s3-us-west-2.amazonaws.com/qgroundcontrol/latest/QGroundControl.dmg
   hdiutil attach QGroundControl.dmg
   cp -r /Volumes/QGroundControl/QGroundControl.app /Applications/
   ```

---

### For Windows Users

#### **1. Ground Station Setup with Mission Planner**

- **Purpose**: A Windows-compatible flight control software for Pixhawk.
- **Download**: [Mission Planner](https://firmware.ardupilot.org/Tools/MissionPlanner/)
- **Installation**:
  1. Download the `.exe` file.
  2. Run the installer with administrator privileges.
  3. Once installed, configure Mission Planner:
     - Select **COM Port** (check in Device Manager).
     - Set the **baud rate** to `57600`.

---

### **2. Orange Pi 5 Pro Setup (Ubuntu 22.04)**

#### **Step 1: Flash Ubuntu**
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
sudo apt install -y python3-pip git v4l-utils libportaudio2
pip3 install opencv-python pyserial pyaudio pymavlink dronekit tensorflow
```

---

## 🚀 Sensor Configuration

### **1. reCamera Setup**

```bash
# Check camera
v4l2-ctl --list-devices
```

**Python Test Script** (`camera_test.py`):
```python
import cv2
cap = cv2.VideoCapture(0)
while True:
    ret, frame = cap.read()
    cv2.imshow("Frame", frame)
    if cv2.waitKey(1) == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```
Run:
```bash
python3 camera_test.py
```

---

### **2. TFMini LiDAR**

```bash
# Enable UART
sudo raspi-config  # Interface Options → Serial → Disable shell, enable hardware
sudo reboot
```

**Python Test Script** (`lidar_test.py`):
```python
import serial
ser = serial.Serial('/dev/ttyS0', 115200)
while True:
    data = ser.read(9)
    if data[0] == 0x59 and data[1] == 0x59:
        print(f"Distance: {data[2] + data[3]*256} cm")
```

---

### **3. Pixhawk MAVLink**

#### For MacOS:
```bash
# Start MAVProxy bridge
pip3 install mavproxy
mavproxy.py --master=/dev/ttyAMA0 --baudrate=57600 --out=udp:127.0.0.1:14550
```

In **QGroundControl**:
- Go to **Comm Links** → Add UDP Connection → Port: `14550`.

#### For Windows:
```bash
# Start MAVProxy bridge
mavproxy.py --master=/dev/ttyAMA0 --baudrate=57600 --out=udp:192.168.1.100:14550
```
Replace `192.168.1.100` with your Windows PC's IP address.

In **Mission Planner**:
- Add a UDP connection on port `14550`.

---

## 🤖 AI Features Implementation (Python Code)

### **1. Follow-Me Mode**
```python
# Object tracking (OpenCV + MAVLink)
import cv2
from pymavlink import mavutil

drone = mavutil.mavlink_connection('udp:127.0.0.1:14550')
cap = cv2.VideoCapture(0)
tracker = cv2.TrackerCSRT_create()

while True:
    ret, frame = cap.read()
    # Add tracking logic here
    drone.mav.command_long_send(...)  # Send movement commands
```

---

### **2. Obstacle Avoidance**
```python
# LiDAR + MAVLink
import serial
from pymavlink import mavutil

lidar = serial.Serial('/dev/ttyS0', 115200)
drone = mavutil.mavlink_connection('udp:127.0.0.1:14550')

while True:
    distance = read_lidar()  # Custom function
    if distance < 100:  # 1m threshold
        drone.mav.command_long_send(...)  # Evasive maneuver
```

---

### **3. Gesture Control**
```python
# Gesture recognition using OpenCV and CNN
import cv2
from keras.models import load_model

model = load_model('gesture_model.h5')
cap = cv2.VideoCapture(0)

while True:
    ret, frame = cap.read()
    # Preprocess frame for CNN
    prediction = model.predict(frame)
    # Map prediction to drone commands
```

---

### **4. Voice Commands (Optional)**
```python
# Speech recognition
import speech_recognition as sr
from pymavlink import mavutil

r = sr.Recognizer()
mic = sr.Microphone()

with mic as source:
    print("Listening for commands...")
    audio = r.listen(source)
    command = r.recognize_google(audio)
    # Map command to MAVLink
```

---

## 🔧 Debugging Tips

1. **Camera not detected?**
```bash
ls /dev/video*  # Check device nodes
sudo modprobe v4l2loopback
```

2. **MAVLink not connecting?**
```bash
sudo chmod 666 /dev/ttyAMA0  # Fix permissions
```

3. **LiDAR not responding?**
- Ensure UART is enabled in `raspi-config`.
- Verify connections and baud rate.

---

## 💡 Challenges & Solutions

| **Challenge**           | **Solution**                                 |
|--------------------------|---------------------------------------------|
| **Power Consumption**    | Use a high-capacity LiPo battery.           |
| **Thermal Management**   | Install a heatsink/fan for the Orange Pi.   |
| **Real-Time Performance**| Optimize AI models for low latency.         |

---

## 📧 Contact

Feel free to reach out for questions or collaborations:

- **Email**: [sanjaykathula7@gmail.com](mailto:sanjaykathula7@gmail.com)
- **GitHub**: [github.com/sanjayka1999](https://github.com/sanjayka1999)

---

🚀 **Let’s build the future of autonomous drones!**
