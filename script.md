* **Setup Cloudflared Tunnel แบบถาวร**
* **ติดตั้งและรัน n8n ด้วย PM2**
* **ตั้งค่า systemd service สำหรับ cloudflared**

---

# สคริปต์แบบ All-in-One (Ubuntu/Debian)

```bash
#!/bin/bash

# --- อัปเดตระบบและติดตั้ง Node.js (ถ้ายังไม่ติดตั้ง) ---
echo ">> Updating system and installing Node.js..."
sudo apt update && sudo apt upgrade -y
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt install -y nodejs npm

# --- ติดตั้ง PM2 แบบ global ---
echo ">> Installing PM2 globally..."
sudo npm install -g pm2

# --- ติดตั้ง n8n ---
echo ">> Installing n8n globally..."
sudo npm install -g n8n

# --- เริ่มรัน n8n ด้วย PM2 ---
echo ">> Starting n8n with PM2..."
pm2 start n8n --name "n8n"
pm2 save

# --- ติดตั้ง cloudflared ---
echo ">> Installing cloudflared..."
wget -q https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb -O cloudflared.deb
sudo dpkg -i cloudflared.deb
rm cloudflared.deb

# --- login cloudflared (แนะนำให้รันแยก เพราะต้องเปิด Browser) ---
echo ">> Please login cloudflared (this will open your browser)..."
cloudflared tunnel login
echo ">> Logged in successfully."

# --- สร้าง tunnel ใหม่ชื่อ n8n-tunnel ---
echo ">> Creating cloudflared tunnel named n8n-tunnel..."
TUNNEL_ID=$(cloudflared tunnel create n8n-tunnel | grep 'Created tunnel' | awk '{print $3}')
echo "Tunnel ID: $TUNNEL_ID"

# --- สร้างไฟล์ config ของ cloudflared ---
echo ">> Writing cloudflared config file..."
mkdir -p ~/.cloudflared
cat > ~/.cloudflared/config.yml << EOF
tunnel: $TUNNEL_ID
credentials-file: ~/.cloudflared/$TUNNEL_ID.json

ingress:
  - hostname: n8n.yourdomain.com
    service: http://localhost:5678
  - service: http_status:404
EOF

# --- ติดตั้ง cloudflared service สำหรับ systemd ---
echo ">> Installing cloudflared as a systemd service..."
sudo cloudflared service install

# --- เริ่มและเปิด service cloudflared ---
echo ">> Enabling and starting cloudflared service..."
sudo systemctl enable cloudflared
sudo systemctl start cloudflared

# --- แสดงสถานะ service ---
sudo systemctl status cloudflared --no-pager

echo ">> Setup complete!"
echo ">>> อย่าลืมตั้ง DNS CNAME ใน Cloudflare ให้"
echo ">>> n8n.yourdomain.com ชี้ไปที่ $TUNNEL_ID.cfargotunnel.com"
echo ">>> และเปิด Proxy (สีส้ม) ด้วยนะ!"

echo ">> นอกจากนี้ PM2 รัน n8n ให้อัตโนมัติบนพอร์ต 5678 แล้ว"
echo ">> เปิดเบราว์เซอร์ไปที่ https://n8n.yourdomain.com เพื่อเข้าใช้งานได้เลย!"
```

---

# คำแนะนำหลังรันสคริปต์

1. เปลี่ยน `n8n.yourdomain.com` เป็นโดเมนจริงของแทมแทม ในไฟล์ config.yml ก่อนรันจริง
   หรือแก้ทีหลังใน `~/.cloudflared/config.yml`

2. สคริปต์นี้จะเปิด browser เพื่อ login Cloudflare ตอน `cloudflared tunnel login`
   ต้องล็อกอินด้วย account Cloudflare ที่มีโดเมนนั้นผูกไว้

3. อย่าลืมตั้ง DNS แบบ CNAME ที่ Cloudflare Dashboard
   ชี้โดเมนของแทมแทมไปที่ `TUNNEL_ID.cfargotunnel.com` และเปิด Proxy (สีส้ม)

4. ถ้าอยากรัน n8n แบบกำหนด environment variables เช่น `N8N_HOST` หรือ `N8N_PORT`
   ให้แก้คำสั่ง `pm2 start n8n` เป็นแบบนี้ เช่น

   ```bash
   pm2 start n8n --name "n8n" --env N8N_HOST=n8n.yourdomain.com --env N8N_PORT=5678
   ```

---

# Bonus: คำสั่ง ngrok แบบรวบรัด

ถ้าอยากลอง ngrok แบบเร็ว ๆ ก่อน

```bash
# โหลด ngrok และแตกไฟล์แล้ว
./ngrok authtoken <your_authtoken>
./ngrok http 5678
```
