# Workshop: การพัฒนาระบบ IoT ด้วย ESP8266 และ Web Dashboard

## ภาพรวมกิจกรรม

กิจกรรมนี้มุ่งให้ผู้เรียนพัฒนาระบบ IoT ขนาดเล็กที่สามารถอ่านข้อมูลจากเซนเซอร์ ส่งข้อมูลผ่านเครือข่าย Wi-Fi บันทึกข้อมูลลงไฟล์ CSV และนำข้อมูลมาแสดงบน Web Dashboard

โครงสร้างระบบที่ใช้ในกิจกรรมมีดังนี้

```text
DHT11 Sensor
     │
     ▼
NodeMCU ESP8266
     │
     │ HTTP POST + JSON
     ▼
Flask Web API บน Ubuntu
     │
     ▼
sensor_data.csv
     │
     ▼
Web Dashboard
 ├── ข้อมูลล่าสุด
 ├── ตารางข้อมูลย้อนหลัง
 ├── กราฟเส้น
 └── กราฟแท่ง
```

---

# หัวข้อที่ 1 การอ่านข้อมูลจากเซนเซอร์และแสดงผลผ่าน Serial Monitor

## 1.1 วัตถุประสงค์

เมื่อจบกิจกรรมนี้ ผู้เรียนสามารถ

1. เชื่อมต่อเซนเซอร์ DHT11 เข้ากับ ESP8266 ได้
2. เขียนโปรแกรมอ่านค่าอุณหภูมิและความชื้นได้
3. แสดงข้อมูลผ่าน Serial Monitor ของ Arduino IDE ได้
4. ตรวจสอบและแก้ไขปัญหาการอ่านค่าเซนเซอร์เบื้องต้นได้

---

## 1.2 อุปกรณ์

| อุปกรณ์                                  |     จำนวน |
| ---------------------------------------- | --------: |
| NodeMCU ESP8266                          |   1 บอร์ด |
| DHT11 Sensor Module                      |     1 ตัว |
| Breadboard                               |    1 แผ่น |
| Jumper Wire                              |    3 เส้น |
| สาย Micro USB                            |    1 เส้น |
| เครื่องคอมพิวเตอร์ที่ติดตั้ง Arduino IDE | 1 เครื่อง |

---

## 1.3 การเชื่อมต่อวงจร

เชื่อมต่อ DHT11 กับ NodeMCU ESP8266 ดังนี้

| ขา DHT11        | ขา ESP8266 |
| --------------- | ---------- |
| VCC หรือ `+`    | `3V3`      |
| DATA หรือ `OUT` | `D4`       |
| GND หรือ `-`    | `GND`      |

> ในโปรแกรม Arduino ขา `D4` ของ NodeMCU ตรงกับ GPIO2

---

## 1.4 การติดตั้งไลบรารี DHT11

เปิด Arduino IDE แล้วดำเนินการดังนี้

1. เลือกเมนู **Tools → Manage Libraries**
2. ค้นหา `DHT sensor library`
3. ติดตั้ง **DHT sensor library by Adafruit**
4. หากระบบแจ้งเตือน ให้ติดตั้ง **Adafruit Unified Sensor** เพิ่มเติม

---

## 1.5 โปรแกรมอ่านข้อมูลจาก DHT11

สร้าง Sketch ใหม่ใน Arduino IDE แล้วคัดลอกโปรแกรมต่อไปนี้

```cpp
#include <DHT.h>

// กำหนดขาที่เชื่อมต่อกับ DHT11
#define DHT_PIN D4

// กำหนดชนิดของเซนเซอร์
#define DHT_TYPE DHT11

DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
  // เริ่มต้น Serial Communication
  Serial.begin(115200);

  // เริ่มต้นเซนเซอร์ DHT11
  dht.begin();

  Serial.println();
  Serial.println("DHT11 Sensor Test");
  Serial.println("-----------------");
}

void loop() {
  // DHT11 ไม่ควรถูกอ่านถี่เกินไป
  delay(2000);

  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  // ตรวจสอบว่าการอ่านข้อมูลสำเร็จหรือไม่
  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Error: Cannot read data from DHT11");
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print(" °C");

  Serial.print(" | Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");
}
```

---

## 1.6 การอัปโหลดโปรแกรม

1. เชื่อมต่อ NodeMCU ESP8266 เข้ากับคอมพิวเตอร์
2. เลือก **Tools → Board → ESP8266 Boards → NodeMCU 1.0 (ESP-12E Module)**
3. เลือก Port ที่เชื่อมต่อกับบอร์ด
4. กดปุ่ม **Verify**
5. กดปุ่ม **Upload**
6. เปิด **Tools → Serial Monitor**
7. กำหนด Baud Rate เป็น `115200`

ตัวอย่างผลลัพธ์

```text
DHT11 Sensor Test
-----------------
Temperature: 29.20 °C | Humidity: 68.00 %
Temperature: 29.30 °C | Humidity: 67.00 %
Temperature: 29.40 °C | Humidity: 67.00 %
```

---

## 1.7 แบบฝึกปฏิบัติ

ให้ผู้เรียนปรับปรุงโปรแกรมดังนี้

