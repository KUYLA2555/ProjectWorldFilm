# WORLD FILM — เว็บไซต์ศูนย์ติดตั้งฟิล์มกรองแสง

เว็บไซต์สแตติก (HTML/CSS ล้วน ไม่มีระบบหลังบ้าน) สำหรับ WORLD FILM
ศูนย์ติดตั้งฟิล์มกรองแสงบ้าน อาคาร สำนักงาน คอนโด

## เปิดดูเว็บ

ดับเบิลคลิกไฟล์ **`index.html`** เพื่อเปิดในเบราว์เซอร์ได้เลย (ไม่ต้องติดตั้งอะไร)

## โครงสร้างไฟล์

```
ProjectWorldFilm/
├── index.html              ← หน้าแรกของเว็บ (เริ่มที่ไฟล์นี้)
├── about.html              ← หน้าเกี่ยวกับเรา (About Us)
├── works.html              ← หน้าผลงานติดตั้ง
├── contact.html            ← หน้าติดต่อเรา
├── README.md               ← ไฟล์นี้ (อธิบายโครงสร้าง)
├── CHANGELOG.md            ← บันทึกประวัติการแก้ไขทุกครั้ง
├── CLAUDE.md               ← คู่มือสำหรับ Claude Code (โครงสร้าง/ข้อควรรู้)
│
├── assets/                 ← รูปภาพทั้งหมดที่เว็บใช้งาน
│   ├── logo-dark.png       ← โลโก้ตัวอักษรกรมท่า (ใช้บนเฮดเดอร์พื้นอ่อน)
│   ├── logo-white.png      ← โลโก้ตัวอักษรขาว (ใช้บนฟุตเตอร์พื้นเข้ม)
│   ├── brands/             ← โลโก้แบรนด์ฟิล์ม
│   │   ├── finnix.png
│   │   ├── 3m.png
│   │   ├── regionfilm.png
│   │   └── ultraguard.png
│   ├── models/             ← โลโก้ประจำรุ่นฟิล์ม (ย่อจากไฟล์ต้นฉบับใน source/ ใช้ในการ์ดเลือกรุ่นหน้าแบรนด์ Finnix/3M เท่านั้น)
│   │   ├── finnix-ceramic.png · finnix-titanium.png · finnix-uvguard.png · finnix-extra-clear.png
│   │   └── 3m-ceramate.png · 3m-ultra-clear.png
│   ├── line.png            ← โลโก้ LINE จริง (ใช้แทนไอคอน SVG เดิมในการ์ดติดต่อ LINE ทุกจุด)
│   ├── facebook.png        ← โลโก้ Facebook จริง (ใช้ในการ์ด LINE/Facebook แถว hero หน้าแรกเท่านั้น)
│   └── source/             ← ไฟล์ภาพต้นฉบับ (ไม่ได้ใช้บนเว็บโดยตรง เก็บไว้เผื่ออนาคต · ไม่ขึ้น GitHub)
│
└── products/               ← หน้าแบรนด์และหน้ารุ่นฟิล์มทั้งหมด
    ├── product.css         ← สไตล์ (CSS) ของหน้าในโฟลเดอร์นี้
    │
    ├── brand-finnix.html       ← หน้าแบรนด์ (รวมรุ่นในแบรนด์นั้น)
    ├── brand-3m.html
    ├── brand-regionfilm.html   ← หน้าแบรนด์แบบโชว์สเปกเลย (ไม่มีหน้ารุ่นย่อย)
    ├── brand-ultraguard.html
    │
    ├── model-finnix-ceramic.html      ← หน้ารุ่น (สเปกฟิล์มรุ่นนั้น)
    ├── model-finnix-titanium.html
    ├── model-finnix-uvguard.html
    ├── model-finnix-extra-clear.html
    ├── model-3m-ultra-clear.html
    ├── model-3m-ceramate.html
    ├── model-ultraguard-ceramic.html
    └── model-ultraguard-nano.html
```

## ผังการเชื่อมหน้า

```
index.html (หน้าแรก)
   └─ เมนู "ฟิล์มของเรา" / การ์ดแบรนด์ 4 ช่อง
        ├─ products/brand-finnix.html      → model-finnix-* (Ceramic, Titanium, UV Guard, Extra Clear)
        ├─ products/brand-3m.html          → model-3m-* (Ultra Clear, Ceramate)
        ├─ products/brand-regionfilm.html  → โชว์สเปก Regionfilm ในหน้าเดียว (ไม่มีหน้ารุ่นย่อย)
        └─ products/brand-ultraguard.html  → model-ultraguard-* (Ceramic, Nano)
```

ทุกหน้าใช้ **โค้ด header/footer ชุดเดียวกัน** — แถบเมนูเป็น **แคปซูลลอยสีกรมท่า**
(โลโก้ขาว = ปุ่มกลับหน้าแรก + เมนู dropdown + ปุ่มทองโทร/คัดลอกเบอร์ + ปุ่มลอย LINE/โทร)

## แก้ไขสิ่งที่ใช้บ่อย

- **เบอร์โทร** `095-229-2086` / **LINE** `@worldcenter` / **Facebook** `https://www.facebook.com/Worldsfilm1`
  → ค้นหาข้อความเหล่านี้ในไฟล์ HTML แล้วแก้ได้เลย (ทั้งปุ่มบนแถบเมนู, ส่วนติดต่อ, ปุ่มลอย, footer)
- **เปลี่ยนโลโก้** → แทนไฟล์ใน `assets/logo-dark.png` และ `assets/logo-white.png`
- **เปลี่ยนโลโก้แบรนด์** → แทนไฟล์ใน `assets/brands/`
- **เพิ่ม/แก้รุ่นฟิล์ม** → ก๊อปไฟล์ `model-*.html` สักไฟล์เป็นแม่แบบ แก้ชื่อ/สเปก แล้วเพิ่มการ์ดลิงก์ในหน้า `brand-*.html` ของแบรนด์นั้น

## หมายเหตุสำคัญ — สเปกตัวอย่าง

**ตัวเลขสเปกในหน้ารุ่นทั้งหมด (VLT / ลดความร้อน / รับประกัน) ยังเป็นข้อมูลตัวอย่าง ไม่ใช่ข้อมูลจริง**
แถบเหลืองแจ้งเตือนบนหน้ารุ่นถูกถอดออกแล้ว (2026-07-18) แต่ตัวเลขยังไม่ได้รับการยืนยัน —
ควรตรวจสอบสเปกจริงกับทางร้านก่อนใช้อ้างอิง

**ตัวเลขที่ยังขัดกันอยู่ รอเจ้าของร้านยืนยัน** (2026-07-22)

- **ลดความร้อนสูงสุด** — หน้าแรก/เกี่ยวกับเรา เขียน 85% · หน้าแบรนด์ Finnix เขียน 88% · หน้ารุ่น Finnix Ceramic เขียน 82%
- **"200+ โครงการที่ส่งมอบ"** ในหน้าผลงาน — ยังไม่ได้ยืนยันว่าตรงกับความจริง
- **คำรับประกันของร้านใช้ "10 ปี" ทุกหน้าแล้ว** ส่วนตารางสเปกรายรุ่นยังเป็นตัวเลขคนละชุด (5–8 ปี ตามรุ่น) ซึ่งก็ยังเป็นข้อมูลตัวอย่าง

— อัปเดตล่าสุด 2026-07-22
