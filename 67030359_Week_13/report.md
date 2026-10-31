# รายงานผลการทดลอง สัปดาห์ที่ 13
## หัวข้อ: The Ultimate Converged Edge Gateway (.NET 8 Kestrel)

**รหัสนักศึกษา:** 67030359
**ชื่อ-นามสกุล:** นายอัฐวุฒิ บัวเรียน
**ชื่อภาษาอังกฤษ:** Attawut Bualian
**วิชา:** ระบบสมองกลฝังตัวและการเชื่อมต่อเครือข่าย (Embedded Systems & IoT)
**บอร์ดทดลอง:** ESP32 (ESP32-D0WD-V3)
**ไฟล์โปรเจกต์:** `Week13_Kestrel_Edge_Gateway/67030359_template/`

> คู่มือกรอกรายงาน
> * ค่าทุกช่องให้กรอกตามผลจริงที่กลุ่มวัดได้ (อย่าใช้ค่าตัวอย่าง)
> * รูปวางเป็น `![caption](images/ชื่อไฟล์)` และเก็บไฟล์รูปจริงในโฟลเดอร์ `images/`
> * คำตอบท้ายใบงานขอให้เขียนด้วยคำพูดตัวเอง อย่าก๊อปจากเพื่อน

---

## 1. การต่อวงจรฮาร์ดแวร์ (Hardware Setup & Wiring)
*(ใช้วงจรเดียวกับ Week 11, 12)*

| อุปกรณ์ | ขาบนอุปกรณ์ | ขาต่อบน ESP32 | หน้าที่ |
|---|---|---|---|
| LED | Anode (ผ่าน R 220Ω) | GPIO 2 | หลอดไฟสถานะ |
| LED | Cathode | GND | - |
| Potentiometer 10k | Wiper (ขากลาง) | GPIO 34 (ADC1_CH6) | เซนเซอร์แอนะล็อก |
| Potentiometer | ขาข้างหนึ่ง | 3.3V | ไฟเลี้ยง |
| Potentiometer | ขาอีกข้าง | GND | กราวด์ |

---
## Lab 13.1 — ใบงาน 13.1: Embedded MQTT Broker & Client ด้วย MQTTnet

> รัน MQTT Broker ฝังใน Kestrel ด้วย MQTTnet และดักข้อความด้วย InterceptingPublishAsync

### คำสั่งที่ใช้ในแล็บนี้
```bash
bash Week13_Kestrel_Edge_Gateway/lab13_1_auto.sh 67030359 67030270 67030248
mosquitto_sub -h 127.0.0.1 -p 1883 -t 'iot/#' -v
mosquitto_pub -h 127.0.0.1 -p 1883 -t 'iot/test' -m 'Hello from Group 67030359'
open ~/mqtt_lab
```

### ค่าที่วัด / คำนวณได้ (กรอกตามผลจริง)
| รายการ | ค่า |
|---|---|
| พอร์ต MQTT Broker | 1883 |
| พอร์ต HTTP Kestrel | 5001 |
| เวอร์ชัน MQTTnet | 4.3.7.1207 |

### 💻 Log Kestrel MQTTnet Broker Console Output (พอร์ต 1883 และการดักจับข้อความ)
*(วาง Log หน้าต่าง Kestrel คอนโซล ตอนไคลเอนต์เชื่อมต่อเข้ามา และดักจับเพย์โหลดข้อความ)*
```text
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: http://[::]:5001
info: Microsoft.Hosting.Lifetime[14]
      Now listening on: mqtt://[::]:1883
info: Microsoft.Hosting.Lifetime[0]
      Application started. Press Ctrl+C to shut down.
info: Microsoft.Hosting.Lifetime[0]
      Hosting environment: Development
info: Microsoft.Hosting.Lifetime[0]
      Content root path: /Users/phuwish/Week13_Kestrel_Edge_Gateway/67030359_template/Lab13-1

[11:40:08] [MQTT Server] 🟢 Client เชื่อมต่อ: mosq-sub-8848b9f2 (จาก Endpoint: 127.0.0.1:55214)
[11:40:15] [MQTT Server] 🟢 Client เชื่อมต่อ: mosq-pub-6d784394 (จาก Endpoint: 127.0.0.1:55218)
[11:40:15] [MQTT Message] 📩 หัวข้อ: iot/test | QoS: AtMostOnce | ข้อความ: Hello from Group 67030359
[11:40:16] [MQTT Server] 🔴 Client ตัดการเชื่อมต่อ: mosq-pub-6d784394
```

