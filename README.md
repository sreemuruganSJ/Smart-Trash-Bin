Code Implementation :
#include <ESP32Servo.h>
#include <Wire.h>
#include "HX711.h"
#include <WiFi.h>
#include <ThingSpeak.h>
Servo inductiveServo;
Servo capacitiveServo;
HX711 scale;
const int inductiveSensorPin = 34;
const int capacitiveSensorPin = 35;
const int inductiveThreshold = 500;
const int capacitiveThreshold = 500;
const int LOADCELL_DOUT_PIN = 26;
const int LOADCELL_SCK_PIN = 27;
const char *ssid = "Vivo v21e";
const char *password = "blackbeast";
const char *thingSpeakApiKey = "6YMLB2YG4X50ZUNY";
const int thingSpeakChannelID =2497312;
WiFiClient client;
void setup() {
Serial.begin(9600);
inductiveServo.attach(32); // Attach servo to GPIO pin 32
capacitiveServo.attach(33); // Attach servo to GPIO pin 33
pinMode(inductiveSensorPin, INPUT);
pinMode(capacitiveSensorPin, INPUT);
scale.begin(LOADCELL_DOUT_PIN, LOADCELL_SCK_PIN);
scale.set_scale();
scale.tare();
WiFi.begin(ssid, password);
Serial.print("Connecting to ");
Serial.println(ssid);
while (WiFi.status() != WL_CONNECTED) {
delay(500);
Serial.print(".");
}
Serial.println("WiFi connected");
ThingSpeak.begin(client); // Initialize ThingSpeak
}
void loop() {
int inductiveSensorValue = analogRead(inductiveSensorPin);
int capacitiveSensorValue = analogRead(capacitiveSensorPin);
float weight = scale.get_units(10);
if (inductiveSensorValue > inductiveThreshold) {
inductiveServo.write(90); // Rotate servo 90 degrees clockwise
delay(1000); // Wait for 1 second
inductiveServo.write(0);
delay(1000); // Wait for 1 second
}
// Check if capacitive sensor detects plastic or other materials
if (capacitiveSensorValue > capacitiveThreshold) {
capacitiveServo.write(90);
delay(1000);
capacitiveServo.write(0);
delay(1000);
}
ThingSpeak.writeField(thingSpeakChannelID, 1, inductiveSensorValue, thingSpeakApiKey);
ThingSpeak.writeField(thingSpeakChannelID, 2, capacitiveSensorValue, thingSpeakApiKey);
ThingSpeak.writeField(thingSpeakChannelID, 3, weight, thingSpeakApiKey);
ThingSpeak.writeField(thingSpeakChannelID, 4, weight, thingSpeakApiKey); // Sending
weight to two fields for redundancy
delay(20000); // Send data to ThingSpeak every 20 seconds
}
