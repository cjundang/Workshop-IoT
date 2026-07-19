# การเขียนโปรแกรม IoT Node ด้วย ESP8266 ผ่าน Command Line

กิจกรรมนี้ใช้ **Ubuntu Terminal และ Arduino CLI 100%** โดยผู้เรียนจะพัฒนาโปรแกรมทีละขั้น ดังนี้

1. แสดงข้อความผ่าน Serial Monitor
2. อ่านค่าเซนเซอร์แสงผ่าน Analog Input
3. อ่านอุณหภูมิและความชื้นจาก DHT11
4. เชื่อมต่อเครือข่าย Wi-Fi
5. ส่งค่า Counter ไปยัง ThingSpeak
6. ส่งอุณหภูมิ ความชื้น และความเข้มแสงไปยัง ThingSpeak

---

# การเตรียมโครงการ

สร้างโฟลเดอร์หลัก:

```bash
mkdir -p "$HOME/esp8266-workshop"
cd "$HOME/esp8266-workshop"
```

กำหนดชื่อบอร์ด ESP8266:

```text
esp8266:esp8266:nodemcuv2
```

ตรวจสอบพอร์ต USB:

```bash
arduino-cli board list
```

ตัวอย่างพอร์ต:

```text
/dev/ttyUSB0
```

---

# กิจกรรมที่ 1 การสื่อสารผ่าน Serial Monitor

## วัตถุประสงค์

ผู้เรียนสามารถเขียนโปรแกรมให้ ESP8266 ส่งข้อความไปยังคอมพิวเตอร์ผ่าน Serial Port ได้

## 1.1 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab01_serial"
cd "$HOME/esp8266-workshop/lab01_serial"
```

## 1.2 สร้างโปรแกรม

```bash
cat > lab01_serial.ino <<'EOF'
int counter = 0;

void setup() {
  Serial.begin(115200);

  Serial.println();
  Serial.println("ESP8266 is ready");
}

void loop() {
  counter++;

  Serial.print("Counter: ");
  Serial.println(counter);

  delay(1000);
}
EOF
```

## 1.3 Compile

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

## 1.4 Upload

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

## 1.5 เปิด Serial Monitor

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
ESP8266 is ready
Counter: 1
Counter: 2
Counter: 3
```

ออกจาก Serial Monitor:

```text
Ctrl + C
```

---

# กิจกรรมที่ 2 อ่านค่าเซนเซอร์แสง

กิจกรรมนี้ใช้เซนเซอร์แสงแบบ Analog เช่น LDR Module

## 2.1 การต่อวงจร

| เซนเซอร์แสง | ESP8266 NodeMCU |
| ----------- | --------------- |
| VCC         | 3.3V            |
| GND         | GND             |
| AO          | A0              |

> ต้องใช้ขา `AO` หรือ Analog Output ไม่ใช่ขา `DO`

## 2.2 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab02_light"
cd "$HOME/esp8266-workshop/lab02_light"
```

## 2.3 สร้างโปรแกรม

```bash
cat > lab02_light.ino <<'EOF'
#define LIGHT_PIN A0

void setup() {
  Serial.begin(115200);

  Serial.println();
  Serial.println("Light sensor is ready");
}

void loop() {
  int lightValue = analogRead(LIGHT_PIN);

  Serial.print("Light value: ");
  Serial.println(lightValue);

  delay(1000);
}
EOF
```

ค่า Analog ของ ESP8266 จะแสดงเป็นตัวเลขในช่วงประมาณ:

```text
0-1023
```

ค่าที่อ่านได้อาจเพิ่มขึ้นหรือลดลงเมื่อมีแสง ทั้งนี้ขึ้นอยู่กับวงจรของโมดูลเซนเซอร์

## 2.4 Compile และ Upload

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

## 2.5 เปิด Serial Monitor

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
Light sensor is ready
Light value: 425
Light value: 438
Light value: 801
```

ทดลองใช้มือบังเซนเซอร์และสังเกตการเปลี่ยนแปลงของค่า

---