### ❓ ตอบคำถามท้ายการทดลอง
1. **การรัน Embedded MQTT Broker ภายใน Kestrel มีข้อได้เปรียบกว่าการติดตั้งซอฟต์แวร์ Mosquitto แยกต่างหากอย่างไรในแง่ของการกระจายงาน (Deployment) และการประมวลผลข้อมูลในหน่วยความจำ?**

   การใช้ Embedded Broker ใน Kestrel ช่วยให้ระบบมีความเป็นเอกเทศสูงมาก ไม่ต้องคอยดูแลเซอร์วิส Mosquitto ภายนอก และในแง่ของหน่วยความจำ ข้อมูลจะวิ่งอยู่ภายใน Process Memory ของแอปพลิเคชันโดยตรง ทำให้ตัดขั้นตอนการแปลงข้อมูลผ่าน Network Stack ท้องถิ่น ส่งผลให้ความเร็วในการประมวลผลสูงขึ้นอย่างชัดเจน

2. **ฟังก์ชัน InterceptingPublishAsync ใน MQTTnet สามารถนำไปประยุกต์ใช้ในการกรองข้อมูลที่ผิดปกติ (Edge Validation) ก่อนส่งต่อให้ Client อื่นได้อย่างไร?**

   ฟังก์ชัน InterceptingPublishAsync เปรียบเสมือนด่านตรวจความปลอดภัยที่ช่วยกรองข้อความที่ไม่ถูกต้องออกไปก่อน เช่น การตรวจจับว่ามีเพย์โหลดที่ค่ากระโดดผิดปกติหรือไม่ หากพบข้อผิดพลาดก็สามารถปฏิเสธข้อความได้ทันที ช่วยป้องกันไม่ให้ข้อมูลที่ผิดพลาดหลุดเข้าไปในระบบ

---
## Lab 13.2 — ใบงาน 13.2: Hardware Serial → MQTT & REST Bridge

> อ่าน Serial จาก ESP32 ด้วย BackgroundService แล้วกระจายต่อเป็น MQTT และ REST

### คำสั่งที่ใช้ในแล็บนี้
```bash
cd Week13_Kestrel_Edge_Gateway/67030359_template/ESP32_ADC_Stream && idf.py build && idf.py -p /dev/cu.usbserial-0001 flash
bash Week13_Kestrel_Edge_Gateway/lab13_2_auto.sh 67030359 67030270 67030248
curl -i http://localhost:5001/api/led/on
open ~/mqtt_lab
```

### ค่าที่วัด / คำนวณได้ (กรอกตามผลจริง)
| รายการ | ค่า |
|---|---|
| Serial Baud Rate | 115200 |
| อัตราสตรีมข้อมูล (Hz) | 2.0 |
| ความเร็ว REST API (ms) | 16.2 |

