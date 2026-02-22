# IoT-Based Multi-Sensor Machine Health Monitoring System

This project presents a real-time machine condition monitoring system for predictive maintenance using multi-sensor data acquisition, wireless communication, and signal processing.

The system continuously monitors vibration, temperature, electrical parameters, and rotational speed of a motor and computes an overall machine health index.

---

## Features

- Real-time multi-parameter machine monitoring
- Dual ESP32 architecture for sensing and communication
- Wireless data transmission using ESP-NOW
- Cloud data logging
- Signal processing using MATLAB (FFT, RMS, statistical analysis)
- Machine health index computation
- Real-time health visualization dashboard

---

## System Architecture

The system consists of three layers:

1. Data Acquisition Layer – Sensors + ESP32
2. Communication Layer – Wireless data transfer
3. Analytics Layer – MATLAB processing
4. Visualization Layer – Dashboard

---

## Hardware Components

- ESP32 microcontrollers
- Vibration sensor
- Temperature sensor
- Current sensor
- Voltage sensor
- RPM sensor

---

## Software Used

- Arduino IDE (ESP32 programming)
- MATLAB (Signal processing)
- Firebase (Cloud storage)

---

## How the System Works

1. Sensors collect real-time machine parameters
2. ESP32 sensing node captures and preprocesses data
3. Gateway ESP32 transmits data to cloud
4. MATLAB processes data and extracts features
5. Machine health index is calculated
6. Results displayed on dashboard

---

## Results

The system successfully detects abnormal operating conditions using correlated multi-parameter analysis and provides early fault indication.

---

## Repository Structure

- ESP32 sensor node code
- ESP32 gateway code
- MATLAB signal processing scripts
- Hardware setup diagrams
- Experimental results

---

## Applications

- Predictive maintenance
- Industrial equipment monitoring
- Fault detection in rotating machines
- Industrial IoT systems

---

## Author

S Vidisha  
Electronics and Communication Engineering
