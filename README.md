# Raspberry Pi Serving Robot

A prototype autonomous serving robot using a Raspberry Pi controller, Arduino-based line following, QR-code table identification, a Flask ordering backend, and a staff dashboard with live camera monitoring.

> **Prototype notice:** This repository documents the supplied project files. Hardware behavior, wiring, camera streaming, and order flow must be tested on the actual robot before operation. Test with the wheels raised, at low speed, and with an accessible physical power disconnect.

## Repository layout

```text
.
├── arduino/
│   └── serving_robot_current_existing.ino
├── raspberry_pi/
│   ├── serving_robot.py
│   └── requirements.txt
└── serving_robot_project/
    └── server/
        ├── app.py
        ├── database.py
        └── templates/
            ├── admin.html
            └── customer.html
```

## System overview

- **Arduino:** Drives the motors through the TB6612FNG, reads five IR line sensors, and monitors an HC-SR04 obstacle sensor. It accepts `GO` and `STOP` serial commands and reports status messages.
- **Raspberry Pi:** Runs the main controller, communicates with the Arduino over USB serial, retrieves and claims orders from Flask, detects table QR codes using the camera, and provides an MJPEG camera stream for the staff dashboard.
- **Flask server:** Provides customer ordering and staff/order APIs. SQLite is initialized by the application.
- **Staff dashboard:** Displays order status and can connect to the Pi's live camera stream.

## Hardware noted in source

- Raspberry Pi with a camera supported by Picamera2
- Arduino Uno (verify board compatibility with the supplied sketch)
- TB6612FNG motor driver and two DC motors
- Five IR line sensors
- HC-SR04 ultrasonic sensor
- USB serial connection between Raspberry Pi and Arduino
- Speaker/audio output if voice prompts are used

Confirm every pin, voltage, common ground, motor supply, and sensor output against the actual assembled hardware before powering the robot. Code comments are not a substitute for an electrical wiring check.

## Software requirements

### Flask server

From `serving_robot_project/server`, create a virtual environment and install Flask:

```bash
python -m venv .venv
# Windows PowerShell: .venv\Scripts\Activate.ps1
# Linux/macOS: source .venv/bin/activate
python -m pip install flask
python app.py
```

The server binds to `0.0.0.0:5000`. On the server computer, open `http://127.0.0.1:5000/` and `http://127.0.0.1:5000/admin`. Keep the server and robot on a trusted local network.

### Raspberry Pi

Install OS-managed camera and OpenCV packages where available (package names depend on the Raspberry Pi OS release):

```bash
sudo apt update
sudo apt install -y python3-picamera2 python3-opencv python3-pil python3-serial python3-requests
```

Set the Flask server URL to the reachable LAN address of the computer running Flask:

```bash
export ROBOT_SERVER_URL=http://<SERVER-LAN-IP>:5000
python3 serving_robot.py
```

The source defaults to `http://DELL.local:5000`; override it with `ROBOT_SERVER_URL` when the hostname or IP differs. Do not commit private network identifiers.

## Camera stream and dashboard

The Pi controller exposes the camera stream on port `8080`. After starting it, find the Pi address with `hostname -I`, then check:

- Health endpoint: `http://<PI-IP>:8080/health`
- MJPEG stream: `http://<PI-IP>:8080/stream.mjpg`

In the staff dashboard, enter the MJPEG URL in the camera URL field and connect. The Pi, server, and staff device must be able to communicate over the LAN.

**Security:** The camera stream is unauthenticated HTTP and is intended only for a trusted local network. Do not expose port 8080 directly to the internet. The Flask app has no staff authentication; do not expose it publicly without authentication, authorization, HTTPS, and deployment hardening.

## Run and test safely

1. Inspect wiring and power connections against the actual hardware.
2. Lift or secure the robot so wheels cannot propel it. Keep the physical emergency power disconnect accessible.
3. Verify sensor outputs on the normal line, background, intersections, and QR target locations.
4. Verify Arduino serial messages and motor direction at low speed.
5. Start Flask and test the customer page and staff dashboard.
6. Confirm the Pi can reach the Flask API and communicate with the Arduino.
7. Test QR recognition, obstacle stopping, table stop behavior, order-state updates, and camera streaming individually before a supervised end-to-end trial.

## Known limitations

- This project has not been independently validated on the physical robot in this repository preparation.
- Camera access, QR matching, serial port discovery, motor direction, and sensor calibration depend on hardware and OS configuration.
- The MJPEG stream has no authentication and should remain LAN-only.
- The Flask server binds to all interfaces for LAN use but does not provide staff login. Debug mode is disabled in the committed copy; this alone does not make the application production-ready.
- No automatic return-to-kitchen or arbitrary map-based navigation is documented by the supplied source.

## Contributing

Open an issue with hardware details, logs, and reproducible steps. Never include Wi-Fi passwords, API keys, customer data, or private camera footage in issues.
