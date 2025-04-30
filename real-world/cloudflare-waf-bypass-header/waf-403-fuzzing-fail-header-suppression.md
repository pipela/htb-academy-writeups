# [Writeup] WAF 403 fuzzing fail header suppression

Wordlist path ที่ใช้ทดสอบเบื้องต้น
/
wp-login.php
wp-admin/
wp-admin/admin.php
wp-admin/admin-ajax.php
wp-json/

## Summary
Command : 
ffuf -u https://target.com/FUZZ
-w wordlist_fuzz.txt
-H "user-agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/135.0.0.0 Safari/537.36"
-H "Cookie: $(cat cf_cookie.txt)"
-H "X-Forwarded-For: 127.0.0.1"
-H "X-Original-URL: /admin"
-H "Referer: https://target.com"
-p 1
-rate 1
-t 3
-fc 404
-r
-of json
-o result.json

ระหว่างทำการ Fuzz เว็บไซต์เป้าหมายผ่าน Cloudflare ด้วย ffuf พบว่าทุก request ที่ยิงจะได้รับ HTTP 403 ทันที แม้จะยิง path ที่เป็นหน้าเว็บปกติ เช่น /

หลังจากวิเคราะห์ header พบว่าสาเหตุไม่ได้มาจาก path ผิดหรือ cookie ไม่ถูกต้องหรือติดหน้า Human check แต่เกิดจากการใส่ Header ที่ผิดปกติในแบบ Pentester style เช่น:
เพิ่ม header พิเศษ
- X-Forwarded-For: 127.0.0.1
- X-Original-URL: /admin

การลบ header เหล่านี้ออก ทำให้สามารถยิง fuzz ต่อได้ทันทีโดยไม่ถูก block

## Tools & Techniques

- ffuf สำหรับ Fuzz Directory
- Burp Suite สำหรับดูความแตกต่างของ Header
- ตรวจพฤติกรรม HTTP status แบบละเอียด 200 vs 403
- ใช้ Delay และ Header แบบ Browser จริง

---

## Steps

1. ใช้ ffuf ยิงไปยัง https://target.com/FUZZ
2. ใส่ header:  -H "X-Forwarded-For: 127.0.0.1" -H "X-Original-URL: /admin"
3. พบว่า request ทั้งหมดตอบกลับ 403 แม้จะเป็น path ปกติ
4. ลบ header เหล่านี้ แล้วยิงอีกครั้ง:
5. พบว่า request ผ่านได้ code 200, 301

## Analysis

- WAF (Cloudflare) มี rule ที่ block header แปลกซึ่งมักใช้ในการ bypass
- ระบบตั้งค่าผิดพลาดจนทำให้ block ทุก request ที่มี header สไตล์นี้
- ผลคือ: ทดสอบใช้ ffuf/curl แบบมี header สไตล์ pentest จะยิงไม่ผ่านเลย

---

## 

- ไม่ใช่ทุก 403 คือ path ผิด บางครั้งคือพฤติกรรมเรา ดูเป็น pentester เกินไป จน WAF มองออก
- ต้องเข้าใจพฤติกรรมของ WAF ไม่ใช่แค่ brute force fuzz อย่างเดียว
- งดใช้ header น่าสงสัยถ้าไม่จำเป็น
- WAF (Cloudflare) ไม่ชอบพฤติกรรมจำพวก Pentester ทำให้โดนบล๊อคทุก request

---