1. แสดงข้อความ `HOT` เมื่ออุณหภูมิสูงกว่า 30 องศาเซลเซียส
2. แสดงข้อความ `NORMAL` เมื่ออุณหภูมิไม่เกิน 30 องศาเซลเซียส
3. ทดลองเป่าลมที่เซนเซอร์และสังเกตค่าความชื้น

ตัวอย่างเงื่อนไข

```cpp
if (temperature > 30.0) {
  Serial.println("Status: HOT");
} else {
  Serial.println("Status: NORMAL");
}
```

---

# หัวข้อที่ 2 การส่งข้อมูลเซนเซอร์ด้วย JSON และ REST API

## 2.1 วัตถุประสงค์

เมื่อจบกิจกรรมนี้ ผู้เรียนสามารถ

1. เชื่อมต่อ ESP8266 เข้ากับเครือข่าย Wi-Fi ได้
2. อธิบายโครงสร้างข้อมูล JSON ได้
3. สร้างข้อมูล JSON จากค่าที่อ่านได้จากเซนเซอร์
4. ส่งข้อมูลไปยัง Web API ด้วย HTTP POST ได้

---

## 2.2 ความหมายของ JSON

JSON ย่อมาจาก **JavaScript Object Notation** เป็นรูปแบบข้อความที่นิยมใช้แลกเปลี่ยนข้อมูลระหว่างอุปกรณ์และ Web Server

ตัวอย่างข้อมูล JSON

```json
{
  "device_id": "esp8266-01",
  "temperature": 29.5,
  "humidity": 68.0
}
```

ข้อมูลแต่ละรายการประกอบด้วย

```text
"ชื่อข้อมูล": ค่า
```

ตัวอย่าง

```text
"temperature": 29.5
```

หมายถึงข้อมูลชื่อ `temperature` มีค่าเท่ากับ `29.5`

---

## 2.3 การเตรียมข้อมูลเครือข่าย

ก่อนเขียนโปรแกรม ผู้เรียนต้องทราบข้อมูลต่อไปนี้

* ชื่อ Wi-Fi หรือ SSID
* รหัสผ่าน Wi-Fi
* IP Address ของเครื่อง Ubuntu

บนเครื่อง Ubuntu ให้เปิด Terminal แล้วใช้คำสั่ง

```bash
hostname -I
```

ตัวอย่างผลลัพธ์

```text
192.168.1.100
```

ดังนั้น URL ของ Web API จะเป็น

```text
http://192.168.1.100:5000/api/sensor
```

> ESP8266 และ Ubuntu ต้องเชื่อมต่ออยู่ในเครือข่ายเดียวกัน

---

## 2.4 โปรแกรม ESP8266 ส่งข้อมูล JSON

แก้ไขค่า `WIFI_SSID`, `WIFI_PASSWORD` และ `SERVER_URL` ให้ตรงกับระบบที่ใช้งาน

```cpp
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClient.h>
#include <DHT.h>

#define DHT_PIN D4
#define DHT_TYPE DHT11

const char* WIFI_SSID = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

// เปลี่ยน IP Address ให้เป็น IP ของเครื่อง Ubuntu
const char* SERVER_URL =
  "http://192.168.1.100:5000/api/sensor";

const char* DEVICE_ID = "esp8266-01";

DHT dht(DHT_PIN, DHT_TYPE);

void connectToWiFi() {
  Serial.print("Connecting to Wi-Fi");

  WiFi.mode(WIFI_STA);
  WiFi.begin(WIFI_SSID, WIFI_PASSWORD);

  unsigned long startTime = millis();

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");

    // ยกเลิกเมื่อเชื่อมต่อไม่สำเร็จภายใน 20 วินาที
    if (millis() - startTime > 20000) {
      Serial.println();
      Serial.println("Wi-Fi connection timeout");
      return;
    }
  }

  Serial.println();
  Serial.println("Wi-Fi connected");

  Serial.print("ESP8266 IP address: ");
  Serial.println(WiFi.localIP());
}

void sendSensorData(
  float temperature,
  float humidity
) {
  if (WiFi.status() != WL_CONNECTED) {
    Serial.println("Wi-Fi disconnected");
    connectToWiFi();
    return;
  }

  WiFiClient client;
  HTTPClient http;

  // เริ่มต้นการเชื่อมต่อกับ Web API
  if (!http.begin(client, SERVER_URL)) {
    Serial.println("Cannot connect to server");
    return;
  }

  // กำหนดชนิดข้อมูลที่ส่งเป็น JSON
  http.addHeader("Content-Type", "application/json");

  // สร้างข้อความ JSON
  String jsonData = "{";
  jsonData += "\"device_id\":\"";
  jsonData += DEVICE_ID;
  jsonData += "\",";
  jsonData += "\"temperature\":";
  jsonData += String(temperature, 2);
  jsonData += ",";
  jsonData += "\"humidity\":";
  jsonData += String(humidity, 2);
  jsonData += "}";

  Serial.println("Sending JSON:");
  Serial.println(jsonData);

  // ส่งข้อมูลด้วย HTTP POST
  int httpCode = http.POST(jsonData);

  if (httpCode > 0) {
    Serial.print("HTTP status code: ");
    Serial.println(httpCode);

    String response = http.getString();

    Serial.print("Server response: ");
    Serial.println(response);
  } else {
    Serial.print("HTTP request failed: ");
    Serial.println(http.errorToString(httpCode));
  }

  // ปิดการเชื่อมต่อ
  http.end();
}

void setup() {
  Serial.begin(115200);
  dht.begin();

  Serial.println();
  Serial.println("ESP8266 Sensor Node");

  connectToWiFi();
}

void loop() {
  float humidity = dht.readHumidity();
  float temperature = dht.readTemperature();

  if (isnan(humidity) || isnan(temperature)) {
    Serial.println("Cannot read data from DHT11");
    delay(5000);
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.print(" °C | Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  sendSensorData(temperature, humidity);

  // ส่งข้อมูลทุก 5 วินาที
  delay(5000);
}
```

