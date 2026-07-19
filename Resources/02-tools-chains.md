# การเตรียม Ubuntu เพื่อเขียนโปรแกรม ESP8266 ด้วย Command Line 100%

กระบวนการนี้โดยเริ่มจาก Ubuntu ที่ยังไม่มีเครื่องมือใด ๆ จนสามารถ:

```text
เขียนโปรแกรม → Compile → เชื่อมต่อ ESP8266 → Upload → ดูผลผ่าน Serial Monitor
```

เครื่องมือหลักคือ **Arduino CLI** ซึ่งรองรับการติดตั้งบอร์ด ไลบรารี การคอมไพล์ และการอัปโหลดโปรแกรมจาก Terminal โดยไม่ต้องใช้ Arduino IDE  

---

## 1. อุปกรณ์ที่ใช้

* เครื่องคอมพิวเตอร์ Ubuntu
* บอร์ด NodeMCU ESP8266
* สาย Micro USB ที่ส่งข้อมูลได้
* เซนเซอร์ DHT11
* สาย Jumper
* เครือข่าย Wi-Fi

การต่อ DHT11:

| DHT11           | NodeMCU ESP8266 |
| --------------- | --------------- |
| VCC หรือ `+`    | 3.3V            |
| DATA หรือ `OUT` | D2              |
| GND หรือ `-`    | GND             |

---

# ส่วนที่ 1 ติดตั้งเครื่องมือบน Ubuntu

## 2. เปิด Terminal

กดปุ่ม:

```text
Ctrl + Alt + T
```

ตรวจสอบระบบ:

```bash
uname -a
```

---

## 3. อัปเดตรายการแพ็กเกจ

```bash
sudo apt update
```

ติดตั้งเครื่องมือพื้นฐาน:

```bash
sudo apt install -y curl git ca-certificates
```

---

## 4. ติดตั้ง Arduino CLI

สร้างโฟลเดอร์สำหรับเก็บโปรแกรม:

```bash
mkdir -p "$HOME/bin"
```

ดาวน์โหลดและติดตั้ง Arduino CLI:

```bash
curl -fsSL \
https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh \
| BINDIR="$HOME/bin" sh
```

คำสั่งติดตั้งอย่างเป็นทางการของ Arduino จะดาวน์โหลด Arduino CLI รุ่นล่าสุด และสามารถกำหนดตำแหน่งติดตั้งด้วย `BINDIR` ได้  

เพิ่มโฟลเดอร์ `bin` เข้าไปใน PATH:

```bash
echo 'export PATH="$HOME/bin:$PATH"' >> "$HOME/.bashrc"
```

โหลดค่าใหม่:

```bash
source "$HOME/.bashrc"
```

ตรวจสอบการติดตั้ง:

```bash
arduino-cli version
```

ควรปรากฏข้อมูลคล้าย:

```text
arduino-cli Version: 1.x.x
```

---

# ส่วนที่ 2 ติดตั้งเครื่องมือสำหรับ ESP8266

## 5. สร้างไฟล์ตั้งค่า Arduino CLI

```bash
arduino-cli config init
```

ตรวจสอบไฟล์ตั้งค่า:

```bash
arduino-cli config dump
```

---

## 6. เพิ่มแหล่งดาวน์โหลด ESP8266

ใช้คำสั่ง:

```bash
arduino-cli config add \
board_manager.additional_urls \
https://arduino.esp8266.com/stable/package_esp8266com_index.json
```

ESP8266 Arduino Core ระบุ URL นี้เป็นแหล่งติดตั้งแพ็กเกจบอร์ดอย่างเป็นทางการ  

ตรวจสอบว่าเพิ่มสำเร็จ:

```bash
arduino-cli config dump
```

ควรเห็น:

```yaml
board_manager:
  additional_urls:
    - https://arduino.esp8266.com/stable/package_esp8266com_index.json
```

---

## 7. อัปเดตรายการบอร์ด

```bash
arduino-cli core update-index
```

ค้นหาแพ็กเกจ ESP8266:

```bash
arduino-cli core search esp8266
```

ติดตั้ง ESP8266 Core:

```bash
arduino-cli core install esp8266:esp8266
```

คำสั่ง `core install` จะติดตั้ง Core พร้อมเครื่องมือที่จำเป็นสำหรับบอร์ดนั้น  
ตรวจสอบ Core ที่ติดตั้ง:

```bash
arduino-cli core list
```

