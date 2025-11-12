# ⚙️ ESP32 — Architecture, Memory, and Key Technical Details

<img width="1690" height="685" alt="image" src="https://github.com/user-attachments/assets/923bf4a3-5199-4f19-911b-5546ce91bcc2" />



The **ESP32** is a powerful dual-core microcontroller designed by **Espressif Systems**, widely used in IoT, automation, and embedded applications.  
It supports **Wi-Fi**, **Bluetooth (Classic + BLE)**, and has advanced peripherals like ADC, DAC, PWM, and UART interfaces.

---

## 🧠 1. Architecture Overview

| Feature | Description |
|----------|-------------|
| **Processor** | Dual-core Tensilica Xtensa LX6 @ 240 MHz |
| **Wi-Fi** | IEEE 802.11 b/g/n (2.4 GHz) |
| **Bluetooth** | v4.2 BR/EDR + BLE |
| **Operating Voltage** | 3.3 V |
| **I/O Voltage Level** | 3.3 V logic (NOT 5 V tolerant) |
| **GPIO Pins** | Up to 30 GPIOs (depends on board) |
| **ADC Channels** | 18 channels (12-bit resolution) |
| **DAC Channels** | 2 (8-bit) |
| **PWM Channels** | 16 |
| **UART / SPI / I²C** | 3x UART, 4x SPI, 2x I²C |
| **Timers** | 4 hardware timers (64-bit) |

---

## 💾 2. ESP32 Memory Organization

| Memory Type | Size | Function |
|--------------|------|----------|
| **Flash Memory** | 4 MB (typical) | Stores program code (sketch) permanently. Uploaded via USB. |
| **SRAM (Data RAM)** | ~520 KB | Stores variables, stack, and temporary data during runtime. |
| **ROM** | 448 KB | Contains bootloader and core firmware. |
| **RTC Fast Memory** | 8 KB | Retains small data during deep sleep. |
| **RTC Slow Memory** | 8 KB | Used for ultra-low-power tasks while sleeping. |

### 📘 How Code Storage Works
When you upload code using Arduino IDE:
1. The compiled program is written into **Flash Memory**.
2. At startup, ESP32’s **ROM bootloader** loads that program into **SRAM** for execution.
3. Any global variables or arrays are allocated dynamically in SRAM.

---

## ⚡ 3. ADC (Analog to Digital Converter)

| Feature | Details |
|----------|----------|
| **Resolution** | 12-bit (0–4095) |
| **Input Voltage Range** | 0–3.3 V |
| **ADC Units** | ADC1 and ADC2 |
| **Channels** | ADC1 → 8 channels (GPIO32–39) <br> ADC2 → 10 channels (GPIO0,2,4,12–15,25–27) |

### Important:
- **ADC1 pins** (like 34, 35, 36, 39) are **input-only** and remain stable during Wi-Fi use.  
- **ADC2** pins often conflict with Wi-Fi hardware and are not recommended during wireless communication.

---

## 🔌 4. Important Pins and Their Purpose

| Pin | Function | Notes |
|------|-----------|-------|
| GPIO 34–39 | Analog Input (ADC1) | Input-only pins (ideal for sensors) |
| GPIO 0 | Boot Mode / ADC2 | Avoid unless needed |
| GPIO 2 | ADC2 / Output | Used during boot; careful using it |
| GPIO 15 | Boot configuration pin |
| GPIO 21, 22 | I²C SDA, SCL (default) |
| GPIO 1, 3 | UART0 TX/RX (used for USB serial) |
| GPIO 16, 17 | UART2 (for external Bluetooth/UART) |
| GPIO 25, 26, 27 | PWM / DAC / General purpose |
| GPIO 23, 19, 18, 5 | SPI default pins |

---

## 🔋 5. Power Management and Deep Sleep

- ESP32 supports **Deep Sleep Mode** where only RTC and low-power domains stay active.  
- It can retain data in **RTC memory** and wake up using:
  - GPIO interrupt
  - Timer
  - Touch sensor
- Deep Sleep current: ~10 µA  
- Normal active current: ~80–260 mA (depends on Wi-Fi usage)

---

## 📶 6. Communication Interfaces

| Protocol | Description |
|-----------|--------------|
| **Wi-Fi** | Full TCP/IP stack, can act as client or server (SoftAP mode supported) |
| **Bluetooth Classic** | Serial communication (SPP) — used in this project |
| **BLE (Bluetooth Low Energy)** | For low-power IoT |
| **UART** | Serial communication with PC / sensors |
| **I²C** | Used for sensors, OLED, RTC |
| **SPI** | High-speed data transfer with modules like SD card or display |

---

## 🔠 7. Baud Rate Explanation

**Baud rate** = number of bits transmitted per second.

Common rates: 9600, 57600, 115200 bps  
ESP32 typically uses **115200 bps** because:
- The USB-to-UART chip (CP2102/CH340) is optimized for it.  
- It ensures smooth serial communication without data loss.  
- Higher rates (230400+) can cause instability in some systems.
00);


## 🔹 8. FSR Characteristics
| Condition | Typical Resistance | Voltage Output | ADC Reading |
|------------|-------------------|----------------|-------------|
| No pressure | >1 MΩ | ~0.0 V | ~0–200 |
| Light touch | 50–100 kΩ | ~0.3–0.8 V | ~300–1000 |
| Firm press | 1–5 kΩ | ~2–3 V | ~2500–3800 |

*FSRs are not perfectly linear; calibration is done experimentally.*

---

## 🔹 9. Threshold Setting
Threshold = midpoint between “vacant” and “occupied” readings.  
If measured:
- Vacant seat → ~1500  
- Occupied seat → ~2700  

Then threshold ≈  
\[
\frac{1500 + 2700}{2} = 2100
\]

Code logic:
```cpp
if (fsrReading > 2100)
   Seat = "Occupied";
else
   Seat = "Vacant";