ไลบรารีของ ESP8266 Arduino Core รองรับการเชื่อมต่อ Wi-Fi และการสื่อสารผ่าน HTTP โดยตรง ทำให้ ESP8266 สามารถทำหน้าที่เป็น IoT Sensor Node ได้

---

# หัวข้อที่ 3 การสร้าง Flask Web API และบันทึกข้อมูลลงไฟล์ CSV

## 3.1 วัตถุประสงค์

เมื่อจบกิจกรรมนี้ ผู้เรียนสามารถ

1. สร้าง Flask Web Application บน Ubuntu ได้
2. สร้าง REST API สำหรับรับ HTTP POST ได้
3. รับและตรวจสอบข้อมูล JSON ได้
4. บันทึกข้อมูลลงไฟล์ CSV พร้อมวันและเวลาได้
5. ทดสอบ Web API ก่อนเชื่อมต่อกับ ESP8266 ได้

Flask เป็น Web Application Framework สำหรับภาษา Python ที่เหมาะกับการสร้าง Web API และโครงงานต้นแบบ เนื่องจากมีโครงสร้างไม่ซับซ้อนและเริ่มต้นใช้งานได้รวดเร็ว

---

## 3.2 การสร้างโฟลเดอร์โครงการ

เปิด Terminal บน Ubuntu แล้วใช้คำสั่ง

```bash
mkdir -p ~/aiot-workshop/templates
cd ~/aiot-workshop
```

โครงสร้างโครงการที่ต้องการคือ

```text
aiot-workshop/
├── app.py
├── sensor_data.csv
└── templates/
    └── dashboard.html
```

ไฟล์ `sensor_data.csv` จะถูกสร้างโดยโปรแกรมโดยอัตโนมัติ

---

## 3.3 การสร้าง Python Virtual Environment

ติดตั้งเครื่องมือที่จำเป็น

```bash
sudo apt update
sudo apt install python3 python3-venv python3-pip -y
```

สร้าง Virtual Environment

```bash
cd ~/aiot-workshop
python3 -m venv venv
```

เปิดใช้งาน Virtual Environment

```bash
source venv/bin/activate
```

ติดตั้ง Flask

```bash
pip install Flask
```

---

## 3.4 โปรแกรม Flask Web API

สร้างไฟล์ `app.py`

```bash
nano app.py
```

คัดลอกโปรแกรมต่อไปนี้