# กิจกรรมที่ 3 อ่านข้อมูลจาก DHT11

## 3.1 การต่อวงจร

| DHT11           | ESP8266 NodeMCU |
| --------------- | --------------- |
| VCC หรือ `+`    | 3.3V            |
| DATA หรือ `OUT` | D2              |
| GND หรือ `-`    | GND             |

## 3.2 ติดตั้งไลบรารี

ค้นหาไลบรารี:

```bash
arduino-cli lib search "DHT sensor library"
```

ติดตั้งไลบรารี DHT:

```bash
arduino-cli lib install "DHT sensor library"
```

ติดตั้งไลบรารีที่ต้องใช้ร่วมกัน:

```bash
arduino-cli lib install "Adafruit Unified Sensor"
```

ตรวจสอบไลบรารี:

```bash
arduino-cli lib list
```

## 3.3 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab03_dht11"
cd "$HOME/esp8266-workshop/lab03_dht11"
```

## 3.4 สร้างโปรแกรม

```bash
cat > lab03_dht11.ino <<'EOF'
#include <DHT.h>

#define DHT_PIN D2
#define DHT_TYPE DHT11

DHT dht(DHT_PIN, DHT_TYPE);

void setup() {
  Serial.begin(115200);

  dht.begin();

  Serial.println();
  Serial.println("DHT11 is ready");
}

void loop() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Cannot read DHT11");
    delay(2000);
    return;
  }

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.println("----------------");

  delay(2000);
}
EOF
```

## 3.5 Compile และ Upload

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เปิด Serial Monitor:

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
DHT11 is ready
Temperature: 30.20 C
Humidity: 67.00 %
----------------
```

---

# กิจกรรมที่ 4 เชื่อมต่อ Wi-Fi

## 4.1 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab04_wifi"
cd "$HOME/esp8266-workshop/lab04_wifi"
```

## 4.2 สร้างโปรแกรม

```bash
cat > lab04_wifi.ino <<'EOF'
#include <ESP8266WiFi.h>

const char* WIFI_NAME = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

void setup() {
  Serial.begin(115200);

  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(WIFI_NAME);

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("Wi-Fi connected");

  Serial.print("ESP8266 IP address: ");
  Serial.println(WiFi.localIP());
}

void loop() {
  Serial.print("Wi-Fi status: ");

  if (WiFi.status() == WL_CONNECTED) {
    Serial.println("Connected");
  } else {
    Serial.println("Disconnected");
  }

  delay(5000);
}
EOF
```

## 4.3 แก้ชื่อและรหัสผ่าน Wi-Fi

```bash
nano lab04_wifi.ino
```

แก้ไข:

```cpp
const char* WIFI_NAME = "ชื่อ-WiFi";
const char* WIFI_PASSWORD = "รหัสผ่าน-WiFi";
```

บันทึกและออกจาก Nano:

```text
Ctrl + O
Enter
Ctrl + X
```

## 4.4 Compile และ Upload

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เปิด Serial Monitor:

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
Connecting to School-WiFi
......
Wi-Fi connected
ESP8266 IP address: 192.168.1.25
Wi-Fi status: Connected
```

---

# กิจกรรมที่ 5 ส่งค่า Counter ไปยัง ThingSpeak

ThingSpeak ใช้ Channel สำหรับจัดเก็บข้อมูล โดยการเขียนข้อมูลต้องใช้ **Write API Key** ของ Channel และสามารถส่งค่าไปยัง `field1`, `field2` และ Field อื่น ๆ ผ่าน REST API ได้

## 5.1 เตรียม ThingSpeak Channel

สร้าง Channel และกำหนด:

```text
Field 1: Counter
```

จากนั้นเปิดแท็บ:

```text
API Keys
```

คัดลอกค่า:

```text
Write API Key
```

## 5.2 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab05_thingspeak"
cd "$HOME/esp8266-workshop/lab05_thingspeak"
```

## 5.3 สร้างโปรแกรม

```bash
cat > lab05_thingspeak.ino <<'EOF'
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>

