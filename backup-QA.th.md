# Trinity Guardian — คลังคำตอบสำรองสำหรับกรรมการ (backup-QA.th.md)

> เตรียมตัวตอบคำถามกรรมการหลัง pitch 4 นาที เรียงจากคำถามที่เจ็บที่สุดก่อน
> กฎเหล็กในการตอบ: ตอบในลมหายใจเดียว แล้วหยุด — ถ้ากรรมการอยากเห็นจริง ค่อยเปิด dashboard
> ชื่อ/ตัวเลขที่ต้องไม่พลาด: `agents.trinityguard.eth` · agent `momo` / `rogue` · ต่อรายการ ≤ 5 USDC · ต่อวัน ≤ 50 USDC · Sepolia · USDC บน Base Sepolia · Intercepta ตรวจข้อมูล mainnet

---

## ก. คำถามหนัก (สถาปัตยกรรม & threat model)

**"middleware เป็นตัวรันเช็ค — ถ้ามันโดนแฮ็ก ที่ทำทั้งหมดนี้ไปทำไม?"**
> สิทธิ์ทั้งหมดอยู่บน chain: spend role, cap, วันหมดอายุ middleware โดนแฮ็ก กรณีเลวร้ายสุดคือทำให้จ่ายไม่ได้ (stop) — ไม่สามารถสั่งเซ็นข้าม role ที่ถูก revoke หรือ asset ที่ผิดได้ เพราะ gate อ่านสถานะ chain สดผ่าน `eth_call` ก่อนเซ็นทุกครั้ง worst case คือ denial of service ไม่ใช่เงินหาย

**"runtime ที่โดนแฮ็กก็ข้าม Guardian ไปเซ็นด้วย burner key ตรงๆ ได้นี่?"**
> ใช่ — สำหรับ unmanaged burner key ปัจจุบัน นี่คือเหตุผลที่โครงสร้างนี้ชี้ทางไปขั้นถัดไป: key ของ agent ควรเป็น session key แบบ smart account ที่เซ็นได้เฉพาะผ่านเส้นทาง Guardian (session keys / แนวทาง EIP-7702) ในขอบเขต hackathon นี้ wallet เป็น throwaway — เราพูดตรงๆ แบบนี้ใน debrief ว่าเป็น scope ที่ยอมรับ

**"ทำไม daily spend accumulator ไม่เก็บบน chain?"**
> เพราะ gas การเขียนทุกไมโครเพย์เมนต์ลง chain แพงกว่าตัวเพย์เมนต์เอง *ตัวกฎ* อยู่บน chain (perTxMax, dailyCap, role, วันหมดอายุ) — *ตัวนับ* เป็นของ middleware แม้ถูกโกงให้นับน้อย ก็ยังขนย้ายเงินข้าม role ที่ revoke หรือ verdict แดงไม่ได้

**"จะใช้ ENS ทำไม? ตาราง mapping ใน contract ก็ได้"**
> เหตุผลสามข้อ: ตัวตน (counterparty ที่มีชื่อ + ENSIP-26 records น่าเชื่อถือกว่า address เปล่า), ลำดับชั้น (registry ระดับ agent พร้อม ENS roles — register/revoke/resolve เป็น primitive ที่มีอยู่แล้ว ไม่ต้องประดิษฐ์เอง), และการหมดอายุ (expiring subname = สิทธิ์ที่ revoke ตัวเอง ไม่ต้องส่ง tx มาล้าง) ENS คือชั้น identity ไม่ใช่ของประดับ

**"ถ้า Intercepta ล่ม / ไม่คืน verdict ล่ะ?"**
> ไม่มี verdict = soft fail → ส่งต่อ World ID ให้คนตัดสิน เราไม่มีทางเปิดผ่านโดยปริยายเมื่อข้อมูลหาย (กฎเดียวกับ payTo ที่ไม่อยู่ใน allowlist)

**"เส้นแบ่ง soft fail / hard fail ชัดแค่ไหน?"**
> Hard fail → ปฏิเสธ + log: verdict แดง, asset ผิด Soft fail → ถามคนผ่าน World ID: amount > perTxMax, payTo ไม่อยู่ใน allowlist, ไม่มี verdict จาก Intercepta ส่วน kill switch (role ถูก revoke) เช็ค *ก่อน* ทุกอย่าง — ชั้น 2/3 ไม่ได้รันเลย

**"ถ้าโทรศัพท์เจ้าของไม่ออนไลน์ตอนต้อง approve?"**
> การจ่ายค้างอย���่ที่ HELD จนคำขอหมดเวลา (มี countdown บน dashboard) แล้วถูกปฏิเสธด้วยเหตุผล "expired" — เงินไม่ขยับโดยปริยาย นี่คือทิศทางที่ล้มเหลวที่ถูกต้อง

---

## ข. มุมคำถามเฉพาะ sponsor