```python
from __future__ import annotations

import csv
import os
import threading
from datetime import datetime
from typing import Any

from flask import Flask, jsonify, render_template, request

app = Flask(__name__)

CSV_FILE = "sensor_data.csv"
CSV_HEADER = [
    "timestamp",
    "device_id",
    "temperature",
    "humidity",
]

# ป้องกันการเขียนและอ่านไฟล์พร้อมกัน
csv_lock = threading.Lock()


def create_csv_file() -> None:
    """สร้างไฟล์ CSV และหัวตาราง หากไฟล์ยังไม่มี"""
    if os.path.exists(CSV_FILE):
        return

    with csv_lock:
        with open(
            CSV_FILE,
            mode="w",
            newline="",
            encoding="utf-8",
        ) as csv_file:
            writer = csv.DictWriter(
                csv_file,
                fieldnames=CSV_HEADER,
            )
            writer.writeheader()


def convert_to_float(
    value: Any,
    field_name: str,
) -> float:
    """แปลงข้อมูลเป็น float และแจ้งข้อผิดพลาดเมื่อข้อมูลไม่ถูกต้อง"""
    try:
        return float(value)
    except (TypeError, ValueError) as error:
        raise ValueError(
            f"{field_name} must be a number"
        ) from error


def save_sensor_data(
    device_id: str,
    temperature: float,
    humidity: float,
) -> dict[str, Any]:
    """บันทึกข้อมูลเซนเซอร์ลงไฟล์ CSV"""
    record = {
        "timestamp": datetime.now().strftime(
            "%Y-%m-%d %H:%M:%S"
        ),
        "device_id": device_id,
        "temperature": round(temperature, 2),
        "humidity": round(humidity, 2),
    }

    with csv_lock:
        with open(
            CSV_FILE,
            mode="a",
            newline="",
            encoding="utf-8",
        ) as csv_file:
            writer = csv.DictWriter(
                csv_file,
                fieldnames=CSV_HEADER,
            )
            writer.writerow(record)

    return record


def read_sensor_data(
    limit: int | None = None,
) -> list[dict[str, Any]]:
    """อ่านข้อมูลจากไฟล์ CSV"""
    create_csv_file()

    with csv_lock:
        with open(
            CSV_FILE,
            mode="r",
            newline="",
            encoding="utf-8",
        ) as csv_file:
            reader = csv.DictReader(csv_file)
            records = list(reader)

    # แปลงข้อมูลตัวเลขจากข้อความเป็น float
    formatted_records = []

    for record in records:
        try:
            formatted_records.append(
                {
                    "timestamp": record["timestamp"],
                    "device_id": record["device_id"],
                    "temperature": float(
                        record["temperature"]
                    ),
                    "humidity": float(
                        record["humidity"]
                    ),
                }
            )
        except (
            KeyError,
            TypeError,
            ValueError,
        ):
            # ข้ามแถวที่ข้อมูลไม่สมบูรณ์
            continue

    if limit is not None:
        return formatted_records[-limit:]

    return formatted_records


@app.route("/")
def dashboard():
    """แสดงหน้า Web Dashboard"""
    return render_template("dashboard.html")


@app.route(
    "/api/sensor",
    methods=["POST"],
)
def receive_sensor_data():
    """รับข้อมูล JSON จาก ESP8266"""
    if not request.is_json:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": (
                        "Content-Type must be "
                        "application/json"
                    ),
                }
            ),
            415,
        )

    data = request.get_json(silent=True)

    if not isinstance(data, dict):
        return (
            jsonify(
                {
                    "status": "error",
                    "message": "Invalid JSON data",
                }
            ),
            400,
        )

    required_fields = [
        "device_id",
        "temperature",
        "humidity",
    ]

    missing_fields = [
        field
        for field in required_fields
        if field not in data
    ]

    if missing_fields:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": "Missing required fields",
                    "missing_fields": missing_fields,
                }
            ),
            400,
        )

    device_id = str(data["device_id"]).strip()

    if not device_id:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": "device_id cannot be empty",
                }
            ),
            400,
        )

    try:
        temperature = convert_to_float(
            data["temperature"],
            "temperature",
        )
        humidity = convert_to_float(
            data["humidity"],
            "humidity",
        )
    except ValueError as error:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": str(error),
                }
            ),
            400,
        )

    # ตรวจสอบช่วงข้อมูลเบื้องต้น
    if temperature < -50 or temperature > 100:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": (
                        "Temperature is outside "
                        "the accepted range"
                    ),
                }
            ),
            400,
        )

    if humidity < 0 or humidity > 100:
        return (
            jsonify(
                {
                    "status": "error",
                    "message": (
                        "Humidity must be between "
                        "0 and 100"
                    ),
                }
            ),
            400,
        )

    record = save_sensor_data(
        device_id=device_id,
        temperature=temperature,
        humidity=humidity,
    )

    print("New sensor data:", record)

    return (
        jsonify(
            {
                "status": "success",
                "message": "Sensor data saved",
                "data": record,
            }
        ),
        201,
    )


@app.route(
    "/api/data",
    methods=["GET"],
)
def get_sensor_data():
    """ส่งข้อมูลย้อนหลังให้ Web Dashboard"""
    limit_text = request.args.get(
        "limit",
        default="20",
    )

    try:
        limit = int(limit_text)
    except ValueError:
        limit = 20

    # ป้องกันการร้องขอข้อมูลมากเกินไป
    limit = max(1, min(limit, 200))

    records = read_sensor_data(limit=limit)

    return jsonify(
        {
            "status": "success",
            "count": len(records),
            "data": records,
        }
    )


@app.route(
    "/api/latest",
    methods=["GET"],
)
def get_latest_sensor_data():
    """ส่งข้อมูลล่าสุดหนึ่งรายการ"""
    records = read_sensor_data(limit=1)

    if not records:
        return jsonify(
            {
                "status": "success",
                "data": None,
            }
        )

    return jsonify(
        {
            "status": "success",
            "data": records[0],
        }
    )


@app.errorhandler(404)
def page_not_found(error):
    return (
        jsonify(
            {
                "status": "error",
                "message": "Resource not found",
            }
        ),
        404,
    )


if __name__ == "__main__":
    create_csv_file()

    # host=0.0.0.0 ทำให้อุปกรณ์อื่นในเครือข่ายเข้าถึงได้
    app.run(
        host="0.0.0.0",
        port=5000,
        debug=True,
    )
```

Flask สามารถอ่านข้อมูล JSON ที่ส่งมากับ HTTP Request ผ่านระบบจัดการ JSON ของ Request Object ได้ โดยผู้ส่งควรกำหนด `Content-Type` เป็น `application/json`

---

## 3.5 การรัน Flask Web Server

ใน Terminal ใช้คำสั่ง

```bash
cd ~/aiot-workshop
source venv/bin/activate
python app.py
```

หากโปรแกรมทำงานสำเร็จ จะแสดงข้อความลักษณะดังนี้

```text
Running on http://127.0.0.1:5000
Running on http://192.168.1.100:5000
```

อย่าปิด Terminal ระหว่างที่กำลังทดสอบระบบ

---

## 3.6 การเปิด Firewall

หาก Ubuntu เปิดใช้งาน Firewall ให้ใช้คำสั่ง

