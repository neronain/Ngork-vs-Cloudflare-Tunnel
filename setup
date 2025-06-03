* **Cloudflared Tunnel แบบตั้งค่าถาวร (Persistent Tunnel)**
* **วิธีใช้ ngrok แบบโปรเจกต์จริงจัง พร้อมทริคเด็ด**

---

# Part 1: Setup Cloudflared Tunnel แบบถาวร ให้มันวิ่ง Auto พร้อมรีบูตเครื่อง

---

### Step 1: สมัครและตั้งค่า Tunnel บน Cloudflare

1. เข้าไปที่ [https://dash.cloudflare.com](https://dash.cloudflare.com) แล้วล็อกอินหรือสมัครบัญชีฟรีก่อน

2. ไปที่เมนู **Zero Trust** (หรือ Cloudflare for Teams)

3. เลือก **Access** > **Tunnels**

4. กด **Create a Tunnel**

5. ตั้งชื่อ Tunnel เช่น `n8n-tunnel`

6. หลังจากกดสร้างแล้ว Cloudflare จะให้คำสั่งดาวน์โหลดไฟล์ config หรือ token สำหรับยืนยันบนเครื่องเรา

---

### Step 2: ติดตั้ง cloudflared บนเซิร์ฟเวอร์

ถ้าเป็น Ubuntu ลองตามนี้:

```bash
curl -LO https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
sudo dpkg -i cloudflared-linux-amd64.deb
```

(ถ้า OS อื่น หรือแมค ให้ดูในลิงก์ดาวน์โหลดตาม OS)

---

### Step 3: Authenticate Tunnel

นำ token ที่ได้จากเว็บมารันบนเครื่อง

```bash
cloudflared tunnel login
```

มันจะเปิดเบราว์เซอร์ให้ล็อกอิน Cloudflare และอนุญาตการเชื่อมต่อ

---

### Step 4: สร้าง Tunnel และตั้งชื่อ

```bash
cloudflared tunnel create n8n-tunnel
```

จะสร้าง tunnel ตัวใหม่พร้อมสร้าง credential file เก็บไว้ในเครื่อง

---

### Step 5: สร้างไฟล์ config ให้ Tunnel รันไปที่ localhost:5678

สร้างไฟล์ config ที่ `/etc/cloudflared/config.yml` หรือ `~/.cloudflared/config.yml` แบบนี้:

```yaml
tunnel: <Tunnel-ID ที่ได้จากคำสั่ง create>
credentials-file: /home/youruser/.cloudflared/<Tunnel-ID>.json

ingress:
  - hostname: n8n.yourdomain.com
    service: http://localhost:5678
  - service: http_status:404
```

**เปลี่ยน** `n8n.yourdomain.com` เป็นโดเมนที่ผูกกับ Cloudflare ของแทมแทม

---

### Step 6: รัน Tunnel แบบ service

สร้าง service systemd

```bash
sudo cloudflared service install
```

มันจะสร้าง service ให้ชื่อ `cloudflared.service` ที่จะทำงานอัตโนมัติหลังรีบูตเครื่อง

---

### Step 7: เปิด service และตรวจสอบสถานะ

```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
sudo systemctl status cloudflared
```

---

### Step 8: ตั้งค่า DNS บน Cloudflare

* ไปที่ Cloudflare Dashboard DNS

* เพิ่ม **CNAME** record

| Name | Target                         | TTL  | Proxy status              |
| ---- | ------------------------------ | ---- | ------------------------- |
| n8n  | `<Tunnel-ID>.cfargotunnel.com` | Auto | **Proxy enabled (สีส้ม)** |

ตัว `<Tunnel-ID>` จะได้จากตอนสร้าง tunnel หรือในหน้า Tunnels ของ Cloudflare

---

### Step 9: ทดสอบเปิดเว็บ

เปิด

```
https://n8n.yourdomain.com
```

จะเจอ n8n ที่รันอยู่บนเครื่อง local แทมแทมแบบปลอดภัย ผ่าน Cloudflare Tunnel ที่ทำเป็น service อัตโนมัติ!

---

# Part 2: ใช้งาน ngrok แบบเจ๋ง ๆ สำหรับ dev หรือโปรเจกต์เล็ก

---

### Step 1: โหลด ngrok

[https://ngrok.com/download](https://ngrok.com/download)

แตกไฟล์แล้วเปิด terminal

---

### Step 2: สมัคร account ngrok (แนะนำ)

สมัคร account ฟรีที่ [https://dashboard.ngrok.com/signup](https://dashboard.ngrok.com/signup)

---

### Step 3: ใส่ authtoken

หลังล็อกอินในเว็บจะได้ `authtoken` มา

รันคำสั่งนี้

```bash
./ngrok authtoken <your_authtoken>
```

---

### Step 4: สร้าง tunnel ไปที่ n8n

```bash
./ngrok http 5678
```

จะได้ URL แบบ

```
Forwarding                    https://abcd-1234-5678.ngrok.io -> http://localhost:5678
```

ใช้ URL นี้ไปแชร์หรือเทสต์กับ client ได้เลย

---

### Step 5: ใช้งานฟีเจอร์ ngrok UI

เปิดเบราว์เซอร์ที่

```
http://127.0.0.1:4040
```

ดู Log และการเชื่อมต่อแบบ Realtime

---

### Step 6: รัน ngrok แบบ Background (Optional)

ถ้าใช้ Linux/Mac อยากรัน ngrok เป็น background process:

```bash
nohup ./ngrok http 5678 > ngrok.log 2>&1 &
```

---

### Step 7: ใช้งาน ngrok config file (Advanced)

สร้างไฟล์ `~/.ngrok2/ngrok.yml` ใส่ config แบบนี้

```yaml
authtoken: <your_authtoken>
tunnels:
  n8n:
    proto: http
    addr: 5678
```

แล้วรันแค่

```bash
./ngrok start n8n
```

---

### ข้อดีของ ngrok แบบนี้

* URL เปลี่ยนเป็น static ถ้าใช้ paid account
* ตั้งค่าง่ายและเร็ว
* เหมาะสำหรับ dev หรือแชร์แบบชั่วคราว

---

# สรุป

| Feature                  | Cloudflared Tunnel (ถาวร)         | ngrok (dev, share ง่าย)          |
| ------------------------ | --------------------------------- | -------------------------------- |
| URL แบบ Custom Domain    | ใช่ (ต้องตั้ง DNS กับ Cloudflare) | ได้ถ้าเสียเงินเท่านั้น           |
| ติดตั้งและ Setup Service | ต้องตั้งค่าเยอะหน่อย              | ง่าย รันทีเดียวใช้เลย            |
| ความเสถียรใช้งานจริง     | เหมาะสำหรับ Production            | เหมาะ Dev / Testing              |
| ฟรีใช้งาน                | ฟรีแบบจำกัด แต่พอเพียง            | ฟรีแบบจำกัด มีข้อจำกัด bandwidth |
| Auto Start หลัง reboot   | ใช่ (ใช้ systemd service)         | ไม่ (ต้องรันเอง)                 |

---
