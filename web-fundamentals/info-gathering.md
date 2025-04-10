# Writeup: Information Gathering
ใช้ศึกษาวิธีการรวบรวมข้อมูลจากเว็บแอปโดยใช้การดัก request, วิเคราะห์ header, fuzz endpoint และตรวจสอบ subdomain เพื่อขยายพื้นที่โจมตี (Attack Surface) ก่อนเข้าสู่การเจาะรบบที่ลึกขึ้น

## เครื่องมือที่ใช้
- Burp Suite (ใช้ Intercept และ Repeater ร่วมกัน)
- curl (ยิง request แบบ manual)
- ffuf (ค้นหา path หรือไฟล์ ที่ซ่อนอยู่)
- dig, nslookup (วิเคราะห์ DNS/Subdomain)
- WhatWeb / Wappalyzer (ดูว่าเว็บเขียนด้วยอะไรและใช้เทคโนโลยีอะไรบ้าง)
