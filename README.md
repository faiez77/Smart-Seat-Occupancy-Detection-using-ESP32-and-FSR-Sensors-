# 🪑 Smart Seat Vacancy Monitoring System (ESP32 + FSR Sensors + Wi-Fi + Bluetooth)

### 🔍 Overview
This project implements a **real-time seat occupancy monitoring system** using an **ESP32** and **Force Sensitive Resistors (FSRs)**.  
The ESP32 reads analog data from two FSR sensors through a **voltage divider circuit**, determines if each seat is occupied or vacant, and displays the results on a **Wi-Fi-based web dashboard**.  
Bluetooth mode is also available for offline monitoring.

---

### 🚀 Features
- Detects seat occupancy using two FSR sensors  
- Real-time display through Wi-Fi web server  
- Auto-refreshing webpage with color status indicators  
- Bluetooth serial output for offline use  
- Adjustable threshold (experimentally found as **2100**)  
- Works on 3.3 V logic, no external ADC required  

---

### 🧠 Components Used

| Component | Quantity | Description |
|------------|-----------|-------------|
| ESP32 Dev Board | 1 | Microcontroller with Wi-Fi + Bluetooth |
| Force Sensitive Resistor (FSR) | 2 | Pressure-dependent resistor sensor |
| 10 kΩ Resistor | 2 | Fixed resistor for voltage divider |
| Breadboard + Jumper Wires | — | For circuit connections |

---

### ⚙️ Circuit Diagram (Voltage Divider)

Each FSR forms a **voltage divider** with a 10 kΩ resistor.  
The junction voltage (**Vout**) is fed to an analog-only ESP32 pin.

| Sensor | Analog Pin | Notes |
|---------|-------------|-------|
| Seat 1 FSR | GPIO 34 | ADC1 channel |
| Seat 2 FSR | GPIO 35 | ADC1 channel |

📘 **Why GPIO 34 & 35?**  
They are *input-only analog pins* from **ADC1**, which stays stable even when Wi-Fi is running.

---

### ⚙️ ADC and Voltage Formula

The analog voltage at the junction is:

Vout = Vin*{R_{fixed}/{R_{fixed} + R_{FSR}}


where R_{fixed} = 10kΩ  , Vin=3.3v.

The ESP32’s **12-bit ADC** converts 0–3.3 V → 0–4095 counts:


ADC_value = Vout*3.3/4095


Example:  
If `ADC = 2100`  
Vout = 2100*3.3/4095 = 1.69 V


---

### ⚡ Baud Rate (115200)
The **baud rate** is the data speed between ESP32 and PC via USB.  
We use **115200 bps** because:
- It’s the default stable rate for the ESP32’s CP2102 USB-UART chip.  
- Allows fast and error-free serial communication.  
- Slower rates (like 9600 bps) cause delay in live data display.  

So we initialize serial as:
```cpp
Serial.begin(115200);


### Full Wi-Fi Code
#include <WiFi.h>

// ===== Wi-Fi Credentials =====
const char* ssid = "POCO F3 GT";
const char* password = "anurag94";

// ===== FSR Sensor Pins =====
const int fsrSeat1 = 34;
const int fsrSeat2 = 35;

// ===== Threshold for Occupancy =====
int threshold = 2100;  // Fixed experimentally

WiFiServer server(80);
int fsr1 = 0, fsr2 = 0;

void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Starting Seat Monitoring System...");

  // ---- Wi-Fi Connection ----
  WiFi.begin(ssid, password);
  Serial.print("Connecting to Wi-Fi");
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ Wi-Fi Connected!");
  Serial.print("🌐 IP Address: ");
  Serial.println(WiFi.localIP());

  server.begin();
}

void loop() {
  WiFiClient client = server.available();
  if (!client) return;

  Serial.println("New client connected");
  String request = client.readStringUntil('\r');
  client.flush();

  // ---- Read Sensors ----
  fsr1 = analogRead(fsrSeat1);
  fsr2 = analogRead(fsrSeat2);

  // ---- Determine Seat State ----
  String seat1Status = (fsr1 > threshold) ? "Occupied" : "Vacant";
  String seat2Status = (fsr2 > threshold) ? "Occupied" : "Vacant";

  // ---- Send HTML Page ----
  client.println("HTTP/1.1 200 OK");
  client.println("Content-type:text/html");
  client.println("Connection: close");
  client.println();

  client.println("<!DOCTYPE html><html>");
  client.println("<head><meta name='viewport' content='width=device-width, initial-scale=1'>");
  client.println("<title>Seat Vacancy Monitor</title>");
  client.println("<style>");
  client.println("body{font-family:Arial;text-align:center;background:#f5f5f5;}");
  client.println(".seat{display:inline-block;margin:20px;padding:30px;border-radius:15px;box-shadow:0 0 10px #ccc;width:200px;}");
  client.println(".occupied{background:#ff6b6b;color:white;}");
  client.println(".vacant{background:#4CAF50;color:white;}");
  client.println("h1{color:#333;}");
  client.println("</style></head>");
  client.println("<body>");
  client.println("<h1>Seat Vacancy Monitor</h1>");

  // ---- Seat 1 ----
  client.print("<div class='seat ");
  client.print((seat1Status == "Occupied") ? "occupied" : "vacant");
  client.print("'><h2>Seat 1</h2><p>");
  client.print(seat1Status);
  client.print("</p><p>FSR Reading: ");
  client.print(fsr1);
  client.println("</p></div>");

  // ---- Seat 2 ----
  client.print("<div class='seat ");
  client.print((seat2Status == "Occupied") ? "occupied" : "vacant");
  client.print("'><h2>Seat 2</h2><p>");
  client.print(seat2Status);
  client.print("</p><p>FSR Reading: ");
  client.print(fsr2);
  client.println("</p></div>");

  client.println("<p><small>Page refreshes every 2 seconds</small></p>");
  client.println("<meta http-equiv='refresh' content='2'>");
  client.println("</body></html>");
  client.println();

  client.stop();
  Serial.println("Client disconnected\n");
}