```bash
sudo ufw allow 5000/tcp
```

ตรวจสอบสถานะ

```bash
sudo ufw status
```

---

## 3.7 การทดสอบ Web API ด้วย curl

ก่อนเชื่อมต่อ ESP8266 ควรทดสอบ Web API จาก Ubuntu ก่อน

เปิด Terminal อีกหน้าต่างหนึ่งแล้วใช้คำสั่ง

```bash
curl -X POST \
  http://127.0.0.1:5000/api/sensor \
  -H "Content-Type: application/json" \
  -d '{
    "device_id": "test-device",
    "temperature": 29.5,
    "humidity": 68.0
  }'
```

ตัวอย่างผลลัพธ์

```json
{
  "data": {
    "device_id": "test-device",
    "humidity": 68.0,
    "temperature": 29.5,
    "timestamp": "2026-07-20 10:30:00"
  },
  "message": "Sensor data saved",
  "status": "success"
}
```

ตรวจสอบไฟล์ CSV

```bash
cat sensor_data.csv
```

ตัวอย่างข้อมูล

```csv
timestamp,device_id,temperature,humidity
2026-07-20 10:30:00,test-device,29.5,68.0
```

---

## 3.8 การทดสอบ API สำหรับอ่านข้อมูล

อ่านข้อมูลย้อนหลัง 20 รายการ

```bash
curl http://127.0.0.1:5000/api/data
```

กำหนดจำนวนข้อมูลที่ต้องการ

```bash
curl "http://127.0.0.1:5000/api/data?limit=10"
```

อ่านข้อมูลล่าสุด

```bash
curl http://127.0.0.1:5000/api/latest
```

---

# หัวข้อที่ 4 การสร้าง Web Dashboard จากข้อมูลในไฟล์ CSV

## 4.1 วัตถุประสงค์

เมื่อจบกิจกรรมนี้ ผู้เรียนสามารถ

1. อ่านข้อมูลจากไฟล์ CSV ผ่าน Web API ได้
2. แสดงข้อมูลล่าสุดบน Dashboard ได้
3. แสดงตารางข้อมูลย้อนหลังได้
4. สร้างกราฟเส้นสำหรับแสดงแนวโน้มตามเวลาได้
5. สร้างกราฟแท่งสำหรับเปรียบเทียบข้อมูลได้
6. ทำให้ Dashboard อัปเดตข้อมูลอัตโนมัติได้

กราฟเส้นเหมาะสำหรับแสดงแนวโน้มของข้อมูลตามเวลา ส่วนกราฟแท่งเหมาะสำหรับเปรียบเทียบค่าระหว่างรายการหรือช่วงข้อมูล

---

## 4.2 การสร้างไฟล์ Dashboard

สร้างไฟล์

```bash
nano ~/aiot-workshop/templates/dashboard.html
```

คัดลอกโค้ดต่อไปนี้

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">

  <meta
    name="viewport"
    content="width=device-width, initial-scale=1.0"
  >

  <title>ESP8266 Sensor Dashboard</title>

  <link
    href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css"
    rel="stylesheet"
  >

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    body {
      background-color: #f4f6f9;
    }

    .dashboard-title {
      font-weight: 700;
    }

    .value-card {
      min-height: 150px;
    }

    .sensor-value {
      font-size: 2.4rem;
      font-weight: 700;
    }

    .chart-container {
      position: relative;
      height: 330px;
    }

    .status-online {
      font-weight: 600;
    }

    .table-container {
      max-height: 420px;
      overflow-y: auto;
    }
  </style>
</head>

