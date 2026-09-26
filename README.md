# Line Chatbot ชุมชนสกลนคร (โค้ดตั้งต้น)

แชทบอทไลน์แบบ keyword-based สำหรับตอบคำถามพื้นฐานของชุมชน เช่น เวลาทำการ รพ.สต.,
เบอร์ฉุกเฉิน, ขั้นตอนขอเอกสารราชการ

## ไฟล์ในโปรเจกต์

- `app.py` — เซิร์ฟเวอร์ Flask หลัก รับ webhook จาก LINE และตอบกลับ
- `requirements.txt` — รายชื่อไลบรารีที่ต้องติดตั้ง
- `Procfile` — ไฟล์บอก Render ว่าจะรันโปรแกรมยังไง

## 1. สร้าง LINE Channel

1. เข้า https://developers.line.biz/console/ แล้วล็อกอินด้วยบัญชี LINE
2. สร้าง Provider (ถ้ายังไม่มี) แล้วสร้าง Channel ชนิด **Messaging API**
3. ไปที่แท็บ **Messaging API** ของ Channel ที่สร้าง แล้วคัดลอก
   - **Channel access token** (กด Issue ถ้ายังไม่มี)
   - **Channel secret** (อยู่ในแท็บ Basic settings)

## 2. ติดตั้งและรันในเครื่องตัวเอง

```bash
# สร้าง virtual environment (แนะนำ)
python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

# ติดตั้งไลบรารี
pip install -r requirements.txt

# ตั้งค่า environment variable
export LINE_CHANNEL_ACCESS_TOKEN="ใส่ token ที่คัดลอกมา"
export LINE_CHANNEL_SECRET="ใส่ secret ที่คัดลอกมา"

# รันเซิร์ฟเวอร์
python app.py
```

เซิร์ฟเวอร์จะรันที่ `http://localhost:5000`

## 3. เปิด webhook ชั่วคราวด้วย ngrok (ทดสอบก่อน deploy จริง)

```bash
ngrok http 5000
```

จะได้ URL แบบ `https://xxxx.ngrok-free.app` เอา URL นี้ต่อท้ายด้วย `/callback`
เช่น `https://xxxx.ngrok-free.app/callback` ไปใส่ในช่อง **Webhook URL**
ที่หน้า Messaging API ของ Console แล้วกด **Verify** และเปิด **Use webhook**

## 4. ทดลองคุยกับบอท

สแกน QR Code ของ Official Account จากหน้า Console (อยู่ในแท็บ Messaging API)
เพิ่มเป็นเพื่อน แล้วลองพิมพ์:

- `เวลาเปิด`
- `เบอร์ฉุกเฉิน`
- `บัตรประชาชน`
- `เมนู`

## 5. เพิ่ม/แก้ไขคำตอบ

แก้ไขได้ที่ dictionary `KNOWLEDGE_BASE` ใน `app.py` — เพิ่มคู่ `"คำสำคัญ": "คำตอบ"`
ได้เลย เวอร์ชันนี้ยังเก็บข้อมูลในโค้ด เหมาะกับช่วงเริ่มต้น/เดโม ถ้าจะทำเป็น
โปรเจกต์เต็มรูปแบบ ขั้นต่อไปคือย้ายข้อมูลนี้ไปเก็บในฐานข้อมูล (SQLite/PostgreSQL)
พร้อมทำหน้า admin panel ให้แก้ไขข้อมูลได้โดยไม่ต้องแก้โค้ด

## 6. Deploy ขึ้น Render (ใช้งานจริง 24 ชม.)

1. อัพโค้ดทั้งหมดขึ้น GitHub repository
2. เข้า https://render.com สมัคร/ล็อกอิน แล้วเลือก **New > Web Service**
3. เชื่อมกับ GitHub repo ที่อัพไว้
4. ตั้งค่า:
   - **Build Command**: `pip install -r requirements.txt`
   - **Start Command**: `gunicorn app:app`
5. ไปที่แท็บ **Environment** เพิ่ม environment variable สองตัวเดิม
   (`LINE_CHANNEL_ACCESS_TOKEN`, `LINE_CHANNEL_SECRET`)
6. กด Deploy แล้วรอจน URL พร้อมใช้งาน (เช่น `https://your-app.onrender.com`)
7. เอา URL นั้น + `/callback` ไปใส่ใน Webhook URL ที่ LINE Console แทนของ ngrok

## ขั้นต่อไปที่แนะนำ (ตามแผนโครงการ)

- ย้าย `KNOWLEDGE_BASE` ไปเก็บในฐานข้อมูลจริง
- ทำหน้า admin panel เล็กๆ ให้เจ้าหน้าที่แก้ไขข้อมูลเองได้
- เพิ่ม logging เก็บสถิติว่าคนถามอะไรบ่อย เพื่อวัดผล KPI ตามฟอร์มโครงการ
- พิจารณาต่อ NLU/AI API ถ้าอยากให้บอทเข้าใจภาษาธรรมชาติมากขึ้น
