# HTB Academy – Web Requests
## เป้าหมายของโมดูลนี้
- เรียนรู้การทำงานของ HTTP Request และ Response
- เข้าใจการใช้ HTTP Methods ต่าง ๆ เช่น GET, POST, PUT, DELETE
- เพื่อค้นหา logic flaw, bypass filter หรือจัดการข้อมูลผิดพลาดที่อาจนำไปสู่การเจาะระบบ

## Tools ที่ใช้
- Burp Suite

Header request GET:
GET / HTTP/1.1
Host: www.example.co.th
Cookie: _fbp=fb.2.1739529404982.924011222349829484; _ga=GA1.1.1127571077.1739529406; _ga_QMZG499SVT=GS1.1.1739540476.3.1.1739541768.60.0.0
Sec-Ch-Ua: "Not:A-Brand";v="24", "Chromium";v="134"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Windows"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/134.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
-----------
อธิบาย Header

Method: GET ไม่มี Body
Cookie จาก Facebook (_fbp), Google Analytics (_ga)
- ไม่ใช่จุดโจมตีโดยตรง แต่สามารถเอาไปใช้ Fingerprint ผู้ใช้
- ในบางเว็บ session ก็ฝังใน Cookie → ใช้เปลี่ยน/ขโมยเพื่อทำ Session Hijack ได้

Sec-CH-UA:
ใช้เพื่อบอกเบราว์เซอร์ว่าเราใช้ Chrome เวอร์ชันไหน OS อะไร
ในบางระบบ auth หรือ redirect logic พึ่ง header พวกนี้ (สามารถ spoof ได้)
ใช้ร่วมกับ User-Agent เพื่อ fingerprint user

Accept-Language: en-US,en;q=0.9
แจ้งว่าภาษาที่เรารองรับคือ English
- บางเว็บใช้ภาษานี้กำหนด logic เช่น redirect ไปหน้า en/home, th/home

Upgrade-Insecure-Requests: 1
บอก browser ว่าต้องการเนื้อหาแบบ HTTPS บางเว็บใช้ header นี้ใน redirect logic
อาจเจอ bug ถ้าไม่ได้ redirect จริง


User-Agent: Mozilla/5.0 (...) Chrome/134.0.0.0 Safari/537.36
ใช้ใน WAF filter หรือ block tools อย่าง sqlmap, curl
เราอาจปรับ User-Agent เป็น browser หรือ bot เพื่อดูการตอบของ logic จากระบบที่ต่างกัน เช่น GoogleBot

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
ระบบแจ้งว่าอยากได้เนื้อหาประเภท HTML, XML หรืออื่นๆ
บางระบบ API จะตอบไม่เหมือนกันถ้าเปลี่ยน Accept: application/json เพื่อใช้ทดสอบ API mapping request


กลุ่ม Sec-Fetch:
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
ใช้เพื่อป้องกัน Clickjacking หรือ CSRF
ใช้ได้กับการป้องกัน เช่น block iframe, block cross-origin






