<body>
  <nav class="navbar navbar-dark bg-dark">
    <div class="container">
      <span class="navbar-brand">
        AIoT Sensor Dashboard
      </span>

      <span
        id="connectionStatus"
        class="badge text-bg-secondary"
      >
        Waiting for data
      </span>
    </div>
  </nav>

  <main class="container py-4">
    <div class="mb-4">
      <h1 class="dashboard-title">
        ESP8266 Environment Monitoring
      </h1>

      <p class="text-secondary mb-0">
        แสดงข้อมูลอุณหภูมิและความชื้นจาก ESP8266
      </p>
    </div>

    <div class="row g-4 mb-4">
      <div class="col-md-4">
        <div class="card shadow-sm value-card">
          <div class="card-body">
            <h5 class="card-title">
              อุณหภูมิล่าสุด
            </h5>

            <div class="sensor-value">
              <span id="latestTemperature">--</span>
              <span class="fs-4">°C</span>
            </div>

            <p class="text-secondary mb-0">
              Temperature
            </p>
          </div>
        </div>
      </div>

      <div class="col-md-4">
        <div class="card shadow-sm value-card">
          <div class="card-body">
            <h5 class="card-title">
              ความชื้นล่าสุด
            </h5>

            <div class="sensor-value">
              <span id="latestHumidity">--</span>
              <span class="fs-4">%</span>
            </div>

            <p class="text-secondary mb-0">
              Relative Humidity
            </p>
          </div>
        </div>
      </div>

      <div class="col-md-4">
        <div class="card shadow-sm value-card">
          <div class="card-body">
            <h5 class="card-title">
              อุปกรณ์และเวลา
            </h5>

            <p class="mb-1">
              Device:
              <strong id="latestDevice">--</strong>
            </p>

            <p class="mb-0">
              Updated:
              <strong id="latestTimestamp">--</strong>
            </p>
          </div>
        </div>
      </div>
    </div>

    <div
      id="errorMessage"
      class="alert alert-danger d-none"
      role="alert"
    ></div>

    <div class="row g-4 mb-4">
      <div class="col-lg-8">
        <div class="card shadow-sm">
          <div class="card-body">
            <h5 class="card-title">
              กราฟเส้นแสดงข้อมูลย้อนหลัง
            </h5>

            <div class="chart-container">
              <canvas id="lineChart"></canvas>
            </div>
          </div>
        </div>
      </div>

      <div class="col-lg-4">
        <div class="card shadow-sm">
          <div class="card-body">
            <h5 class="card-title">
              กราฟแท่งข้อมูลล่าสุด
            </h5>

            <div class="chart-container">
              <canvas id="barChart"></canvas>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div class="card shadow-sm">
      <div class="card-body">
        <div
          class="d-flex justify-content-between align-items-center mb-3"
        >
          <h5 class="card-title mb-0">
            ข้อมูลย้อนหลัง
          </h5>

          <button
            class="btn btn-primary btn-sm"
            onclick="loadDashboardData()"
          >
            Refresh
          </button>
        </div>

        <div class="table-responsive table-container">
          <table class="table table-striped table-hover">
            <thead class="table-dark sticky-top">
              <tr>
                <th>ลำดับ</th>
                <th>วันและเวลา</th>
                <th>Device</th>
                <th>อุณหภูมิ</th>
                <th>ความชื้น</th>
              </tr>
            </thead>

            <tbody id="historyTableBody">
              <tr>
                <td
                  colspan="5"
                  class="text-center text-secondary"
                >
                  ยังไม่มีข้อมูล
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
  </main>

  <script>
    let lineChart = null;
    let barChart = null;

    function showError(message) {
      const errorBox =
        document.getElementById("errorMessage");

      errorBox.textContent = message;
      errorBox.classList.remove("d-none");
    }

    function hideError() {
      const errorBox =
        document.getElementById("errorMessage");

      errorBox.classList.add("d-none");
    }

    function updateConnectionStatus(hasData) {
      const status =
        document.getElementById("connectionStatus");

      if (hasData) {
        status.textContent = "Data received";
        status.className =
          "badge text-bg-success status-online";
      } else {
        status.textContent = "No data";
        status.className =
          "badge text-bg-secondary";
      }
    }

    function updateLatestData(latestRecord) {
      if (!latestRecord) {
        document.getElementById(
          "latestTemperature"
        ).textContent = "--";

        document.getElementById(
          "latestHumidity"
        ).textContent = "--";

        document.getElementById(
          "latestDevice"
        ).textContent = "--";

        document.getElementById(
          "latestTimestamp"
        ).textContent = "--";

        updateConnectionStatus(false);
        return;
      }

      document.getElementById(
        "latestTemperature"
      ).textContent =
        latestRecord.temperature.toFixed(1);

      document.getElementById(
        "latestHumidity"
      ).textContent =
        latestRecord.humidity.toFixed(1);

      document.getElementById(
        "latestDevice"
      ).textContent =
        latestRecord.device_id;

      document.getElementById(
        "latestTimestamp"
      ).textContent =
        latestRecord.timestamp;

      updateConnectionStatus(true);
    }

    function updateHistoryTable(records) {
      const tableBody =
        document.getElementById(
          "historyTableBody"
        );

      tableBody.innerHTML = "";

      if (records.length === 0) {
        tableBody.innerHTML = `
          <tr>
            <td
              colspan="5"
              class="text-center text-secondary"
            >
              ยังไม่มีข้อมูล
            </td>
          </tr>
        `;

        return;
      }

      const newestFirst = [...records].reverse();

      newestFirst.forEach((record, index) => {
        const row = document.createElement("tr");

        row.innerHTML = `
          <td>${index + 1}</td>
          <td>${record.timestamp}</td>
          <td>${record.device_id}</td>
          <td>${record.temperature.toFixed(1)} °C</td>
          <td>${record.humidity.toFixed(1)} %</td>
        `;

        tableBody.appendChild(row);
      });
    }

    function updateLineChart(records) {
      const labels = records.map(
        record => record.timestamp
      );

      const temperatures = records.map(
        record => record.temperature
      );

      const humidities = records.map(
        record => record.humidity
      );

      if (lineChart !== null) {
        lineChart.data.labels = labels;
        lineChart.data.datasets[0].data =
          temperatures;
        lineChart.data.datasets[1].data =
          humidities;
        lineChart.update();
        return;
      }

      const context =
        document.getElementById(
          "lineChart"
        ).getContext("2d");

      lineChart = new Chart(context, {
        type: "line",

        data: {
          labels: labels,

          datasets: [
            {
              label: "Temperature (°C)",
              data: temperatures,
              borderWidth: 2,
              tension: 0.2,
              yAxisID: "temperatureAxis"
            },
            {
              label: "Humidity (%)",
              data: humidities,
              borderWidth: 2,
              tension: 0.2,
              yAxisID: "humidityAxis"
            }
          ]
        },

        options: {
          responsive: true,
          maintainAspectRatio: false,

          interaction: {
            mode: "index",
            intersect: false
          },

          scales: {
            temperatureAxis: {
              type: "linear",
              position: "left",

              title: {
                display: true,
                text: "Temperature (°C)"
              }
            },

            humidityAxis: {
              type: "linear",
              position: "right",
              min: 0,
              max: 100,

              title: {
                display: true,
                text: "Humidity (%)"
              },

              grid: {
                drawOnChartArea: false
              }
            }
          }
        }
      });
    }

    function updateBarChart(latestRecord) {
      const values = latestRecord
        ? [
            latestRecord.temperature,
            latestRecord.humidity
          ]
        : [0, 0];

      if (barChart !== null) {
        barChart.data.datasets[0].data = values;
        barChart.update();
        return;
      }

      const context =
        document.getElementById(
          "barChart"
        ).getContext("2d");

      barChart = new Chart(context, {
        type: "bar",

        data: {
          labels: [
            "Temperature",
            "Humidity"
          ],

          datasets: [
            {
              label: "Latest value",
              data: values,
              borderWidth: 1
            }
          ]
        },

        options: {
          responsive: true,
          maintainAspectRatio: false,

          scales: {
            y: {
              beginAtZero: true
            }
          }
        }
      });
    }

    async function loadDashboardData() {
      try {
        hideError();

        const response = await fetch(
          "/api/data?limit=20",
          {
            cache: "no-store"
          }
        );

        if (!response.ok) {
          throw new Error(
            `HTTP error ${response.status}`
          );
        }

        const result = await response.json();
        const records = result.data ?? [];

        const latestRecord =
          records.length > 0
            ? records[records.length - 1]
            : null;

        updateLatestData(latestRecord);
        updateHistoryTable(records);
        updateLineChart(records);
        updateBarChart(latestRecord);
      } catch (error) {
        console.error(error);

        updateConnectionStatus(false);

        showError(
          "ไม่สามารถอ่านข้อมูลจาก Web API ได้"
        );
      }
    }

    // โหลดข้อมูลทันทีเมื่อเปิดหน้าเว็บ
    loadDashboardData();

    // อัปเดตข้อมูลทุก 5 วินาที
    setInterval(
      loadDashboardData,
      5000
    );
  </script>
