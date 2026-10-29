# PSD eNGINNo | SMART GENERATOR

ระบบบริหารจัดการเครื่องกำเนิดไฟฟ้าสำรองฉุกเฉิน (Emergency Generator) สำหรับหลายอาคารในเครือ
(BITEC I, BITEC II, BTB, BTE, UBC II, Summer LaSalle [SL-P1 + SL-Campus A], S23)

## สถาปัตยกรรมระบบ

- **Backend:** Google Apps Script ผูกกับ Google Sheet เดียว
  Spreadsheet ID: `1rWIBahPOhljKkhZHvhhZIhGx8faqtHaxMNS-dcMkfM4`
  ไฟล์หลัก: `Code.gs`, `SheetUtils.gs`, `Auth.gs`, `Api_Data.gs`, `Api_DailyCheck.gs`,
  `Api_Export.gs`, `Api_TestReport.gs`, `Api_AuditExport.gs`, `Api_FuelReport.gs`
- **Frontend:** `index.html` — single-page HTML/CSS/JS ไฟล์เดียว เรียก backend ผ่าน `callApi(action, data)` (fetch ไปยัง Apps Script Web App URL)
- **Hosting:** ตอนนี้ไฟล์ `index.html` ที่ใช้งานจริงถูกอัปโหลดแยกไปที่ **GitHub Pages** (ไม่ได้เสิร์ฟผ่านไฟล์ "index" ใน Apps Script Editor โดยตรง)
  ดังนั้นทุกครั้งที่แก้ไข `index.html` ต้องอัปโหลด/commit ไฟล์ใหม่เข้า repo GitHub ด้วย ไม่ใช่แค่วางใน Apps Script Editor

## วิธี Deploy

### Backend (Apps Script)
1. เปิด Google Sheet → Extensions → Apps Script
2. วางไฟล์ `.gs` ทับไฟล์เดิมตามชื่อ (ถ้ามีการแก้ backend)
3. Deploy → Manage deployments → แก้ไข (ไอคอนดินสอ) → Version: **New version** → Deploy

### Frontend (GitHub Pages)
1. เข้า repo GitHub ของโปรเจกต์ → เปิดไฟล์ `index.html`
2. กดไอคอนดินสอ (Edit this file)
3. เลือกเนื้อหาทั้งหมด (Ctrl+A) ลบทิ้ง แล้ววางเนื้อหาไฟล์ `index.html` ใหม่ลงไป
4. ใส่ commit message แล้วกด Commit changes (Commit directly to the main branch)
5. รอ GitHub Pages build เสร็จ (~30-60 วินาที) แล้ว Hard refresh หน้าเว็บ (Ctrl+Shift+R) เพื่อเคลียร์ cache ก่อนเช็กผล

---

## 🆕 Changelog ล่าสุด (2026-10-29)

### 1. Dashboard — สีข้อความ L/% ตามเกณฑ์คงเหลือ
กราฟเปรียบเทียบความจุถัง (Storage Tank / Day Tank) หน้า Dashboard รวม:
ตัวเลข `L (%)` ใต้แต่ละแท่ง และยอด "Remaining" ของแต่ละกลุ่ม จะเปลี่ยนสีตามเกณฑ์
- 🔵 สีฟ้า ≥ 90%
- 🟢 สีเขียว 50–89%
- 🟡 สีเหลือง 40–49%
- 🔴 สีแดง < 40%

*(ฟังก์ชัน `capPctColor_()` ใน index.html)*

### 2–4. หน้า "บันทึกเชื้อเพลิง" (Storage Fuel Control) — Update ในแอปได้ทันที
ปรับให้สามารถอัปเดตปริมาณเติม/ใช้น้ำมันผ่านแอปได้โดยตรง แทนการไปกรอกใน Google Sheet เอง:

- เพิ่มปุ่ม **"✏️ Update"** ใต้ทุกการ์ดถัง (Store Tank และ Day Tank ประจำเครื่อง) — กดแล้วระบบเลือกเครื่องนั้นในฟอร์มให้อัตโนมัติ และเลื่อนไปที่ฟอร์มพร้อมไฮไลต์
- ฟอร์มถูกจัดเป็น 2 คอลัมน์:
  - ซ้าย **"ข้อมูลเครื่อง"**: Company / Building / Code (ดึงอัตโนมัติ)
  - ขวา **"กรอกข้อมูลการใช้งาน"**: Date, Time (auto ปัจจุบัน แก้ไขได้), Purpose of Use, Full/80%/50% Storage (ดึงอัตโนมัติ), Fuel Level Add, Fuel Level Before (ดึงอัตโนมัติ), Fuel Level After, Fuel Used, Operator, Approved By, Remarks
- **Purpose of Use** เปลี่ยนเป็น dropdown: `Emergency Use` / `Weekly Test Run` / `Refuel Only` / `Other (ระบุเหตุผลอื่น)` — เลือก Other แล้วพิมพ์เหตุผลเองได้ ข้อความจะถูกบันทึกแทนคำว่า "Other"
- ช่อง **Fuel Level Add / After / Used** จะเทาและกรอกไม่ได้ จนกว่าจะเลือก Purpose of Use ก่อน:
  - เลือก Emergency Use / Weekly Test Run → เปิดช่อง After + Used
  - เลือก Refuel Only / Other → เปิดช่อง Add + After (และคำนวณ After = Before + Add ให้อัตโนมัติเมื่อกรอก Add)
- หลังกด "✓ บันทึก" ข้อมูลจะถูกบันทึกเข้าชีต **Storge fuel control** และระบบ sync ยอด Fuel Level After กลับไปที่คอลัมน์ **J (Actual remain(Litre))** ในชีต **Re-Order Point_Fuel** ให้อัตโนมัติ (ใช้ฟังก์ชัน `syncFuelTankActual_()` เดิมของระบบ)

### 5. หน้า Settings — เพิ่มธีมพื้นหลัง Light
เพิ่มการ์ด **"🌗 ธีมพื้นหลังโปรแกรม"** แยกจากสีหลัก (Theme สี) เดิม
เลือกได้ Dark (เดิม) / Light (ขาว) โดยฟอนต์/ตัวอักษรไม่เปลี่ยน

---

## หมายเหตุที่รอการตัดสินใจร่วมกับทีม (ยังไม่ได้แก้)
- ฟอร์ม Weekly Test / Emergency Test ผู้ใช้งานยังสับสนและกรอกข้อมูลไม่ถูกต้อง — รอคุยกับทีมก่อนปรับแก้เพิ่มเติม
