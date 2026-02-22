#include <OneWire.h>

#include <DallasTemperature.h>
#include <Wire.h>
#include <Adafruit_ADXL345_U.h>
#include <math.h>
#include <ZMPT101B.h>
#include "ACS712.h"
#include <WiFi.h>
#include <esp_now.h>

// ===== RECEIVER MAC ADDRESS =====
uint8_t receiverMAC[] = {0x00, 0x4B, 0x12, 0xEF, 0x53, 0x1C};//C0:49:EF:CB:AB:90//00:4B:12:EF:53:1C

// =================== DS18B20 ===================
#define ONE_WIRE_BUS 15
OneWire oneWire(ONE_WIRE_BUS);
DallasTemperature sensors(&oneWire);

// =================== ADXL345 ===================
Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified(12345);
float offsetX = 0, offsetY = 0, offsetZ = 0;
const int samplesPerSecond = 50;
const int sampleDelayMs = 1000 / samplesPerSecond;

// =================== ZMPT101B ===================
#define SENSITIVITY 650.9f
#define PIN_ZMPT 32
#define FREQ 50.0
ZMPT101B voltageSensor(PIN_ZMPT, FREQ);

// =================== RPM Sensor ===================
#define RPM_PIN 4
volatile unsigned long pulseCount = 0;
volatile unsigned long lastPulseTime = 0;
const unsigned long debounceInterval = 10;
const int pulsesPerRevolution = 2;
float calibrationFactor = 0.147;
float rpm = 0;

// =================== ACS712 ===================
#define ACS_PIN 34
#define VREF 3.3
#define ADC_RES 4095
#define ACS_SENSITIVITY 185
#define KNOWN_CURRENT 444.0
#define BASELINE_SAMPLES 2000
#define RUN_SAMPLES 500
ACS712 ACS(ACS_PIN, VREF, ADC_RES, ACS_SENSITIVITY);
float baseline_mA = 0.0;
float scaleFactor = 1.0;

// =================== Interrupt for RPM ===================
void IRAM_ATTR countPulse() {
  unsigned long now = millis();
  if (now - lastPulseTime > debounceInterval) {
    pulseCount++;
    lastPulseTime = now;
  }
}

// =================== ESP-NOW Callback ===================
void OnDataSent(const wifi_tx_info_t *info, esp_now_send_status_t status) {
  Serial.print("📤 Delivery Status: ");
  Serial.println(status == ESP_NOW_SEND_SUCCESS ? "✅ Success" : "❌ Fail");
}

// =================== Setup ===================
void setup() {
  Serial.begin(115200);
  delay(1000);

  // Initialize sensors
  sensors.begin();
  if (!accel.begin()) {
    Serial.println("❌ ADXL345 not detected! Check wiring.");
    while (1);
  }

  accel.setRange(ADXL345_RANGE_16_G);
  calibrateOffsets();

  voltageSensor.setSensitivity(SENSITIVITY);
  pinMode(RPM_PIN, INPUT);
  attachInterrupt(digitalPinToInterrupt(RPM_PIN), countPulse, FALLING);

  // ACS712 calibration
  Serial.println("\n=== ACS712 Auto Calibration ===");
  delay(3000);
  baseline_mA = readAverage_mA(BASELINE_SAMPLES);
  Serial.print("Baseline: "); Serial.println(baseline_mA);
  delay(8000);
  float loaded_mA = readAverage_mA(BASELINE_SAMPLES);
  float adj = loaded_mA - baseline_mA;
  if (adj < 0) adj = 0;
  scaleFactor = KNOWN_CURRENT / adj;
  Serial.print("Scale Factor: "); Serial.println(scaleFactor);

  // ===== ESP-NOW Initialization =====
  WiFi.mode(WIFI_STA);
  if (esp_now_init() != ESP_OK) {
    Serial.println("❌ ESP-NOW init failed!");
    while (1);
  }

  // Register send callback
  esp_now_register_send_cb(OnDataSent);

  // Add peer
  esp_now_peer_info_t peerInfo = {};
  memcpy(peerInfo.peer_addr, receiverMAC, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;

  if (esp_now_add_peer(&peerInfo) != ESP_OK) {
    Serial.println("❌ Failed to add peer!");
    while (1);
  }

  Serial.println("✅ ESP-NOW Transmitter Ready!");
}

// =================== Main Loop ===================
void loop() {
  unsigned long timestamp = millis();

  // Temperature
  sensors.requestTemperatures();
  float tempC = sensors.getTempCByIndex(0);

  // ================== ADXL Raw Burst ==================
  float burstX[samplesPerSecond];
  float burstY[samplesPerSecond];
  float burstZ[samplesPerSecond];

  for (int i = 0; i < samplesPerSecond; i++) {
    sensors_event_t event;
    accel.getEvent(&event);
    burstX[i] = event.acceleration.x - offsetX;
    burstY[i] = event.acceleration.y - offsetY;
    burstZ[i] = event.acceleration.z - offsetZ;
    delay(sampleDelayMs);
  }

  // Voltage
  float voltageV = voltageSensor.getRmsVoltage();

  // RPM
  noInterrupts();
  unsigned long count = pulseCount;
  pulseCount = 0;
  interrupts();
  rpm = (count / (float)pulsesPerRevolution) * 60.0 * calibrationFactor;

  // Current
  float raw = readAverage_mA(RUN_SAMPLES);
  float adj = raw - baseline_mA;
  if (adj < 0) adj = 0;
  float scaled = adj * scaleFactor;

  // Convert burst arrays to CSV strings
  String burstStrX = "";
  String burstStrY = "";
  String burstStrZ = "";
  for (int i = 0; i < samplesPerSecond; i++) {
    burstStrX += String(burstX[i], 3);
    burstStrY += String(burstY[i], 3);
    burstStrZ += String(burstZ[i], 3);
    if (i < samplesPerSecond - 1) {
      burstStrX += ",";
      burstStrY += ",";
      burstStrZ += ",";
    }
  }

  // Create CSV data string including bursts
  String csv = String(timestamp) + "," + String(tempC, 2) + "," +
               "[" + burstStrX + "]," +
               "[" + burstStrY + "]," +
               "[" + burstStrZ + "]," +
               String(voltageV, 2) + "," +
               String(rpm, 2) + "," +
               String(scaled, 2);

  // Print locally
  Serial.println(csv);

  // Send via ESP-NOW
  esp_err_t result = esp_now_send(receiverMAC, (uint8_t *)csv.c_str(), csv.length() + 1);
  if (result != ESP_OK) {
    Serial.printf("❌ Send error: %d\n", result);
  }

  delay(2000);
}

// =================== Helper Functions ===================
void calibrateOffsets() {
  Serial.println("Calibrating ADXL345 offsets... Keep motor stationary!");
  const int calibrationSamples = 50;
  float sumX = 0, sumY = 0, sumZ = 0;

  for (int i = 0; i < calibrationSamples; i++) {
    sensors_event_t event;
    accel.getEvent(&event);
    sumX += event.acceleration.x;
    sumY += event.acceleration.y;
    sumZ += event.acceleration.z;
    delay(50);
  }

  offsetX = sumX / calibrationSamples;
  offsetY = sumY / calibrationSamples;
  offsetZ = sumZ / calibrationSamples;
  Serial.println("✅ ADXL345 Calibration Done.");
}

float readAverage_mA(int samples) {
  double sum = 0;
  for (int i = 0; i < samples; i++) sum += ACS.mA_AC();
  return sum / samples;
}