**World ($7,500): "World ID ของคุณ meaningful ตรงไหน?"**
> การจ่ายเงินที่เสี่ยงคือ meaningful action ที่สุดแล้ว journey ครบ: request → เจ้าของ verify ใน World App → validated ที่ backend → protected action และเส้นทาง "ปฏิเสธ" เป็น demo act ชั้นเยี่ยม ไม่ใช่ error state

**ENS ($6,000): "ENS central ยังไง ไม่ใช่แค่ของตกแต่ง?"**
> Guardian เซ็นไม่ได้เว้นแต่ chain จะบอกว่าเซ็นได้ อ่าน spend role สดทุกครั้งก่อนเซ็น (`eth_call`, ไม่ cache), text records เก็บ cap, expiring subname เก็บวันหมดอายุ demo kill switch = revoke ผ่าน ENSv2 EAC หยุดการจ่ายที่เขียวทุกอย่าง

**Intercepta ($2,000): "เรียก API จริงหรือ mock?"**
> API จริง — quick-scan-address, scan-token, scan-message เรา screen address บน mainnet แม้ payment รันบน testnet Act 2 (scam flag) กับ Act 4 (lookalike token) จบด้วยการปฏิเสธ *พร้อมเหตุผลโชว์บนจอ*

**Curvegrid ($1,000): "spending limits, approved counterparties, required human approvals?"**
> ครบทั้งสาม — limits มาจาก text records บน chain (ไม่ใช่ config file), counterparty ผ่าน ENS identity ที่มีชื่อ + allowlist, human approval ผ่าน World ID เรายก mandate ตัวอย่างของทาง Curvegrid ขึ้นไปไว้บน chain เลย

---

## ค. ความจริงของ demo (ตอบตรง ตอบสั้น)

**"รันบน mainnet ไหม?"**
> Identity/mandate บน Sepolia, การจ่ายเป็น USDC testnet บน Base Sepolia, แต่ screening ตรวจข้อมูล mainnet จริง สำหรับ demo ของ *ตัวเฝ้า* ใน hackathon — testnet + screening จริง คือ configuration ที่ซื่อสัตย์ที่สุด

**"ตรงไหนที�� mock?"**
> ไม่มีใน decision path มีแต่ demo controls บน dashboard (revoke/restore role, "pretend 47 spent today") เพื่อ *ขับเคลื่อน* ห้า acts แบบสด — ตัวเช็คเป็น API call จริงและ chain read จริงทั้งคู่

**"ทำไมต้องมี agent `rogue`?"**
> มันคือตัวแสดง kill switch: โค้ดเดียวกัน แต่ role ถูก revoke (`roleActive: false`) พิสูจน์ว่าการ revoke เป็นเรื่อง identity ระดับ agent ไม่ใช่ config ระดับ wallet

---

## ง. Scope & ทีม

**"เวลาไม่พอจะตัดอะไรก่อน?"** → ตัดชั้น World ID (soft fail กลายเป็นปฏิเสธอัตโนมัติ) — stack สองชั้นยังยืนได้ เราวาง fallback นี้ไว้ตั้งแต่ต้น
**"หลัง hackathon ทำต่อยังไง?"** → session key / smart-account signing ให้ key เดียวที่ agent ถือคือ key ที่ scope ผ่าน Guardian; ย้าย spend accumulator ขึ้น chain ถ้าเศรษฐศาสตร์ gas เปลี่ยน
**"debrief / ข้อจำกัด?"** → พูดเองก่อนถูกถาม: unmanaged burner key, accumulator off-chain, residual trust ใน middleware — ทั้งสามเขียนไว้แล้ว ไม่มีอะไรถูกซ่อน

---

## จ. ตัวเลขตอบเร็ว

| คำถาม | คำตอบ |
|---|---|
| Cap? | ≤ 5 USDC ต่อการจ่าย · ≤ 50 USDC ต่อวัน (จาก text records บน chain) |
| Namespace? | `agents.trinityguard.eth` ใต้ `trinityguard.eth` (ENSv2, Sepolia) |
| Agents? | `momo` (ถูกเฝ้า), `rogue` (role ถูก revoke) + `shopping`, `research`, `travel` |
| การจ่าย? | x402, USDC บน Base Sepolia (eip155:84532) |
| Screening? | Intercepta — address / token / message, ข้อมูล mainnet |
| หน้าต่าง approve? | มี countdown + หมดเวลา = ปฏิเสธ (ไม่มี auto-pay) |
| กฎเหล็ก? | ชั้นไหนปฏิเสธ การจ่ายหยุด — ไม่มีชั้นไหน override กันได้ |

---

## ฉ. ถ้ากรรมการถามต่อว่า "แล้วทำไมเราต้องเชื่อ middleware ของพวกคุณ?"

> คุณไม่ต้องเชื่อเรา ���ชื่อ chain เรื่องสิทธิ์ เชื่อ Intercepta เรื่อง screening เชื่อ World เรื่องคน middleware ของเราเป็นท่อที่เปลี่ยนได้ทุกเมื่อ — และทุก decision ที่มันตัดสินมี log พร้อม evidence ที่เปิดย้อนดูได้บน dashboard