const char* WIFI_NAME = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

const char* WRITE_API_KEY = "YOUR_WRITE_API_KEY";

int counter = 0;

void connectWiFi() {
  Serial.print("Connecting to Wi-Fi");

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("Wi-Fi connected");
}

void sendCounter() {
  counter++;

  String url = "https://api.thingspeak.com/update";
  url += "?api_key=";
  url += WRITE_API_KEY;
  url += "&field1=";
  url += String(counter);

  BearSSL::WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;

  Serial.print("Sending counter: ");
  Serial.println(counter);

  if (!http.begin(client, url)) {
    Serial.println("Cannot connect to ThingSpeak");
    return;
  }

  int httpCode = http.GET();

  Serial.print("HTTP response: ");
  Serial.println(httpCode);

  if (httpCode > 0) {
    Serial.print("ThingSpeak entry ID: ");
    Serial.println(http.getString());
  }

  http.end();
}

void setup() {
  Serial.begin(115200);

  connectWiFi();
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    sendCounter();
  } else {
    connectWiFi();
  }

  delay(20000);
}
EOF
```

`client.setInsecure()` ทำให้ตัวอย่างสั้นและเหมาะกับการทดลองในห้องเรียน แต่ระบบจริงควรตรวจสอบใบรับรอง TLS อย่างถูกต้อง

## 5.4 แก้ค่าที่กำหนด

```bash
nano lab05_thingspeak.ino
```

แก้ไข:

```cpp
const char* WIFI_NAME = "ชื่อ-WiFi";
const char* WIFI_PASSWORD = "รหัสผ่าน-WiFi";

const char* WRITE_API_KEY = "Write-API-Key";
```

## 5.5 Compile และ Upload

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เปิด Serial Monitor:

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
Wi-Fi connected
Sending counter: 1
HTTP response: 200
ThingSpeak entry ID: 1
```

---

# กิจกรรมที่ 6 ส่ง DHT11 และเซนเซอร์แสงไปยัง ThingSpeak

## 6.1 กำหนด Field ใน ThingSpeak

กำหนด Channel Fields ดังนี้:

| Field   | ข้อมูล      |
| ------- | ----------- |
| Field 1 | Counter     |
| Field 2 | Temperature |
| Field 3 | Humidity    |
| Field 4 | Light       |

ThingSpeak รองรับการส่งข้อมูลหลาย Field ภายในการอัปเดต Channel ครั้งเดียว โดยใช้พารามิเตอร์ `field1`, `field2` และ Field ลำดับถัดไป

## 6.2 สร้างโครงการ

```bash
mkdir -p "$HOME/esp8266-workshop/lab06_iot_node"
cd "$HOME/esp8266-workshop/lab06_iot_node"
```

## 6.3 สร้างโปรแกรม

