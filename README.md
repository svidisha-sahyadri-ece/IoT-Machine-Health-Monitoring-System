# IoT Based Multi-Sensor Machine Health Monitoring System

## 📌 Overview
This project presents an IoT-based real-time machine health monitoring system designed for predictive maintenance in industrial motors. The system uses multiple sensors to monitor mechanical, thermal and electrical parameters simultaneously.

The system uses dual ESP32 architecture for high speed data acquisition and wireless cloud transmission. Signal processing and machine health index calculation are performed using MATLAB.

---

## 🚀 Features
- Multi-sensor real time motor monitoring
- Dual ESP32 architecture (Sensor Node + Gateway Node)
- Wireless data transfer using ESP-NOW
- Cloud data storage using Firebase
- MATLAB based signal processing (FFT, RMS, Kurtosis, Crest Factor)
- Machine Health Index calculation
- Real-time dashboard visualization

---

## 🧠 Parameters Monitored
- Vibration (ADXL345)
- Temperature (DS18B20)
- Current (ACS712)
- Voltage (ZMPT101B)
- RPM Sensor

---

## 🏗 System Architecture
Sensor Node ESP32 collects high speed sensor data  
⬇  
Gateway ESP32 transmits data to Firebase Cloud  
⬇  
MATLAB performs signal processing and health score computation  
⬇  
Dashboard shows real time machine health  

---

## 🛠 Technologies Used
- ESP32
- ESP-NOW Communication
- Firebase Realtime Database
- MATLAB Signal Processing
- IoT Cloud Monitoring

---

## 📊 Signal Processing Methods
- Fast Fourier Transform (FFT)
- Root Mean Square (RMS)
- Crest Factor
- Kurtosis Analysis

---

## 📈 Applications
- Predictive Maintenance
- Industrial Motor Monitoring
- Smart Factory Systems
- Condition Based Maintenance

---

## 🔮 Future Scope
- Edge AI based fault detection
- Adaptive threshold based monitoring
- Industrial deployment scaling
- Machine learning based fault classification

---

## 👩‍💻 Author
S Vidisha  
Electronics and Communication Engineering  
