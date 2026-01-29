A Water Level Detection System measures how full a water tank is (low, medium, high, overflow) instead of only “water present or not”.
You can build a simple 4-level indicator using:
Arduino
Water level sensor (analog type or multiple probe wires)
3 LEDs (Low, Mid, High) + 1 Buzzer (Overflow)
Optional web dashboard to show the level
Working idea
The sensor gives an analog value:
Small value → empty tank
Bigger value → more water touching the sensor → higher level
We compare the value with thresholds and decide the level.
GitHub project structure
Copy code

water-level-detection/
│
├── arduino/
│   └── water_level.ino
├── server/
│   ├── app.py
│   └── templates/
│       └── index.html
├── README.md
└── requirements.txt
1️⃣ Arduino code (arduino/water_level.ino)
Copy code
Cpp
int sensorPin = A0;

int ledLow  = 4;
int ledMid  = 5;
int ledHigh = 6;
int buzzer  = 7;

void setup() {
  pinMode(ledLow, OUTPUT);
  pinMode(ledMid, OUTPUT);
  pinMode(ledHigh, OUTPUT);
  pinMode(buzzer, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int value = analogRead(sensorPin);

  String level;

  if (value < 300) {
    level = "LOW";
  }
  else if (value < 500) {
    level = "MID";
  }
  else if (value < 700) {
    level = "HIGH";
  }
  else {
    level = "OVERFLOW";
  }

  // reset all
  digitalWrite(ledLow, LOW);
  digitalWrite(ledMid, LOW);
  digitalWrite(ledHigh, LOW);
  digitalWrite(buzzer, LOW);

  if (level == "LOW") {
    digitalWrite(ledLow, HIGH);
  }
  else if (level == "MID") {
    digitalWrite(ledMid, HIGH);
  }
  else if (level == "HIGH") {
    digitalWrite(ledHigh, HIGH);
  }
  else if (level == "OVERFLOW") {
    digitalWrite(ledHigh, HIGH);
    digitalWrite(buzzer, HIGH); // alarm
  }

  Serial.println(level); // send to PC
  delay(1000);
}
(You can change threshold values after testing your sensor.)
2️⃣ Python Flask server (server/app.py)
Copy code
Python
from flask import Flask, render_template
import serial

app = Flask(__name__)
ser = serial.Serial('COM3', 9600)  # change to your port

level = "LOW"

@app.route('/')
def index():
    global level
    if ser.in_waiting:
        level = ser.readline().decode().strip()
    return render_template("index.html", level=level)

app.run(debug=True)
3️⃣ Web page (server/templates/index.html)
Copy code
Html
<!DOCTYPE html>
<html>
<head>
    <title>Water Level Monitor</title>
    <meta http-equiv="refresh" content="2">
    <style>
        .box{
            width:250px;height:120px;
            font-size:30px;
            line-height:120px;
            color:white;
            text-align:center;
        }
        .LOW { background:red; }
        .MID { background:orange; }
        .HIGH { background:green; }
        .OVERFLOW { background:blue; }
    </style>
</head>
<body>
    <h1>Water Level Detection System</h1>
    <div class="box {{level}}">{{level}}</div>
</body>
</html>
4️⃣ requirements.txt
Copy code

flask
pyserial
Install:
Copy code
Bash
pip install -r requirements.txt
5️⃣ README.md (for GitHub)
Copy code
Markdown
# Water Level Detection System 🚰

Arduino based system to monitor tank water level (Low, Mid, High, Overflow) with web display and alarm.

## Features
- 4 level indication using analog water sensor
- LED indicators for each level
- Buzzer alert on overflow
- Live status on web page

## Hardware
- Arduino Uno
- Water level sensor
- 3 LEDs + resistors
- Buzzer
- Jumper wires

## How it works
1. Sensor reads analog water level.
2. Arduino compares with thresholds.
3. LEDs show current level.
4. Overflow triggers buzzer.
5. Level sent to Python Flask server.
6. Browser shows live level.

## Run
1. Upload Arduino code.
2. Update COM port in `app.py`.
3. Run:
   ```bash
   python app.py
Open http://127.0.0.1:5000
Copy code

## Future improvements
- Automatic motor ON/OFF for tank filling  
- SMS or mobile notification on overflow  
- Store data and show level graph  
- Use ESP8266/ESP32 for Wi-Fi (no PC needed)

This makes a good mini project showing sensors, Arduino control, alerts, and a simple IoT web dashboard.# Water-level-detection-system-