</body>
</html>
```

---

## 4.3 การเปิด Dashboard

ตรวจสอบว่า Flask Server ยังทำงานอยู่ จากนั้นเปิด Web Browser แล้วเข้า

```text
http://127.0.0.1:5000
```

หากเปิดจากคอมพิวเตอร์เครื่องอื่น ให้ใช้ IP Address ของ Ubuntu

```text
http://192.168.1.100:5000
```

Dashboard จะแสดง

1. ค่าอุณหภูมิล่าสุด
2. ค่าความชื้นล่าสุด
3. Device ID
4. วันและเวลาที่ได้รับข้อมูล
5. กราฟเส้นแสดงข้อมูลย้อนหลัง 20 รายการ
6. กราฟแท่งแสดงค่าล่าสุด
7. ตารางข้อมูลย้อนหลัง
8. การอัปเดตข้อมูลอัตโนมัติทุก 5 วินาที

---

# การทดสอบระบบแบบครบวงจร

## ขั้นตอนที่ 1 ตรวจสอบ Flask Server

บน Ubuntu รันคำสั่ง

```bash
cd ~/aiot-workshop
source venv/bin/activate
python app.py
```

---

## ขั้นตอนที่ 2 ตรวจสอบ IP Address ของ Ubuntu

```bash
hostname -I
```

นำ IP Address ที่ได้ไปใส่ในโปรแกรม ESP8266

```cpp
const char* SERVER_URL =
  "http://192.168.1.100:5000/api/sensor";
