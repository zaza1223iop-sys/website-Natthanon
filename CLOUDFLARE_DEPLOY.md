# คู่มือการติดตั้งและเชื่อมต่อ Cloudflare Pages & Cloudflare D1 Database
> สำหรับโปรเจกต์ **My Study Planner – ระบบจัดการการบ้านและรายวิชา**

ระบบนี้ได้รับการออกแบบและพัฒนาโค้ดโครงสร้าง Backend (Cloudflare Pages Functions) และ Database Schema สำหรับ **Cloudflare D1 (Serverless SQL Database)** ไว้อย่างสมบูรณ์เรียบร้อยแล้ว โดยมีสถาปัตยกรรมดังนี้:

---

## 🏗️ โครงสร้างสถาปัตยกรรม (Architecture Overview)

```
[ Frontend: Web Browser ]
        │  (HTML5 + Tailwind CSS + Lucide + Vanilla JS)
        ▼
[ Cloudflare Pages (Global Edge Network) ]
        │
   ┌────┴─────────────────────────────┐
   ▼                                  ▼
[ Static Assets ]          [ Pages Functions: /api/* ]
 (index.html, css, js)       ├── /api/status.js     (ตรวจสอบสถานะ D1)
                             ├── /api/init.js       (สร้างตารางอัตโนมัติ)
                             ├── /api/courses.js    (CRUD รายวิชา)
                             ├── /api/homework.js   (CRUD การบ้าน & สถานะ)
                             ├── /api/timetable.js  (CRUD ตารางเรียน)
                             └── /api/settings.js   (จัดการการตั้งค่า)
                                      │
                                      ▼
                      [ Cloudflare D1 SQL Database ]
                        (courses, homework, timetable, settings)
```

---

## 🚀 วิธีการขึ้นโฮสติ้งและเชื่อมต่อ Database บน Cloudflare

คุณสามารถเลือกได้ **2 วิธี** ตามความสะดวก:

---

### วิธีที่ 1: ติดตั้งผ่าน Cloudflare Web Dashboard (แนะนำสำหรับผู้เริ่มต้น - ไม่ต้องใช้ CLI)