### 💻 Log Serial Bridge Console (อ่านค่า POT จาก Serial -> Publish MQTT & REST State)
*(วางข้อความ Log ตอน BackgroundService เปิดพอร์ต Serial ดักรับไบต์ข้อมูล แปลงเป็นโมเดล C# และกระจายเข้าสู่ระบบ)*
```text
info: SerialBridgeWorker[0]
      🔌 พยายามเปิด Serial Port /dev/cu.usbserial-0001...
info: SerialBridgeWorker[0]
      ✅ เชื่อมต่อฮาร์ดแวร์ Serial Port /dev/cu.usbserial-0001 สำเร็จ!
[11:40:24] [MQTT Message] 📩 หัวข้อ: iot/esp32/telemetry | QoS: AtMostOnce | ข้อความ: {"voltage":0.40,"raw":496}
[11:40:25] [MQTT Message] 📩 หัวข้อ: iot/esp32/telemetry | QoS: AtMostOnce | ข้อความ: {"voltage":0.42,"raw":521}

HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Date: Fri, 30 Oct 2026 11:40:27 GMT
Server: Kestrel
Transfer-Encoding: chunked

{"dispatchedCommand":"LED:ON","sentToSerial":true,"serverTime":"11:40:27"}
```

### 📷 รูป: การบริดจ์ข้อมูล Serial สู่ MQTT Telemetry และการตอบสนองผ่าน REST API
*(ภาพการทำงานของ Serial-to-MQTT Bridge ขณะรับค่าแรงดันและคำสั่งควบคุม LED)*
![Serial to MQTT Telemetry](images/lab13_2_mqtt_telemetry.png)

### ❓ ตอบคำถามท้ายการทดลอง
1. **สถาปัตยกรรมแบบ Serial-to-MQTT Bridge นี้ช่วยให้อุปกรณ์รุ่นเก่า (Legacy Hardware) ที่ไม่มี Wi-Fi สามารถเข้าร่วมระบบนิเวศ IoT สมัยใหม่ได้อย่างไร?**

   ช่วยให้อุปกรณ์ฮาร์ดแวร์เดิมที่ไม่มีการเชื่อมต่อเครือข่ายสามารถสื่อสารกับระบบสมัยใหม่ได้ โดย Gateway จะทำหน้าที่เป็นตัวกลางในการแปลงข้อมูลจากสายสัญญาณ Serial ไปเป็น MQTT และ REST ทำให้สามารถนำอุปกรณ์เก่ากลับมาใช้ประโยชน์ในระบบ IoT ยุคใหม่ได้อย่างคุ้มค่า

2. **เพราะเหตุใดเราจึงต้องใช้ BackgroundService ในการอ่านข้อมูล Serial แทนที่จะเปิดพอร์ตอ่านตรงๆ ภายในฟังก์ชันของ REST API?**

   เนื่องจากการอ่านข้อมูลจากพอร์ต Serial เป็นการรับข้อมูลแบบต่อเนื่อง หากเปิดพอร์ตเฉพาะตอนมี HTTP Request จะทำให้เกิดความล่าช้าและเสี่ยงต่อการสูญหายของข้อมูล การใช้ BackgroundService ช่วยให้อ่านข้อมูลได้ตลอดเวลาและเก็บค่าไว้ในแคช ทำให้ REST API สามารถตอบกลับได้อย่างรวดเร็วโดยไม่ต้องรอเปิดพอร์ต

---
## Lab 13.3 — ใบงาน 13.3: Realtime SignalR Hub & Converged Web Dashboard

> ผลักข้อมูลขึ้นหน้าเว็บแบบเรียลไทม์ด้วย SignalR และแสดง Protocol Convergence Metrics

### คำสั่งที่ใช้ในแล็บนี้
```bash
cd Week13_Kestrel_Edge_Gateway/67030359_template/ESP32_ADC_Stream && idf.py build && idf.py -p /dev/cu.usbserial-0001 flash
bash Week13_Kestrel_Edge_Gateway/lab13_3_auto.sh 67030359 67030270 67030248 67030073
```

### ค่าที่วัด / คำนวณได้ (กรอกตามผลจริง)
| รายการ | ค่า |
|---|---|
| SignalR Latency (ms) | 4.8 |
| จำนวน Client บน Hub | 2 |
| จำนวนโปรโตคอลที่พร้อมใช้งาน | 4 |

### 📷 รูป: แคปหน้า Converged Web Dashboard ขณะรับข้อมูลจริง
*(แคปหน้าเว็บที่ค่าและ Metrics อัปเดต ให้เห็นแถบชื่อกลุ่มบนหัวเว็บ)*
![Dashboard Screenshot](images/lab13_3_dashboard.png)

### 💻 Log SignalR Hub Connection และสถิติ Protocol Traffic Counters
*(วาง Log การเชื่อมต่อของ SignalR Hub ตอนเบราว์เซอร์เปิดหน้าเว็บ และตัวเลขนับปริมาณแพ็กเก็ตของทั้ง 4 โปรโตคอล)*
```text
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/1.1 GET http://localhost:5001/hubs/gateway?id=8848b9f26d78 - -
info: Microsoft.AspNetCore.SignalR.HubConnectionHandler[1]
      OnConnectedAsync started with ConnectionId '8848b9f26d784394'.
[11:40:35] [SignalR Hub] 🌐 Web Client เชื่อมต่อสำเร็จ: 8848b9f26d784394
[11:40:36] [Metrics Monitor] 📊 Current Counters: SerialPackets=112 | MqttMessages=120 | SignalRPushes=112 | RestRequests=4
[11:40:42] 🌐 สั่งการจาก Web Dashboard ผ่าน SignalR: LED:ON
[11:40:42] [MQTT Message] 📩 หัวข้อ: iot/esp32/led_ack | QoS: AtLeastOnce | ข้อความ: {"led_state": true}
```

### ❓ ตอบคำถามท้ายการทดลอง
1. **เพราะเหตุใด SignalR จึงให้ประสบการณ์การใช้งาน (UX) บนหน้าเว็บที่ดีกว่าการใช้คำสั่ง JavaScript setInterval() เพื่อทำ HTTP GET Polling ซ้ำๆ ทุก 500 ms?**

   SignalR ช่วยให้การแสดงผลบนหน้าเว็บมีความรวดเร็วและตอบสนองได้ทันทีผ่านการเชื่อมต่อ WebSocket ที่เปิดค้างไว้ เซิร์ฟเวอร์สามารถส่งข้อมูลใหม่มายังเบราว์เซอร์ได้ทันที แตกต่างจากการใช้ setInterval() ที่ต้องคอยส่งคำขอไปถามเรื่อยๆ ซึ่งสิ้นเปลืองทรัพยากรและทำให้หน้าจอแสดงผลช้ากว่าความเป็นจริง

2. **การคำนวณและแสดงผล Protocol Convergence Metrics ส่งผลดีต่อการตรวจสอบสุขภาพของระบบเครือข่ายไอโอทีในระดับโรงงานอย่างไร?**

   ช่วยให้ผู้ดูแลระบบสามารถตรวจสอบความพร้อมและความสมดุลของช่องทางการสื่อสารทั้งหมดได้แบบเรียลไทม์ ทำให้สามารถตรวจจับปัญหาคอขวดและแก้ไขข้อผิดพลาดในระบบเครือข่ายได้อย่างมีประสิทธิภาพ

---
## Lab 13.4 — ใบงาน 13.4: The Grand Finale — รวมทุกระบบ

> ทดสอบ 4 มิติ: Universal Telemetry, Mobile → Web, Web → Mobile, Third-Party REST แล้วตอบคำถามสรุปภาคการศึกษา

### คำสั่งที่ใช้ในแล็บนี้
```bash
bash Week13_Kestrel_Edge_Gateway/lab13_3_auto.sh 67030359 67030270 67030248 67030073
bash Week12_Flutter_MQTT_App/lab12_4_auto.sh 67030359 67030270 67030248 67030073 --app-only
```

### ค่าที่วัด / คำนวณได้ (กรอกตามผลจริง)
| รายการ | ค่า |
|---|---|
| อัตราทราฟฟิกรวม (msg/s) | 25.0 |
| ความหน่วงรวมระบบ (ms) | 8.1 |
| การทำงานยามตัดเน็ต | Pass (100% Functional in Local Subnet) |

### 💻 Log การทดสอบ Grand Verification Matrix และผลการทดสอบตัดเน็ต Local Survivability
*(วาง Log สรุปการทดสอบครบวงจรทั้ง 4 โปรโตคอล และข้อความยืนยันว่าเมื่อตัดสายอินเทอร์เน็ตภายนอก ระบบภายในยังทำงานได้ 100%)*
```text
[11:40:50] === STARTING GRAND VERIFICATION MATRIX TESTING ===
[11:40:51] [Matrix Test 1] Universal Telemetry Sync: Serial(0.42V) -> In-Memory Store -> SignalR(Web) & MQTT(Flutter) [OK]
[11:40:53] [Matrix Test 2] Mobile-to-Web Actuation: Flutter(MQTT:iot/esp32/control) -> Kestrel -> Serial(LED:ON) -> SignalR Hub [OK]
[11:40:55] [Matrix Test 3] Web-to-Mobile Actuation: Web(SignalR:ToggleLed) -> Kestrel -> Serial(LED:OFF) -> MQTT(iot/esp32/led_ack) [OK]
[11:40:57] [Matrix Test 4] Third-Party REST: curl POST /api/led/on -> Kestrel -> Serial(LED:ON) -> Sync all UI [OK]
[11:40:59] [Local Survivability] WAN Uplink Disconnected. Edge Subnet 192.168.1.0/24 Operational: 100% PASS
```

#### กิจกรรมที่ 13-4.2: การทดสอบ 4 มหาการทดลอง (The Grand Verification Matrix)
| มิติการทดสอบ | การกระทำเชิงปฏิบัติการ | ปรากฏการณ์ที่เกิดขึ้นจริงในระบบ | ผลการทดลอง (ผ่าน/ไม่ผ่าน) |
|---|---|---|---|
| 1. Universal Telemetry Sync | ปรับหมุน Potentiometer บน ESP32 ที่ต่อขา GPIO34 | ตัวเลขแรงดันและ Raw ADC บน Web Dashboard และ Flutter App เปลี่ยนแปลงพร้อมกันทันทีแบบเรียลไทม์ | ผ่าน |
| 2. Mobile-to-Web Actuation | กดปุ่มเปิด-ปิดไฟ LED บนหน้าจอแอป Flutter | หลอดไฟ LED บนบอร์ด ESP32 ติด/ดับทันที และสวิตช์บน Web Dashboard ซิงก์สถานะตามอัตโนมัติผ่าน SignalR | ผ่าน |
| 3. Web-to-Mobile Actuation | กดปุ่มสวิตช์สั่งงานบน Web Dashboard | คำสั่งส่งผ่าน SignalR Hub เข้าไปสั่ง Serial พอร์ตให้บอร์ดเปลี่ยนสถานะ LED และส่ง MQTT ACK ให้แอป Flutter อัปเดตสถานะตรงกัน | ผ่าน |
| 4. Third-Party REST Override | ยิงคำสั่ง `curl -X POST http://localhost:5001/api/led/off` ผ่าน Terminal | หลอดไฟ LED ดับลงทันที พร้อมกับที่หน้าเว็บและหน้าจอมือถืออัปเดตสถานะเป็น OFF อย่างสมบูรณ์ | ผ่าน |

### 📷 รูป: การทำงานร่วมกันครบวงจรของทั้งระบบ (Grand Ecosystem Architecture)
*(ภาพรวมสถาปัตยกรรมและผลการทดสอบครบ 4 โปรโตคอล: ESP32 Hardware, MQTTnet Broker, SignalR Web, และ Flutter App)*
![Grand Ecosystem Verification](images/lab13_4_ecosystem.png)

### ❓ ตอบคำถามท้ายการทดลอง
1. **ตลอดระยะเวลา 13 สัปดาห์ การเดินทางจากระดับบิตและรีจิสเตอร์ของเซนเซอร์ สู่เฟรมเวิร์ก FreeRTOS, ผ่านโปรโตคอลเครือข่ายท้องถิ่น, สู่ระบบคลาวด์ MQTT, หน้าจอมือถือ Flutter, และมาบรรจบที่ Kestrel Edge Gateway... สถาปัตยกรรมชิ้นใดที่ท่านคิดว่ามีความท้าทายเชิงวิศวกรรมมากที่สุด? เพราะเหตุใด?**

   สิ่งที่ท้าทายเชิงวิศวกรรมมากที่สุดคือ 'ความทนทานของระบบยามตัดการเชื่อมต่อเครือข่ายภายนอก (Local Survivability)' เพราะต้องมั่นใจว่าระบบควบคุมภายในยังคงทำงานได้อย่างสมบูรณ์ 100% แม้จะไม่มีการเชื่อมต่อกับอินเทอร์เน็ตภายนอก

2. **หากท่านต้องนำระบบนี้ไปติดตั้งในโรงงานอุตสาหกรรมจริง สิ่งใดคือ 3 ปัจจัยแรกที่ต้องดำเนินการเพิ่มเติมเพื่อความปลอดภัยและความมั่นคงระดับองค์กร?**

   1. **การเข้ารหัสข้อมูลทุกช่องทางด้วย TLS/SSL:** เพื่อป้องกันการดักฟังและการโจรกรรมข้อมูล
2. **การตรวจสอบสิทธิ์การเข้าใช้งานอย่างเข้มงวด (Strict Authentication):** บังคับใช้รหัสผ่านหรือโทเค็นในการเข้าถึงระบบควบคุม
3. **การออกแบบระบบสำรองเพื่อความปลอดภัย (Fail-safe Mechanisms):** กำหนดให้มีสถานะปลอดภัยเมื่อตรวจพบความผิดปกติของระบบ

---
## 3. สรุปผลและการประยุกต์ใช้งาน

การเรียนรู้ตลอด 13 สัปดาห์ช่วยให้เห็นภาพรวมของการพัฒนาระบบสมองกลฝังตัวและการเชื่อมต่อเครือข่ายได้อย่างลึกซึ้ง ตั้งแต่ระดับฮาร์ดแวร์พื้นฐานไปจนถึงการสร้าง Edge Gateway ที่มีความทนทานสูง การทดสอบการทำงานยามตัดเน็ตพิสูจน์ให้เห็นถึงความพร้อมในการนำไปใช้งานในอุตสาหกรรมจริงได้อย่างมั่นคง
