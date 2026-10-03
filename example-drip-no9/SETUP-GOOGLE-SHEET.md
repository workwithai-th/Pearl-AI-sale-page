# คู่มือตั้งค่า — อบเบอร์ 9 Sale Page

ทำครั้งเดียว ประมาณ 10 นาที มี 3 ส่วน: เชื่อม Google Sheet · ใส่ข้อมูลร้าน · ขึ้น Netlify

---

## ส่วนที่ 1 · เชื่อมฟอร์มเข้า Google Sheet (5 นาที)

### 1. สร้างชีต
1. เข้า <https://sheets.new> (ต้องล็อกอิน Google)
2. ตั้งชื่อชีตว่า `Drip No.9 Orders`
3. แถวแรกใส่หัวคอลัมน์ (คัดลอกไปวางได้เลย):

```
เวลา | ชื่อ | เบอร์โทร | ที่อยู่ | แพ็คเกจ | วิธีชำระ | หมายเหตุ | ยอดรวม
```

### 2. วางโค้ด Apps Script
1. ในหน้าชีต ไปที่เมนู **ส่วนขยาย → Apps Script**
2. ลบโค้ดที่มีอยู่ในไฟล์ `Code.gs` แล้ววางโค้ดนี้แทน:

```javascript
function doPost(e) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName('Orders') || ss.getSheets()[0];
  var d = JSON.parse(e.postData.contents);
  sheet.appendRow([
    d.ts || new Date(),
    d.name,
    d.phone,
    d.address,
    d.package,
    d.payment,
    d.note,
    d.total
  ]);
  return ContentService
    .createTextOutput(JSON.stringify({ ok: true }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. กด **บันทึก** (ไอคอนแผ่นจาน) — ตั้งชื่อโปรเจกต์ว่า `drip-no9-orders` ก็ได้

### 3. Deploy เป็น Web app
1. กดปุ่ม **Deploy → New deployment**
2. เลือก **การกำหนดค่าเพิ่มเติม (⚙)** → เลือกประเภท **เว็บแอป**
3. ตั้งค่า:
   - **Execute as:** Me
   - **Who has access:** Anyone
4. กด **Deploy** → ถ้าเด้งขออนุญาต กด **Authorize** → เลือกบัญชี → Allow
5. **คัดลอก URL** ที่ได้ (ลงท้ายด้วย `/exec`)

### 4. เสียบเข้าหน้าเว็บ
1. เปิดไฟล์ `index.html` ด้วยโปรแกรมแก้ข้อความ (Notepad ก็ได้)
2. หาบรรทัด (อยู่ในส่วน `<script>` ท้ายไฟล์):

```javascript
var CONFIG = {
  SCRIPT_URL: "",
  BANK: { name: "", number: "" }
};
```

3. วาง URL ที่คัดลอกมาใส่ระหว่างเครื่องหมายคำพูด เช่น:

```javascript
var CONFIG = {
  SCRIPT_URL: "https://script.google.com/macros/s/AKfy.../exec",
  BANK: { name: "ธนาคารกสิกรไทย", number: "123-4-56789-0" }
};
```

4. **ทดสอบ:** เปิด `index.html` ใน browser → กรอกฟอร์ม → กดยืนยัน → กลับไปดูชีต
   ว่าข้อมูลตกในแถวใหม่ = สำเร็จ (Success Criteria ข้อ 3 ✅)

> หมายเหตุ: ถ้าแก้โค้ดใน Apps Script ทีหลัง ต้อง **Deploy → Manage deployments → Edit (✏) → Version: New version → Deploy** ทุกครั้ง ไม่งั้น URL เดิมยังใช้โค้ดเก่าอยู่

---

## ส่วนที่ 2 · ใส่ข้อมูลร้านของคุณ (2 นาที)

ยังเว้นไว้ 3 จุดใน `index.html` — ใส่เมื่อพร้อม ไม่กระทบการทำงานอื่น:

| จุด | หาบรรทัด | ใส่อะไร |
|---|---|---|
| บัญชีโอนเงิน | `BANK: { name: "", number: "" }` | ชื่อธนาคาร + เลขบัญชี (แสดงหลังกดสั่งซื้อ) |
| ช่องทางติดต่อ | footer `สั่งซื้อ/สอบถาม: ทักแชทเพจ...` | ลิงก์ LINE / เพจ Facebook / เบอร์ร้าน |
| ความหมาย "เบอร์ 9" | ส่วนจุดขาย / คำโฆษณา | เพิ่มที่มาชื่อหรือวัตถุดิบเมื่อมีข้อมูล |

---

## ส่วนที่ 3 · ขึ้น Netlify ให้ได้ลิงก์ส่งแชท (3 นาที)

1. เข้า <https://app.netlify.com/signup> สมัครฟรี (ด้วย Google ก็ได้)
2. ไปที่ <https://app.netlify.com/drop>
3. **ลากไฟล์ `index.html`** จากโฟลเดอร์นี้ไปวางตรงหน้าเพจ
4. ได้ลิงก์ทันที รูปแบบ `https://ชื่อสุ่ม.netlify.app` — กด **Site settings → Change site name** เปลี่ยนเป็นชื่อที่จำง่าย เช่น `dripno9.netlify.app`
5. ทดสอบเปิดจาก**มือถือ** แล้วส่งลิงก์ให้ตัวเองใน LINE ดู preview

**อัปเดตหน้าเว็บทีหลัง:** แก้ไฟล์ในเครื่อง → ลากไฟล์ใหม่มาวางที่หน้าเดิมใน Netlify → เว็บอัปเดตทันที (ลิงก์เดิม)

**ภาพ preview ตอนแชร์ (OG image):** ตอนนี้ LINE จะโชว์ชื่อหน้า+คำโฆษณา (ไม่มีภาพ)
เมื่อมีรูปสินค้าจริงแล้ว: บันทึกเป็น `og-image.jpg` (1200×630) อัปโหลดไปพร้อมไฟล์ใน Netlify
แล้วเติมบรรทัดนี้ใน `<head>`:

```html
<meta property="og:image" content="https://dripno9.netlify.app/og-image.jpg">
```

---

## เช็กลิสต์ตรวจรับ (ตาม Working Brief)

- [ ] 1. เปิดลิงก์จากมือถือได้ ดูดีบนจอเล็ก
- [ ] 2. ข้อมูลครบ: สินค้า · 350 · 2 แพ็ค 650 ส่งฟรี · วิธีชง
- [ ] 3. กรอกฟอร์มทดสอบ → ข้อมูลตก Google Sheet จริง
- [ ] 4. กดปุ่ม/ลิงก์ทุกจุดทำงาน ไม่มีทางตัน
- [ ] 5. แชร์ลิงก์ใน LINE มี preview ชวนคลิก
- [ ] 6. โหลดหน้าเสร็จเร็ว (ไฟล์เดียว ไม่มีสคริปต์หนัก — โหลดเฉพาะฟอนต์จาก Google)