```

---

## ขั้นตอนที่ 3 อัปโหลดโปรแกรมลง ESP8266

เปิด Arduino IDE แล้ว

1. ตรวจสอบชื่อ Wi-Fi
2. ตรวจสอบรหัสผ่าน Wi-Fi
3. ตรวจสอบ IP Address ของ Ubuntu
4. Verify โปรแกรม
5. Upload โปรแกรม
6. เปิด Serial Monitor ที่ `115200`

ตัวอย่างผลลัพธ์

```text
Wi-Fi connected
ESP8266 IP address: 192.168.1.120
Temperature: 29.40 °C | Humidity: 68.00 %
Sending JSON:
{"device_id":"esp8266-01","temperature":29.40,"humidity":68.00}
HTTP status code: 201
Server response: {"status":"success", ...}
```

---

## ขั้นตอนที่ 4 เปิด Web Dashboard

เปิด Browser แล้วไปที่

```text
http://192.168.1.100:5000
```

รอประมาณ 5 วินาทีเพื่อให้ Dashboard อัปเดตข้อมูล

---

# การตรวจสอบและแก้ไขปัญหา

## ปัญหา: DHT11 อ่านข้อมูลไม่ได้

ข้อความที่พบ

```text
Cannot read data from DHT11
```

แนวทางแก้ไข

1. ตรวจสอบการเชื่อมต่อ VCC, DATA และ GND
2. ตรวจสอบว่า DATA เชื่อมต่อกับ D4
3. ตรวจสอบชนิดเซนเซอร์ว่าเป็น DHT11
4. เพิ่มช่วงเวลาระหว่างการอ่านข้อมูล
5. ถอดและเสียบสาย USB ใหม่

---

## ปัญหา: ESP8266 เชื่อมต่อ Wi-Fi ไม่ได้

แนวทางแก้ไข

1. ตรวจสอบชื่อ SSID
2. ตรวจสอบรหัสผ่าน
3. ตรวจสอบว่าเครือข่ายรองรับความถี่ 2.4 GHz
4. นำ ESP8266 เข้าใกล้ Access Point
5. หลีกเลี่ยง Wi-Fi ที่ต้องเข้าสู่ระบบผ่านหน้าเว็บ

---

## ปัญหา: ESP8266 ส่งข้อมูลไปยัง Ubuntu ไม่ได้

แนวทางแก้ไข

1. ตรวจสอบว่า Flask Server กำลังทำงาน
2. ตรวจสอบ IP Address ของ Ubuntu
3. ตรวจสอบว่า ESP8266 และ Ubuntu อยู่ในเครือข่ายเดียวกัน
4. ตรวจสอบ Port `5000`
5. เปิด Firewall ด้วยคำสั่ง

```bash
sudo ufw allow 5000/tcp
```

6. ทดสอบจากคอมพิวเตอร์เครื่องอื่นด้วย URL

```text
http://IP_ADDRESS_OF_UBUNTU:5000/api/data
```

---

## ปัญหา: ได้ HTTP Status Code 400

หมายความว่าข้อมูล JSON ไม่ครบหรือรูปแบบไม่ถูกต้อง

ข้อมูลที่ถูกต้องต้องมีฟิลด์

```json
{
  "device_id": "esp8266-01",
  "temperature": 29.5,
  "humidity": 68.0
}
```

---

## ปัญหา: ได้ HTTP Status Code 415

หมายความว่า Request ไม่ได้กำหนดชนิดข้อมูลเป็น JSON

ในโปรแกรม ESP8266 ต้องมีคำสั่ง

```cpp
http.addHeader(
  "Content-Type",
  "application/json"
);
```

---

## ปัญหา: Dashboard ไม่มีกราฟ

1. ตรวจสอบว่าเครื่อง Ubuntu เชื่อมต่ออินเทอร์เน็ต เนื่องจากตัวอย่างนี้โหลด Bootstrap และ Chart.js ผ่าน CDN
2. เปิด Developer Tools ของ Browser เพื่อตรวจสอบข้อผิดพลาด
3. ตรวจสอบว่า `/api/data` ส่งข้อมูลได้
4. ทดลองเปิด

```text
http://127.0.0.1:5000/api/data
```

---

# แบบฝึกปฏิบัติท้ายกิจกรรม

## งานที่ 1 การกำหนดสถานะอุณหภูมิ

เพิ่มข้อมูล `status` โดยกำหนดว่า

* ต่ำกว่า 25°C แสดง `COOL`
* ตั้งแต่ 25–30°C แสดง `NORMAL`
* สูงกว่า 30°C แสดง `HOT`

---

## งานที่ 2 การเปลี่ยน Device ID

ให้แต่ละกลุ่มกำหนด Device ID ไม่ซ้ำกัน เช่น

```cpp
const char* DEVICE_ID = "group-01";
```

---

## งานที่ 3 การปรับช่วงเวลาส่งข้อมูล

เปลี่ยนจากส่งข้อมูลทุก 5 วินาทีเป็นทุก 10 วินาที

```cpp
delay(10000);
```

---

## งานที่ 4 การเพิ่มข้อมูลบน Dashboard

เพิ่มการแสดงผลดังต่อไปนี้

1. จำนวนข้อมูลทั้งหมด
2. อุณหภูมิสูงสุด
3. อุณหภูมิต่ำสุด
4. อุณหภูมิเฉลี่ย
5. ความชื้นเฉลี่ย

---

# สรุป

กิจกรรมนี้แสดงกระบวนการทำงานของระบบ IoT ตั้งแต่ต้นทางถึงปลายทาง ประกอบด้วย

```text
1. ESP8266 อ่านข้อมูลจาก DHT11
2. แสดงข้อมูลผ่าน Serial Monitor
3. สร้างข้อมูลในรูปแบบ JSON
4. ส่งข้อมูลด้วย HTTP POST
5. Flask Web API รับและตรวจสอบข้อมูล
6. บันทึกข้อมูลลงไฟล์ CSV
7. อ่านข้อมูลย้อนหลังจาก CSV
8. แสดงข้อมูลบน Web Dashboard
9. แสดงข้อมูลล่าสุด ตาราง กราฟเส้น และกราฟแท่ง
```

ผู้เรียนสามารถนำโครงสร้างโปรแกรมนี้ไปประยุกต์ใช้กับเซนเซอร์ชนิดอื่น โดยเปลี่ยนเฉพาะส่วนอ่านค่าเซนเซอร์และชื่อข้อมูลที่ส่งผ่าน JSON ส่วน Web API การจัดเก็บ CSV และ Dashboard สามารถใช้โครงสร้างเดิมเป็นพื้นฐานได้
