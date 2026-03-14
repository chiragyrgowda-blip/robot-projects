# RoboBot - Full Setup Guide

## Wiring Diagram

### Arduino Mega → L298N Motor Driver (×2)
```
Arduino Mega      L298N #1 (Left Motors)    L298N #2 (Right Motors)
Pin 5  (ENA)  →  ENA                        -
Pin 22 (IN1)  →  IN1                        -
Pin 23 (IN2)  →  IN2                        -
Pin 6  (ENB)  →  -                          ENA
Pin 24 (IN3)  →  -                          IN1
Pin 25 (IN4)  →  -                          IN2

L298N #1 OUT1/OUT2 → Left Front Motor
L298N #1 OUT3/OUT4 → Left Rear Motor
L298N #2 OUT1/OUT2 → Right Front Motor
L298N #2 OUT3/OUT4 → Right Rear Motor

Power: 7.4V LiPo → L298N 12V pin (both drivers)
       L298N 5V out → Arduino VIN (or use separate 5V)
```

### Arduino Mega → Raspberry Pi
```
Arduino USB → Pi USB port
(Serial communication at 9600 baud, /dev/ttyUSB0)
```

### APDS-9960 → Raspberry Pi (I2C)
```
APDS-9960    Raspberry Pi
VCC       →  3.3V (Pin 1)
GND       →  GND  (Pin 6)
SDA       →  GPIO2 / SDA (Pin 3)
SCL       →  GPIO3 / SCL (Pin 5)
INT       →  GPIO4 (Pin 7) [optional interrupt]
```

### ESP32-CAM
- Powers via 5V USB or 3.3V from Pi
- Connects over WiFi (same network as Pi)
- Stream URL: http://<ESP32-IP>:81/stream

### USB Microphone → Raspberry Pi USB port
### Speaker → PAM8403 amp → Pi 3.5mm audio jack or GPIO PWM

## Software Setup (Raspberry Pi)

```bash
# Update system
sudo apt update && sudo apt upgrade -y
sudo apt install python3-pip python3-opencv espeak portaudio19-dev -y

# Python dependencies
pip3 install face_recognition opencv-python SpeechRecognition pyttsx3 pyserial smbus2

# Enable I2C
sudo raspi-config → Interface Options → I2C → Enable

# Find Arduino port
ls /dev/tty*  # Look for ttyUSB0 or ttyACM0

# Add user to dialout group (serial access)
sudo usermod -a -G dialout $USER
```

## Running the Robot

```bash
# 1. Upload arduino_motor.ino to Arduino Mega
# 2. Upload esp32cam_stream.ino to ESP32-CAM
# 3. Add face photos to known_faces/ folder (e.g. known_faces/Alice.jpg)
# 4. Update ARDUINO_PORT in robot_brain.py if needed
# 5. For ESP32-CAM, update the stream URL line in face_recognition_thread()
# 6. Run:
python3 robot_brain.py
```

## Voice Commands
| Say this           | Action          |
|--------------------|-----------------|
| "forward"          | Move forward    |
| "backward"         | Move backward   |
| "left"             | Turn left       |
| "right"            | Turn right      |
| "stop"             | Stop motors     |
| "fast"             | Increase speed  |
| "slow"             | Decrease speed  |
| "hello"            | Greeting        |
| "what can you do"  | Robot describes itself |

## Gesture Commands (APDS-9960)
| Gesture     | Action        |
|-------------|---------------|
| Swipe UP    | Move forward  |
| Swipe DOWN  | Move backward |
| Swipe LEFT  | Turn left     |
| Swipe RIGHT | Turn right    |

## Face Recognition
- Place one clear frontal photo per person in known_faces/
- Name the file exactly as you want the robot to say (e.g. "John.jpg")
- Robot will greet recognized faces automatically

## Cost Estimate (approx)
| Item              | Cost (INR) |
|-------------------|-----------|
| Raspberry Pi 4 2GB | ₹4,500   |
| Arduino Mega       | ₹800     |
| ESP32-CAM          | ₹400     |
| APDS-9960          | ₹350     |
| Robot chassis + motors | ₹1,200 |
| L298N ×2           | ₹400     |
| USB Mic + Speaker  | ₹600     |
| LiPo + Power bank  | ₹1,200   |
| Wires + misc       | ₹300     |
| **Total**          | **~₹9,750** |
