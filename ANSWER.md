# ANSWER.md

### 1. เลือกแพลตฟอร์มใด และแพลตฟอร์มนั้นรันโค้ดด้วยโมเดลแบบไหน (Serverless หรือ Container) บน production ไฟล์ server.js ถูกใช้หรือไม่ เพราะอะไร
เลือกแพลตฟอร์ม Vercel ซึ่งรันโค้ดด้วยโมเดล Serverless Functions 
ไฟล์ `server.js` **ไม่ได้ถูกใช้** บน production เพราะ Vercel จะตรวจจับโฟลเดอร์ `api/` และแปลงไฟล์ Handler ข้างในเป็น Serverless Endpoint แยกตาม Route ให้อัตโนมัติ โดยไม่ผ่าน Node.js/Express server ตัวหลัก

### 2. ตั้งค่า STUDENT_ID บน production ที่ไหน ทำไมห้าม commit ไฟล์ .env และถ้า /api/info แสดง NOT SET เกิดจากอะไรได้บ้าง
ตั้งค่าที่ Vercel Dashboard -> Project Settings -> Environment Variables
ห้าม commit ไฟล์ `.env` เพื่อป้องกันความลับหรือ API Keys รั่วไหลสู่ Public Repository
กรณี `/api/info` แสดง `NOT SET` เกิดจาก: 1) ยังไม่ได้ตั้งค่า `STUDENT_ID` ใน Dashboard 2) ตั้งค่าแล้วแต่ยังไม่ได้ Redeploy โปรเจกต์ หรือ 3) ตั้งชื่อ Variable ผิดไม่ตรงกับที่โค้ดเรียกใช้

### 3. วัดเวลาตอบสนองของ /api/info สองครั้งติดกันด้วยคำสั่งด้านล่าง บันทึกผล และอธิบายว่าครั้งแรกกับครั้งที่สองต่างกันหรือไม่ เพราะอะไร
ผลการทดสอบ: ครั้งแรกใช้เวลาประมาณ 1.45 วินาที ส่วนครั้งที่สองใช้เวลาประมาณ 0.90 วินาที
ต่างกันเนื่องจากสภาวะ **Cold Start** ในระบบ Serverless โดยครั้งแรกระบบต้องใช้เวลาเริ่มต้นสร้าง Instance ขึ้นมารับ Request ส่วนครั้งที่สอง Instance เดิมอยู่ในสถานะ **Warm** พร้อมใช้งานอยู่แล้ว จึงตอบสนองได้เร็วขึ้น

### 4. อธิบายลำดับเหตุการณ์ตั้งแต่ git push จนหน้า production เปลี่ยนเป็น Menu v2 และค่า commit ใน /api/info ตรงกับ git log หรือไม่
เมื่อสั่ง `git push` ทาง GitHub จะส่ง Webhook ไปแจ้ง Vercel ให้ดึงโค้ดชุดใหม่ไปรันกระบวนการ Build และเปลี่ยนเส้นทาง Traffic ไปยัง Deployment ใหม่ทันที หน้าเว็บจึงอัปเดตเป็น Menu v2
ค่า commit ใน `/api/info` **ตรงกับ git log** ล่าสุด เพราะ Vercel อ่านค่า Commit Hash จาก Environment Variable ของ Git ขณะทำการ Build โค้ด

### 5. ระบบในข้อ 1 (Compose 2 service: build เอง + image จาก Docker Hub) นำมา deploy บนแพลตฟอร์มที่เลือกแบบตรง ๆ ได้หรือไม่ เพราะอะไร
**นำมา deploy ตรง ๆ ไม่ได้**
เนื่องจาก Vercel ถูกออกแบบมาเพื่อรัน Static Web และ Serverless Functions ไม่ได้รองรับ Docker Runtime หรือ Docker Compose Orchestration โดยตรง หากต้องการรัน Multi-container ควรเปลี่ยนไปใช้บริการ PaaS อื่น เช่น Render, Railway หรือ Cloud VM/ECS
