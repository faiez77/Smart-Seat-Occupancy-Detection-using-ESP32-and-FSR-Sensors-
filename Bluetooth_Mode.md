#include "BluetoothSerial.h"
BluetoothSerial SerialBT;

const int fsrSeat1 = 34;
const int fsrSeat2 = 35;
int threshold = 2100;

void setup() {
  Serial.begin(115200);
  SerialBT.begin("SeatMonitor_BT");
  Serial.println("Bluetooth mode started...");
}

void loop() {
  int fsr1 = analogRead(fsrSeat1);
  int fsr2 = analogRead(fsrSeat2);

  String seat1Status = (fsr1 > threshold) ? "Occupied" : "Vacant";
  String seat2Status = (fsr2 > threshold) ? "Occupied" : "Vacant";

  SerialBT.print("Seat 1: "); SerialBT.print(seat1Status);
  SerialBT.print(" | FSR1: "); SerialBT.println(fsr1);
  SerialBT.print("Seat 2: "); SerialBT.print(seat2Status);
  SerialBT.print(" | FSR2: "); SerialBT.println(fsr2);
  SerialBT.println("---------------------");

  delay(1000);
}
