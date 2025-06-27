# 🚗 Arduino Toll Gate System

This project is a simple **Arduino-based automated toll gate** using an ultrasonic sensor to detect approaching vehicles and a servo motor to open/close the gate. The system displays messages on an I2C LCD and provides feedback using a buzzer.

https://github.com/user-attachments/assets/c46f6643-33b2-47a1-85f2-5e97164562c5

## 🔧 Components Used

- Arduino Uno
- Servo Motor
- Ultrasonic Sensor (HC-SR04)
- I2C LCD Display (16x2)
- Buzzer
- Breadboard
- Jumper Wires
- USB Cable for programming

## 📋 Features

- Detects vehicle using an ultrasonic sensor.
- Displays welcome message and toll fee on LCD.
- Automatically opens and closes the gate using a servo motor.
- Buzzer feedback when gate opens.
- Delay added for safe gate operation.

## 💻 Code Explanation

- **Ultrasonic Sensor**: Measures the distance to detect a vehicle.
- **LCD Display**: Shows "Welcome!" and "Toll Fee 150" messages.
- **Servo Motor**: Controls the gate arm to open/close.
- **Buzzer**: Alerts when gate is triggered to open.

## 🚀 Getting Started

1. Connect all components as per the circuit.
2. Upload the code to your Arduino Uno using Arduino IDE.
3. Power the system and observe the automated toll gate in action.

## 🔌 Circuit Connections

| Component       | Arduino Pin |
|----------------|-------------|
| Servo Motor    | D4          |
| Ultrasonic Trig| D5          |
| Ultrasonic Echo| D7          |
| Buzzer         | D8          |
| LCD (I2C)      | A4 (SDA), A5 (SCL) |

## 🧠 Code

#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// Pin assignments
const int trigPin = 5;      // Ultrasonic Trig
const int echoPin = 7;      // Ultrasonic Echo
const int buzzerPin = 8;    // Buzzer
const int servoPin = 4;     // Servo

long duration;
int distance;

Servo gateServo;
LiquidCrystal_I2C lcd(0x27, 16, 2); // Common I2C address is 0x27. Try 0x3F if 0x27 doesn't work.

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);
  gateServo.attach(servoPin);
  gateServo.write(0);    // Gate closed

  lcd.init();
  lcd.backlight();
  lcd.setCursor(0, 0);
  lcd.print("Welcome!");
  Serial.begin(9600);
}

void openGate() {
  // Buzz before opening
  tone(buzzerPin, 1000);
  delay(500);
  noTone(buzzerPin);

  // Slowly open gate
  for (int pos = 25; pos <= 105; pos += 1) {
    gateServo.write(pos);
    delay(10);
  }
  delay(2500); // Wait for vehicle to pass

  // Slowly close gate
  for (int pos = 105; pos >= 25; pos -= 1) {
    gateServo.write(pos);
    delay(10);
  }
  delay(2000); // Wait before next measurement
}

void loop() {
  // Ultrasonic sensor logic
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH, 30000);

  if (duration == 0) {
    delay(200);
    return;
  }

  distance = duration * 0.034 / 2;

  if (distance > 0 && distance < 12) {
    lcd.setCursor(0, 1);
    lcd.print("Toll Fee 150 "); // 16 chars max
    openGate();
    lcd.setCursor(0, 1);
    lcd.print("                "); // Clear second line
  } else {
    lcd.setCursor(0, 0);
    lcd.print("Welcome!        ");
    lcd.setCursor(0, 1);
    lcd.print("                ");
    delay(200);
  }
}
