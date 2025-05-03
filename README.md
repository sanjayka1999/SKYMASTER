# SKYMASTER
Advanced Drone Building Guide 
# Advanced Drone Building Using AI

This repository contains all the necessary resources, code, and guidelines to build an advanced drone with features like **follow-me**, **obstacle avoidance**, **gesture control**, and **voice command**. The project leverages cutting-edge technologies, including the **Orange Pi 5 pro**, **reCamera**, **ReSpeaker USB Mic Array**, **TFMini Plus LiDAR**, and **Pixhawk 6X**.

---

## 📌 Goals

The primary objective of this project is to design and implement a fully functional drone that integrates AI-based features with robust hardware capabilities. The drone will support the following functionalities:
1. **Follow-Me**: Automatically follow a target based on visual data.
2. **Obstacle Avoidance**: Detect and avoid obstacles in real time.
3. **Gesture Control**: Respond to specific hand gestures for control.
4. **Voice Command**: Execute commands based on voice inputs (optional).

---

## 📂 Hardware Components

Below is the list of hardware components used in this project:

1. **[Orange Pi 5 pro]([https://www.amazon.com/Orange-Pi-Pro-Frequency-Bluetooth/dp/B0CSJVDL5G?tag=usdeshoppin04-20&th=1])**: The onboard computer for running AI models and processing sensor data.
2. **[reCamera](https://www.seeedstudio.com/reCamera-2002w-64GB-p-6249.html)**: Captures video for follow-me and gesture control features.
3. **[ReSpeaker USB Mic Array](https://www.seeedstudio.com/ReSpeaker-USB-Mic-Array-p-4247.html)**: Captures voice commands for voice-based control.
4. **[TFMini Plus LiDAR](https://www.amazon.com/Stemedu-0-1m-12m-Distance-Detection-Waterproof/dp/B07L8B2FVK)**: Measures distance and enables obstacle avoidance.
5. **[Pixhawk 6X Flight Controller Kit](https://holybro.com/collections/x500-kits/products/px4-development-kit-x500-v2?variant=43018371629245)**: Handles flight control and navigation.

---

## 🛠️ Hardware Connections

### **Orange Pi 5 Connections**
1. **reCamera**:
   - Connect to the **USB 3.0** or **MIPI CSI** interface of the Orange Pi 5.
   - Mount the camera on the drone frame with an unobstructed view of the target.
2. **ReSpeaker USB Mic Array**:
   - Connect to the **USB 3.0** port of the Orange Pi 5.
   - Mount it in a position that optimizes capture of voice commands.
3. **TFMini Plus LiDAR**:
   - Connect to the **UART** or **I2C** interface of the Orange Pi 5.
   - Mount facing forward for obstacle detection or downward for ground-based distance measurement.
4. **Pixhawk 6X**:
   - Connect via **UART** or **I2C** for communication.
   - Use a telemetry radio (e.g., SiK Radio) for wireless communication.
5. **Power Supply**:
   - Use a 5V power module connected to the drone's battery to supply power to the Orange Pi 5.

### **Pixhawk 6X Connections**
1. **Motors and ESCs**: Connect motors and ESCs to the Pixhawk 6X motor outputs.
2. **GPS Module**: Connect a GPS module (e.g., u-blox M8N) to the Pixhawk for navigation.
3. **Telemetry Radio**: Connect a telemetry radio for communication with the ground station.

---

## 💻 Software Setup

### **Step 1: Operating System Installation**
- Install a compatible Linux distribution like **Ubuntu** or **Armbian** on the Orange Pi 5.

### **Step 2: Install Software Dependencies**
1. Update the system:
   ```bash
   sudo apt update && sudo apt upgrade
   ```
2. Install Python and its required libraries:
   ```bash
   sudo apt install python3-pip
   pip3 install numpy opencv-python pyserial pyaudio tensorflow pymavlink
   ```
3. Install AI frameworks:
   ```bash
   pip3 install tensorflow
   ```

---

## 🧑‍💻 Configuration for Sensors and Features

### **1. reCamera**
Capture video feeds using OpenCV:
```python
import cv2

cap = cv2.VideoCapture(0)  # Adjust camera index if needed
while True:
    ret, frame = cap.read()
    cv2.imshow("Frame", frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```

### **2. ReSpeaker USB Mic Array**
Capture voice commands using PyAudio:
```python
import pyaudio
import wave

FORMAT = pyaudio.paInt16
CHANNELS = 2
RATE = 16000
CHUNK = 1024
RECORD_SECONDS = 5
OUTPUT_FILENAME = "output.wav"

audio = pyaudio.PyAudio()
stream = audio.open(format=FORMAT, channels=CHANNELS, rate=RATE, input=True, frames_per_buffer=CHUNK)
frames = []

for i in range(0, int(RATE / CHUNK * RECORD_SECONDS)):
    data = stream.read(CHUNK)
    frames.append(data)

stream.stop_stream()
stream.close()
audio.terminate()

with wave.open(OUTPUT_FILENAME, 'wb') as wf:
    wf.setnchannels(CHANNELS)
    wf.setsampwidth(audio.get_sample_size(FORMAT))
    wf.setframerate(RATE)
    wf.writeframes(b''.join(frames))
```

### **3. TFMini Plus LiDAR**
Read distance data using PySerial:
```python
import serial

ser = serial.Serial('/dev/ttyUSB0', 115200)  # Adjust the port and baud rate
while True:
    data = ser.read(9)
    if data[0] == 0x59 and data[1] == 0x59:
        distance = data[2] + data[3] * 256
        print(f"Distance: {distance} cm")
```

### **4. Pixhawk 6X Communication**
Send MAVLink commands to the Pixhawk:
```python
from pymavlink import mavutil

master = mavutil.mavlink_connection('/dev/ttyAMA0', baud=57600)
master.mav.command_long_send(
    master.target_system, master.target_component,
    mavutil.mavlink.MAV_CMD_NAV_TAKEOFF, 0, 0, 0, 0, 0, 0, 0, 10  # Takeoff to 10 meters
)
```

---

## 🚀 Workflow for Advanced Features

1. **Follow-Me**:
   - Use the reCamera to detect and track the target.
   - Send follow-me commands to the Pixhawk using MAVLink.

2. **Obstacle Avoidance**:
   - Use the TFMini Plus LiDAR to detect obstacles.
   - Adjust the drone’s flight path in real time using MAVLink commands.

3. **Gesture Control**:
   - Use the reCamera to recognize specific hand gestures.
   - Translate gestures into commands for the Pixhawk.

4. **Voice Command**:
   - Use the ReSpeaker USB Mic Array to capture voice inputs.
   - Convert speech to text and send commands to the Pixhawk.

---

## 💡 Challenges and Solutions

1. **Power Consumption**:
   - Ensure the drone’s battery can support all components. Use a high-capacity LiPo battery.
2. **Thermal Management**:
   - Install a heatsink or cooling fan for the Orange Pi 5 to prevent overheating.
3. **Real-Time Performance**:
   - Optimize AI models to minimize latency during data processing.

---

## 🏆 Conclusion

By following this guide, you’ll successfully build a feature-rich drone capable of **follow-me**, **obstacle avoidance**, **gesture control**, and **voice command** functionalities. This project showcases the integration of hardware and software in a real-world application, making it an excellent final year project.

---

## 🤝 Contribution and Contact

Feel free to contribute to this project! Open a pull request or contact me for questions:
- **Email**: [your_email@example.com]
- **GitHub**: [github.com/sanjayka1999](https://github.com/sanjayka1999)
