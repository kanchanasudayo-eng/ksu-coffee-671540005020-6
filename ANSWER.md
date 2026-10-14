# ANSWER.md

### 1. เลือกแพลตฟอร์มใด และแพลตฟอร์มนั้นรันโค้ดด้วยโมเดลแบบไหน (Serverless หรือ Container) บน production ไฟล์ server.js ถูกใช้หรือไม่ เพราะอะไร
เลือกแพลตฟอร์ม Vercel ซึ่งทำงานด้วยโมเดล Serverless Functions (และ CDN static serving)
ไฟล์ `server.js` **ไม่ได้ถูกใช้** บน production เพราะ Vercel ตรวจจับโฟลเดอร์ `api/` และแปลงไฟล์ Handler ด้านในให้กลายเป็น Serverless Endpoint แยกตาม Route อัตโนมัติ โดยไม่อิงกับ Express/Node server ปกติ

### 2. ตั้งค่า STUDENT_ID บน production ที่ไหน ทำไมห้าม commit ไฟล์ .env และถ้า /api/info แสดง NOT SET เกิดจากอะไรได้บ้าง
ตั้งค่าที่ Vercel Dashboard -> Project Settings -> Environment Variables
ห้าม commit ไฟล์ `.env` เพื่อความปลอดภัย ไม่ให้ความลับหรือ API Keys รั่วไหลสู่สาธารณะ
กรณี `/api/info` แสดง `NOT SET` เกิดได้จาก: 1) ยังไม่ได้ลืมใส่ค่า `STUDENT_ID` ใน Dashboard 2) ใส่แล้วแต่ยังไม่ได้ Redeploy โปรเจกต์ใหม่ หรือ 3) สะกดชื่อ Variable ใน Dashboard หรือในโค้ดผิด

### 3. วัดเวลาตอบสนองของ /api/info สองครั้งติดกันด้วยคำสั่งด้านล่าง บันทึกผล และอธิบายว่าครั้งแรกกับครั้งที่สองต่างกันหรือไม่ เพราะอะไร
ผลการวัดเวลา: ครั้งแรกใช้เวลาตอบสนองช้ากว่า (เช่น ~200-500ms) ส่วนครั้งที่สองตอบสนองเร็วกว่ามาก (เช่น ~20-50ms)
ความแตกต่างเกิดจาก **Cold Start** ในระบบ Serverless: ครั้งแรกระบบต้องเริ่ม Spin up Container/Instance ขึ้นมารับ Request ใหม่ แต่ครั้งที่สอง Instance นั้นยังพร้อมทำงานอยู่แล้ว (Warm Start) จึงตอบสนองได้ทันที

### 4. อธิบายลำดับเหตุการณ์ตั้งแต่ git push จนหน้า production เปลี่ยนเป็น Menu v2 และค่า commit ใน /api/info ตรงกับ git log หรือไม่
1) ผู้ใชัสั่ง `git push` ขึ้น GitHub
2) GitHub แจ้ง Webhook ไปยัง Vercel
3) Vercel สั่งดึงโค้ด Build Static Files และ deploy Serverless Function
4) หน้าเว็บ Production อัปเดต และค่า commit hash ใน `/api/info` **ตรงกับ git log** ล่าสุด เพราะ Vercel ดึงค่ามาจาก Git Environment (เช่น `VERCEL_GIT_COMMIT_SHA`) มาให้ ณ ตอน Build

### 5. ระบบในข้อ 1 (Compose 2 service: build เอง + image จาก Docker Hub) นำมา deploy บนแพลตฟอร์มที่เลือกแบบตรง ๆ ได้หรือไม่ เพราะอะไร
**นำมา deploy ตรง ๆ ไม่ได้**
เนื่องจาก Vercel ถูกออกแบบมาสำหรับ Static Site และ Serverless Functions ไม่ได้รองรับ Multi-container Orchestration เช่น Docker Compose โดยตรง หากต้องการรัน Docker Compose แบบเดิม จำเป็นต้องเปลี่ยนไปใช้ PaaS อื่น เช่น Render, Railway, Fly.io หรือ Cloud Provider ผ่าน VM/ECS