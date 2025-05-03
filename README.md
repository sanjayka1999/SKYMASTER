SKYMASTER - Advanced AI Drone with Orange Pi 5 Pro
A professional-grade guide for building an autonomous drone with dual-platform support

Table of Contents
MacOS Configuration

Hardware Setup

Software Installation

Sensor Configuration

AI Features

Windows Configuration

Hardware Setup

Software Installation

Sensor Configuration

AI Features

Troubleshooting

Contributing

🍎 MacOS Configuration
📦 Hardware Setup (MacOS)
Required Components
Component	Purpose	Documentation
Orange Pi 5 Pro	AI Processing	Datasheet
Pixhawk 6X	Flight Control	User Guide
TFMini Plus LiDAR	Obstacle Detection	Datasheet
Connection Diagram
Orange Pi 5 Pro
├── USB 3.0 → reCamera
├── UART0 → TFMini LiDAR (TX/RX)
└── UART1 → Pixhawk TELEM2 (MAVLink)
💾 Software Installation (MacOS)
1. QGroundControl Setup
bash
# Download and install
wget https://s3-us-west-2.amazonaws.com/qgroundcontrol/latest/QGroundControl.dmg
hdiutil attach QGroundControl.dmg
cp -r /Volumes/QGroundControl/QGroundControl.app /Applications/
2. Orange Pi 5 Pro Setup
Step 1: Flash Ubuntu
Download Ubuntu 22.04 Server

Flash using BalenaEtcher

Step 2: Initial Configuration
bash
# Connect via SSH
ssh orangepi@<local_ip>

# Install dependencies
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y python3.10-venv git v4l-utils libportaudio2
Step 3: Python Environment
bash
python3 -m venv skymaster
source skymaster/bin/activate
pip install --upgrade pip
pip install opencv-python==4.7.0.72 pyserial==3.5 pymavlink==2.4.37
🔌 Sensor Configuration (MacOS)
1. reCamera Setup
python
# camera_test.py
import cv2

def test_camera():
    cap = cv2.VideoCapture(0, cv2.CAP_V4L2)
    if not cap.isOpened():
        raise RuntimeError("Camera not detected!")
    
    while True:
        ret, frame = cap.read()
        cv2.imshow("Feed", frame)
        if cv2.waitKey(1) == ord('q'):
            break
            
    cap.release()
    cv2.destroyAllWindows()

if __name__ == "__main__":
    test_camera()
2. MAVLink Communication
bash
# Start MAVProxy router
mavproxy.py --master=/dev/ttyAMA0 --baudrate=57600 \
            --out=udpin:0.0.0.0:14550 \
            --out=tcpin:0.0.0.0:5760
🧠 AI Features Implementation (MacOS)
Follow-Me Algorithm
python
# follow_me.py
import cv2
from pymavlink import mavutil

class Follower:
    def __init__(self):
        self.drone = mavutil.mavlink_connection('udpin:0.0.0.0:14550')
        self.tracker = cv2.TrackerCSRT_create()
        
    def track_target(self):
        cap = cv2.VideoCapture(0)
        # Implementation logic here
🪟 Windows Configuration
📦 Hardware Setup (Windows)
Special Considerations
Use USB-to-TTL Converter for direct Pixhawk connection

Configure Windows Subsystem for Linux for Orange Pi management

💾 Software Installation (Windows)
1. Mission Planner Setup
Download from ArduPilot Official Site

Install with Administrator Privileges

2. Orange Pi Management
powershell
# In PowerShell
wsl --install -d Ubuntu
ssh orangepi@<ip>
🔌 Sensor Configuration (Windows)
MAVProxy Alternative
powershell
python -m pip install pymavlink
python -c "from pymavlink import mavutil; conn = mavutil.mavlink_connection('com3:57600')"
🧠 AI Features Implementation (Windows)
Gesture Control
python
# gesture.py (Windows-compatible)
import cv2
import pyvirtualcam

with pyvirtualcam.Camera(width=640, height=480, fps=30) as cam:
    cap = cv2.VideoCapture(0)
    while True:
        ret, frame = cap.read()
        cam.send(frame)
🔧 Troubleshooting
MacOS
QGroundControl Connection Issues:

bash
lsof -i :14550  # Check port usage
Camera Permissions:

bash
sudo chmod 666 /dev/video*
Windows
COM Port Conflicts: Use [Device Manager] to resolve conflicts

Driver Issues: Install FTDI Drivers

🤝 Contributing
Fork the GitHub Repo

Submit PRs to dev branch

Follow Contribution Guidelines

📧 Contact: sanjaykathula7@gmail.com
🔗 Documentation: SKYMASTER Wiki

SKYMASTER Banner

Last Updated: October 2023

This version features:
✅ Strict platform separation
✅ Hyperlinked references
✅ Version-pinned dependencies
✅ Professional code formatting
✅ Troubleshooting matrices
✅ Contributor guidelines