#### ขั้นตอนที่ 1: สร้างฐานข้อมูล Cloudflare D1
1. ล็อกอินเข้าสู่ [Cloudflare Dashboard](https://dash.cloudflare.com)
2. ที่เมนูด้านซ้าย เลือก **Storage & Databases** &rarr; **D1 SQL Database**
3. คลิกปุ่ม **Create database**
4. ตั้งชื่อฐานข้อมูล เช่น: `study-planner-db` แล้วคลิก **Create**
5. เมื่อสร้างเสร็จ คลิกเข้าไปที่ฐานข้อมูล `study-planner-db` แล้วไปที่แท็บ **Console**
6. เปิดไฟล์ [schema.sql](file:///d:/My%20Study%20Planner/schema.sql) ในโฟลเดอร์ คัดลอกโค้ด SQL ทั้งหมดไปวางในช่อง Console แล้วคลิก **Execute** เพื่อสร้างตารางทั้งหมด

#### ขั้นตอนที่ 2: นำโปรเจกต์ขึ้น Cloudflare Pages (ผ่าน GitHub หรือ Direct Upload)
**ทางเลือก A: เชื่อมต่อผ่าน GitHub (แนะนำที่สุด):**
1. อัปโหลดโฟลเดอร์ `d:\My Study Planner` ขึ้น GitHub Repository ของคุณ
2. ใน Cloudflare Dashboard ไปที่ **Compute (Workers & Pages)** &rarr; **Create application** &rarr; เลือกแท็บ **Pages**
3. คลิก **Connect to Git** และเลือก Repository ของคุณ
4. ตั้งค่า Build Settings:
   - **Framework preset**: `None`
   - **Build command**: *(ปล่อยว่างไว้)*
   - **Build output directory**: `.` *(จุดเดียว หรือ ปล่อยว่าง)*
5. คลิก **Save and Deploy**

**ทางเลือก B: อัปโหลดโฟลเดอร์โดยตรง (Direct Upload):**
1. บีบอัดไฟล์ในโฟลเดอร์ `d:\My Study Planner` เป็นไฟล์ `.zip` (หรือลากวางทั้งโฟลเดอร์)
2. ใน Cloudflare Dashboard ไปที่ **Workers & Pages** &rarr; **Create application** &rarr; **Pages** &rarr; **Upload assets**
3. ตั้งชื่อโปรเจกต์ เช่น `my-study-planner` แล้วลากไฟล์ขึ้นไป

#### ขั้นตอนที่ 3: ผูก Database Binding (เชื่อมต่อ Cloudflare D1 เข้ากับ Pages)
1. ในหน้าโปรเจกต์ Pages ของคุณ ไปที่แท็บ **Settings** &rarr; **Functions**
2. เลื่อนลงมาที่หัวข้อ **D1 database bindings** แล้วคลิก **Add binding**
3. กรอกข้อมูลดังนี้:
   - **Variable name**: พิมพ์คำว่า `DB` *(ตัวพิมพ์ใหญ่)*
   - **D1 database**: เลือก `study-planner-db`
4. คลิก **Save**
5. ไปที่แท็บ **Deployments** แล้วคลิก **Retry deployment** หรือสร้าง Deploy ใหม่ เพื่อให้ Function เชื่อมกับ D1 ทันที

---

### วิธีที่ 2: Deploy ด้วย Wrangler CLI (ผ่าน Command Line)

หากเครื่องของคุณมี Node.js และต้องการ Deploy ผ่านคำสั่งอย่างรวดเร็ว:

1. **ติดตั้ง Wrangler และล็อกอิน:**
   ```bash
   npx wrangler login
   ```
2. **สร้างฐานข้อมูล D1 บน Cloudflare:**
   ```bash
   npx wrangler d1 create study-planner-db
   ```
   *(นำ `database_id` ที่ได้จากข้อความผลลัพธ์ ไปใส่ในไฟล์ [wrangler.toml](file:///d:/My%20Study%20Planner/wrangler.toml))*

3. **รัน Schema สร้างตารางบน Cloudflare D1:**
   ```bash
   npx wrangler d1 execute study-planner-db --file=./schema.sql --remote
   ```

4. **Deploy ขึ้น Cloudflare Pages:**
   ```bash
   npx wrangler pages deploy . --project-name=my-study-planner
   ```

เมื่อเสร็จสิ้น Cloudflare จะแจ้ง URL ของเว็บไซต์คุณ เช่น:
`https://my-study-planner.pages.dev` สามารถเข้าใช้งานได้จากทั่วโลกทันที!

---

## ⚡ คุณสมบัติพิเศษที่ระบบเตรียมไว้ให้พร้อมแล้ว

1. **Auto-Migration & Seeder (`/api/init`)**:
   - เมื่อเว็บแอปเปิดขึ้นมาบน Cloudflare หากฐานข้อมูลยังว่างเปล่า ระบบจะเรียก `/api/init` อัตโนมัติเพื่อตรวจสอบความสมบูรณ์ของตาราง
2. **Dual-Mode Sync (ออนไลน์ / ออฟไลน์)**:
   - **เมื่อเชื่อมต่อ Cloudflare D1 สำเร็จ**: แถบด้านบนจะแสดงไอคอนสีเขียว `☁️ Cloudflare D1: เชื่อมต่อแล้ว` และบันทึกข้อมูลขึ้นก้อน Cloudflare D1 ทันที
   - **เมื่อเปิดออฟไลน์ หรือเปิดผ่าน Local**: ระบบจะสลับไปใช้ `💾 ข้อมูลในเครื่อง (LocalStorage)` โดยอัตโนมัติ ทำให้เว็บแอปไม่ล่มและใช้งานได้ตลอดเวลา
3. **ปุ่มซิงค์ข้อมูลบนหน้าตั้งค่า**:
   - สามารถกดปุ่ม *"ซิงค์ข้อมูล Local ขึ้น Cloudflare D1"* หรือ *"ดึงข้อมูลล่าสุดจาก Cloudflare D1"* ได้ทุกเมื่อ
