 # คำถามทบทวน

## 1.Multiple Source Files: เหตุใดต้องแยก source code เป็นหลายไฟล์?
```
เพราะช่วยให้โค้ด อ่านง่าย แก้ไขง่าย และจัดการง่าย
แต่ละไฟล์มีหน้าที่เฉพาะ เช่น sensor.c อ่านค่าเซนเซอร์, display.c แสดงผล, led.c ควบคุมไฟ
เวลามีการเปลี่ยนแปลงบางส่วน เช่น ปรับการแสดงผล ก็ไม่กระทบส่วนอื่น
ช่วยให้ทำงานเป็นทีมได้ดีขึ้น และลดปัญหา compile ซ้ำทั้งโปรเจกต์
```
## 2.CMakeLists.txt Management: การเพิ่มไฟล์ source ใหม่ต้องแก้ไขอะไรบ้าง?
```
เมื่อเพิ่ม .c ใหม่ (เช่น display.c, led.c) ต้องเพิ่มชื่อไฟล์ในบรรทัด SRCS ของ CMakeLists.txt
idf_component_register(SRCS "main.c" "sensor.c" "display.c" "led.c"
                      INCLUDE_DIRS ".")
 เพื่อให้ระบบ build รู้ว่าต้อง compile ไฟล์ใหม่เหล่านั้นรวมใน binary
```
## 3.Header Files: บทบาทของไฟล์ .h คืออะไร และทำไมต้องมี?
```
ไฟล์ .h คือ “ประกาศ” ส่วนติดต่อของแต่ละโมดูล
ประกาศ function prototype, struct, และ constant ที่ไฟล์อื่นต้องใช้
เช่น sensor.h ประกาศ float read_sensor(); เพื่อให้ main.c หรือ display.c เรียกได้
ถ้าไม่มี .h โปรแกรมจะไม่รู้จักฟังก์ชันจากไฟล์อื่น → compile error
```
## 4.Include Directories: เหตุใด CMakeLists.txt ต้องระบุ INCLUDE_DIRS?
```
เพราะต้องบอกให้ CMake รู้ว่าจะหาไฟล์ .h จากที่ไหน
idf_component_register(SRCS "main.c" "sensor.c"
                      INCLUDE_DIRS ".")
ถ้ามีหลายโฟลเดอร์ เช่น components/sensor/include ต้องเพิ่ม path ตรงนี้ด้วย
 ถ้าไม่ระบุ INCLUDE_DIRS → #include "sensor.h" จะหาไฟล์ไม่เจอ
 ```
## 5.Git Ignore: ไฟล์ .gitignore ช่วยอะไรในการจัดการ ESP32 project?
```
.gitignore ป้องกันไม่ให้ Git เก็บไฟล์ที่ไม่จำเป็น เช่น
โฟลเดอร์ build/ (binary, temporary)
ไฟล์ .pyc, .env, หรือ sdkconfig.old
👉 ทำให้ repository สะอาด และลดขนาดโปรเจกต์ที่ push ขึ้น GitHub
ตัวอย่าง:
build/
*.bin
*.pyc
sdkconfig.old
```
## 6.Task Management: การใช้ FreeRTOS task ในโมดูล LED ช่วยอะไร?
```
ช่วยให้ไฟ LED ทำงาน อิสระจาก main loop
เช่น LED กระพริบทุก 500 ms โดยไม่บล็อกงานอื่น
ใช้ xTaskCreate() เพื่อสร้าง task แยกสำหรับ LED
ทำให้โปรแกรมทำหลายอย่างพร้อมกัน (multitasking) เช่น อ่าน sensor + แสดงผล + กระพริบไฟ
```
## 7.Code Organization: ข้อดีของการแยกโมดูล sensor, display, led เป็นไฟล์แยกคืออะไร?
```
เพิ่ม ความชัดเจนของโครงสร้างโค้ด
ลดการซ้ำซ้อน (reuse ได้ในโปรเจกต์อื่น)
ง่ายต่อการ debug และ maintenance
สามารถพัฒนา/ทดสอบแต่ละส่วนแยกได้ เช่น ทดสอบ sensor.c โดยไม่ต้องรันทั้งหมด
```

# บันทึกผลการทดลอง

```
ขั้นตอนที่ 1 (เฉพาะ sensor.c):

จำนวนไฟล์ source: 2
ขนาด binary: 131072 bytes
การทำงาน: โปรแกรมจำลองอ่านข้อมูลจาก sensor โดยแสดงอุณหภูมิ และ ความชื้อบน console พร้อมแสดงสถานะ sensor ทุกๆ 3 รอบ
ขั้นตอนที่ 2 (เพิ่ม display.c):

จำนวนไฟล์ source: 3
ขนาด binary: 166320 bytes
การทำงาน: เพิ่มการแสดงผลบนจอจำลองด้วย display_init(), display_show_message() และ display_show_data() เพื่อเเสดงข้อความเเละค่าที่อ่านได้จาก sensor
ขั้นตอนที่ 3 (เพิ่ม led.c):

จำนวนไฟล์ source: 4
ขนาด binary: 181,000 bytes
การทำงาน: เพิ่มการทำงาน LED run ให้ LED กระพริบทุกๆ 3 วินาที เเละแสดงสถานะ LED off หรือ on บน display ทุก loop

การเพิ่มไฟล์ source ส่งผลต่อขนาด binary อย่างไร?
Ans: เพิ่มไฟล์ .c ใหม่จะทำให้ binary ใหญ่ขึ้นตามโค้ดและข้อมูลที่เพิ่มเข้าไป
LED กะพริบทุกกี่วินาที 3 วินาที
แต่ละโมดูลแสดงข้อมูล file และ line อย่างไร?
Ans: ใช้ __FILE__ และ __LINE__ ใน Log เพื่อบอกชื่อไฟล์และบรรทัดที่โค้ดรัน
```