```bash
cat > lab06_iot_node.ino <<'EOF'
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>
#include <WiFiClientSecureBearSSL.h>
#include <DHT.h>

#define DHT_PIN D2
#define DHT_TYPE DHT11
#define LIGHT_PIN A0

DHT dht(DHT_PIN, DHT_TYPE);

const char* WIFI_NAME = "YOUR_WIFI_NAME";
const char* WIFI_PASSWORD = "YOUR_WIFI_PASSWORD";

const char* WRITE_API_KEY = "YOUR_WRITE_API_KEY";

unsigned long counter = 0;

void connectWiFi() {
  Serial.print("Connecting to Wi-Fi");

  WiFi.begin(WIFI_NAME, WIFI_PASSWORD);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println();
  Serial.println("Wi-Fi connected");

  Serial.print("IP address: ");
  Serial.println(WiFi.localIP());
}

void sendToThingSpeak(
  float temperature,
  float humidity,
  int lightValue
) {
  counter++;

  String url = "https://api.thingspeak.com/update";

  url += "?api_key=";
  url += WRITE_API_KEY;

  url += "&field1=";
  url += String(counter);

  url += "&field2=";
  url += String(temperature, 1);

  url += "&field3=";
  url += String(humidity, 1);

  url += "&field4=";
  url += String(lightValue);

  BearSSL::WiFiClientSecure client;
  client.setInsecure();

  HTTPClient http;

  if (!http.begin(client, url)) {
    Serial.println("Cannot connect to ThingSpeak");
    return;
  }

  int httpCode = http.GET();

  Serial.print("HTTP response: ");
  Serial.println(httpCode);

  if (httpCode > 0) {
    Serial.print("ThingSpeak entry ID: ");
    Serial.println(http.getString());
  }

  http.end();
}

void setup() {
  Serial.begin(115200);

  dht.begin();

  connectWiFi();

  Serial.println("IoT Node is ready");
}

void loop() {
  float temperature = dht.readTemperature();
  float humidity = dht.readHumidity();

  int lightValue = analogRead(LIGHT_PIN);

  if (isnan(temperature) || isnan(humidity)) {
    Serial.println("Cannot read DHT11");
    delay(2000);
    return;
  }

  Serial.println("--------------------");

  Serial.print("Temperature: ");
  Serial.print(temperature);
  Serial.println(" C");

  Serial.print("Humidity: ");
  Serial.print(humidity);
  Serial.println(" %");

  Serial.print("Light value: ");
  Serial.println(lightValue);

  if (WiFi.status() != WL_CONNECTED) {
    connectWiFi();
  }

  sendToThingSpeak(
    temperature,
    humidity,
    lightValue
  );

  // ส่งข้อมูลทุก 20 วินาทีระหว่างทดลอง
  delay(20000);
}
EOF
```

## 6.4 แก้ค่าการเชื่อมต่อ

```bash
nano lab06_iot_node.ino
```

แก้ไข:

```cpp
const char* WIFI_NAME = "ชื่อ-WiFi";
const char* WIFI_PASSWORD = "รหัสผ่าน-WiFi";

const char* WRITE_API_KEY = "Write-API-Key";
```

## 6.5 Compile

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

## 6.6 Upload

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

## 6.7 เปิด Serial Monitor

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

ผลลัพธ์ตัวอย่าง:

```text
IoT Node is ready
--------------------
Temperature: 30.20 C
Humidity: 67.00 %
Light value: 625
HTTP response: 200
ThingSpeak entry ID: 15
```

จากนั้นเปิดหน้า ThingSpeak Channel เพื่อตรวจสอบกราฟของ:

```text
Counter
Temperature
Humidity
Light
```

---

# เปลี่ยนระยะเวลาเป็น 5 นาที

ระหว่างการทดลองใช้:

```cpp
delay(20000);
```

หมายถึงส่งข้อมูลทุก 20 วินาที

เมื่อระบบทำงานถูกต้องแล้ว เปลี่ยนเป็น:

```cpp
delay(300000);
```

เนื่องจาก:

```text
5 นาที × 60 วินาที × 1,000 มิลลิวินาที
= 300,000 มิลลิวินาที
```

---

# ชุดคำสั่งสำหรับ Compile และ Upload

เข้าโฟลเดอร์โครงการ:

```bash
cd "$HOME/esp8266-workshop/lab06_iot_node"
```

Compile:

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

Upload:

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เปิด Serial Monitor:

```bash
arduino-cli monitor \
-p /dev/ttyUSB0 \
--config baudrate=115200
```

---

# สรุปลำดับการเรียนรู้

```text
กิจกรรมที่ 1
แสดงข้อความผ่าน Serial Monitor
        ↓
กิจกรรมที่ 2
อ่านค่า Analog จากเซนเซอร์แสง
        ↓
กิจกรรมที่ 3
อ่านอุณหภูมิและความชื้นจาก DHT11
        ↓
กิจกรรมที่ 4
เชื่อมต่อ ESP8266 กับ Wi-Fi
        ↓
กิจกรรมที่ 5
ส่งค่า Counter ไปยัง ThingSpeak
        ↓
กิจกรรมที่ 6
ส่ง Counter, Temperature, Humidity และ Light
ไปยัง ThingSpeak
```
