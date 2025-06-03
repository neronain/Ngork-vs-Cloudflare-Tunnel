1. **Cloudflared Tunnel แบบไม่ต้องติดตั้งจริง ๆ** (ก็คือใช้แบบ quick-run แบบ portable)
2. **การใช้งาน ngrok** (ตัวเลือกเบื้องต้นอีกตัวที่คนชอบใช้)

---

## 1. Setup Cloudflared Tunnel แบบไม่ต้องติดตั้ง (Quick Run)

ถ้าแทมแทมอยากใช้ **Cloudflared** แบบไม่ต้องลงติดตั้งบนเครื่องจริง ๆ แค่โหลดไฟล์ binary มาแล้วรันเลย (เหมาะกับ Windows/Mac/Linux)

---

### ขั้นตอนแบบเร็ว

* เข้าไปโหลด [Cloudflared Binary](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation)

* เลือกไฟล์ที่เหมาะกับ OS ของแทมแทม

* แตกไฟล์ออก (ถ้าเป็น zip)

* เปิด Terminal (หรือ CMD) แล้วไปรันคำสั่งนี้ตรง ๆ

```bash
./cloudflared tunnel --url http://localhost:5678
```

(Windows อาจต้องพิมพ์ `cloudflared.exe` แทน)

---

### ข้อดี

* ไม่ต้องติดตั้งจริงจัง หรือ admin rights ก็รันได้เลย
* เดี๋ยวจะได้ลิงก์แบบ [https://random-id.trycloudflare.com](https://random-id.trycloudflare.com) มาให้เปิดใช้ได้ทันที
* ปลอดภัยและเร็ว

---

### ข้อจำกัด

* เป็น session ชั่วคราว ถ้าปิด terminal ก็จะตัด tunnel ทันที
* ถ้าอยากให้ใช้งานต่อเนื่องต้องตั้งเป็น service แบบเต็ม หรือสร้าง tunnel configuration ผ่าน Cloudflare Dashboard

---

## 2. การใช้งาน ngrok

อีกตัวที่ฮิตและง่ายมาก คือ **ngrok** ใช้สร้าง tunnel จากเครื่องเราไปยังอินเทอร์เน็ตโดยไม่ต้องเปิดพอร์ต router เหมือนกัน

---

### วิธีใช้ ngrok แบบง่าย ๆ

1. เข้าเว็บ [https://ngrok.com/download](https://ngrok.com/download) แล้วโหลด ngrok ตัวล่าสุดที่เหมาะกับ OS

2. แตกไฟล์และเปิด terminal ใน folder ที่เก็บ ngrok

3. รันคำสั่ง

```bash
./ngrok http 5678
```

(Windows ก็ `ngrok.exe http 5678`)

4. ngrok จะสร้าง URL แบบ [https://xxxxxxxx.ngrok.io](https://xxxxxxxx.ngrok.io) ให้มาใช้เปิดจากภายนอกได้เลย

---

### ข้อดีของ ngrok

* ใช้งานง่าย แค่โหลดมาแล้วรันได้เลย
* มี UI เว็บดู status tunnel ด้วยที่ [http://127.0.0.1:4040](http://127.0.0.1:4040)
* ฟรีสำหรับ tunnel แบบ basic (มีข้อจำกัดบ้าง)

---

### ข้อจำกัดของ ngrok (ฟรี)

* URL เปลี่ยนทุกครั้งที่รัน (ถ้าไม่สมัคร account แบบเสียเงิน)
* Bandwidth และ session จำกัด

---

## สรุป

| Feature           | Cloudflared Quick Run                                                      | ngrok Quick Run                                          |
| ----------------- | -------------------------------------------------------------------------- | -------------------------------------------------------- |
| ไม่ต้องติดตั้ง    | ✔ (แค่โหลด binary มาใช้ได้เลย)                                             | ✔ (โหลดแล้วรันได้ทันที)                                  |
| URL ที่ได้        | [https://random-id.trycloudflare.com](https://random-id.trycloudflare.com) | [https://random-id.ngrok.io](https://random-id.ngrok.io) |
| เปิด port ไม่ต้อง | ✔                                                                          | ✔                                                        |
| เสถียร/ยาวนาน     | น้อย (ถ้ารันแบบ quick run)                                                 | น้อย (ถ้ารันแบบฟรี)                                      |
| การตั้งค่าเพิ่ม   | ต้องทำบน Cloudflare Dashboard                                              | สมัคร account จ่ายเงิน                                   |

---