ควรพบ:

```text
esp8266:esp8266
```

---

## 8. ค้นหาชื่อบอร์ด NodeMCU

```bash
arduino-cli board listall | grep -i nodemcu
```

โดยทั่วไป NodeMCU 1.0 ใช้รหัสบอร์ด:

```text
esp8266:esp8266:nodemcuv2
```

รหัสนี้เรียกว่า **FQBN** หรือ Fully Qualified Board Name

---

# ส่วนที่ 3 ตรวจสอบการเชื่อมต่อ ESP8266

## 9. ต่อบอร์ดกับคอมพิวเตอร์

เสียบสาย USB ระหว่าง ESP8266 กับเครื่อง Ubuntu แล้วรัน:

```bash
arduino-cli board list
```

คำสั่ง `board list` ใช้แสดงบอร์ดและพอร์ตที่เชื่อมต่อกับเครื่อง  

ตัวอย่างผลลัพธ์:

```text
Port         Protocol Type              Board Name
/dev/ttyUSB0 serial   Serial Port (USB) Unknown
```

บางครั้ง Arduino CLI อาจแสดง `Unknown` แต่ยังสามารถอัปโหลดได้ โดยระบุ FQBN เอง

ตรวจสอบพอร์ตเพิ่มเติม:

```bash
ls /dev/ttyUSB*
```

หรือ:

```bash
ls /dev/ttyACM*
```

สำหรับ NodeMCU ESP8266 พอร์ตมักเป็น:

```text
/dev/ttyUSB0
```

---

## 10. แก้ปัญหา Permission ของ USB

เพิ่มผู้ใช้เข้าในกลุ่ม `dialout`:

```bash
sudo usermod -aG dialout "$USER"
```

จากนั้นออกจากระบบ Ubuntu แล้วเข้าสู่ระบบใหม่ หรือ Restart เครื่อง:

```bash
sudo reboot
```

หลังจากเปิดเครื่องกลับมา ตรวจสอบกลุ่ม:

```bash
groups
```

ควรเห็นคำว่า:

```text
dialout
```

ตรวจสอบพอร์ตอีกครั้ง:

```bash
arduino-cli board list
```

---

# ส่วนที่ 4 ทดลองโปรแกรม Blink

ควรทดลอง Blink ก่อน เพื่อยืนยันว่าการ Compile และ Upload ทำงานสมบูรณ์

## 11. สร้างโฟลเดอร์โปรแกรม

```bash
mkdir -p "$HOME/esp8266-workshop/blink"
```

เข้าไปยังโฟลเดอร์:

```bash
cd "$HOME/esp8266-workshop/blink"
```

ข้อสำคัญคือชื่อไฟล์ `.ino` ต้องตรงกับชื่อโฟลเดอร์:

```text
โฟลเดอร์: blink
ไฟล์:     blink.ino
```

---

## 12. สร้างโปรแกรม Blink

ใช้คำสั่งต่อไปนี้:

```bash
cat > blink.ino <<'EOF'
#define LED_PIN LED_BUILTIN

void setup() {
  pinMode(LED_PIN, OUTPUT);
}

void loop() {
  digitalWrite(LED_PIN, LOW);
  delay(1000);

  digitalWrite(LED_PIN, HIGH);
  delay(1000);
}
EOF
```

ดูเนื้อหาไฟล์:

```bash
cat blink.ino
```

---

## 13. Compile โปรแกรม

```bash
arduino-cli compile \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เมื่อสำเร็จจะเห็นข้อมูลคล้าย:

```text
Sketch uses ... bytes
Global variables use ... bytes
```

กระบวนการ Compile จะแปลง Sketch เป็น C++ ค้นหาไลบรารี และใช้ Compiler สร้างไฟล์ที่บอร์ดสามารถประมวลผลได้ ([Arduino][6])

---

## 14. Upload โปรแกรม

ตรวจสอบพอร์ตก่อน:

```bash
arduino-cli board list
```

กรณีพอร์ตเป็น `/dev/ttyUSB0` ให้ใช้:

```bash
arduino-cli upload \
-p /dev/ttyUSB0 \
--fqbn esp8266:esp8266:nodemcuv2 \
.
```

เมื่อสำเร็จควรเห็นข้อความคล้าย:

```text
Writing at ...
Hash of data verified.
Leaving...
Hard resetting via RTS pin...
```

LED บนบอร์ดควรกะพริบทุก 1 วินาที
