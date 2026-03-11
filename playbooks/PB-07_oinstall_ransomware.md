# PB-07: OInstall_x64.exe detected as Ransomware

| รายการ | รายละเอียด |
|--------|-----------|
| **Alert Name** | OInstall_x64.exe detected as Ransomware |
| **Severity** | 🔴 Critical |
| **MITRE ATT&CK** | T1486 (Data Encrypted for Impact), T1588.001 (Obtain Capabilities: Malware) |
| **Platform** | SentinelOne EDR/XDR |
| **วันที่สร้าง** | มีนาคม 2026 |

---

## 1. ภาพรวมของ Alert

**OInstall_x64.exe** หรือที่รู้จักในชื่อ **"Office C2R Install"** เป็นเครื่องมือ **โจรสลัด (Pirate Tool)** ที่ใช้ติดตั้ง Microsoft Office โดยไม่มี License

> ⚠️ **สำคัญมาก**: SentinelOne ตรวจจับเป็น **Ransomware** เพราะ:
> 1. ไฟล์นี้เป็น **Cracked Software** ที่มักฝังมัลแวร์
> 2. บาง Version ถูกฝัง **Ransomware** หรือ **Cryptominer** จริงๆ
> 3. เครื่องมือนี้มีพฤติกรรมคล้าย Malware (แก้ไข System Files, Disable Security)

**ไม่ว่าจะเป็น True Positive หรือ False Positive → ไฟล์นี้ต้องถูกลบออก** เพราะเป็นซอฟต์แวร์ละเมิดลิขสิทธิ์

---

## 2. ขั้นตอนการตอบสนอง (Response Steps)

### Step 1: 🚨 ดำเนินการเร่งด่วน (IMMEDIATE ACTIONS)
> เนื่องจาก Alert นี้เป็น **Ransomware** ต้องดำเนินการทันทีก่อนวิเคราะห์

1. เข้า **SentinelOne Console** → **Incidents / Threats**
2. ค้นหา Alert: `OInstall_x64.exe detected as Ransomware`
3. **ตรวจสอบ Mitigation Status ทันที**:
   - ✅ ถ้า SentinelOne **Killed + Quarantined** แล้ว → ดี ไปต่อ
   - ❌ ถ้ายังไม่ได้ดำเนินการ → **Kill + Quarantine ทันที**:
     - **"Actions"** → **"Kill"** → แล้ว **"Quarantine"**
4. **Network Quarantine เครื่องทันที**:
   - **Sentinels** → เลือกเครื่อง → **"Actions"** → **"Disconnect from Network"**
   - ⚠️ ทำทันที! ไม่ต้องรอวิเคราะห์ก่อน (เพราะเป็น Ransomware Alert)
5. จดบันทึกข้อมูลสำคัญ:
   - Endpoint Name, IP, User, File Path, Hash, Timestamp
6. เปิด **Incident Ticket** พร้อมระบุ Severity = **Critical**

### Step 2: ตรวจสอบว่ามีการเข้ารหัสไฟล์หรือไม่
1. ตรวจสอบใน **Attack Storyline**:
   - ดูว่า `OInstall_x64.exe` มีการ **แก้ไขไฟล์จำนวนมาก** หรือไม่
   - ⚠️ ถ้ามีการเปลี่ยน File Extension เป็น `.encrypted`, `.locked`, `.crypto` → **Ransomware กำลังทำงาน!**
   - ⚠️ ถ้ามีการสร้างไฟล์ `README.txt` หรือ `HOW_TO_DECRYPT.txt` → **Ransomware เข้ารหัสแล้ว!**
2. ถ้า **ไม่มีการเข้ารหัสไฟล์**:
   - อาจเป็น Detection ก่อน Ransomware จะทำงาน (SentinelOne หยุดทัน)
   - หรืออาจเป็น Cracked Software ที่ยังไม่ได้ทำอะไร
3. ถ้า **มีการเข้ารหัสไฟล์**:
   - ⚠️ **Escalate ทันที** → แจ้ง SOC Manager
   - เตรียม **Rollback** (ดู Step 6)

### Step 3: ตรวจสอบ Hash ด้วย Threat Intelligence
1. คัดลอก **SHA256 Hash**
2. ค้นหาใน **VirusTotal**:
   - ดู Detection Rate
   - ดู Classification → Ransomware? HackTool? PUP?
   - ดู Family Name → เช่น KMSPico, Office Activator, etc.
3. ตรวจสอบว่าเป็น:
   - **Cracked Software (HackTool/PUP)** → ยังอันตราย แต่ไม่ใช่ Ransomware
   - **Ransomware จริง** → Critical Incident
4. บันทึกผล

### Step 4: ตรวจสอบ Attack Storyline
1. ดู **Parent Process**: ใครรัน `OInstall_x64.exe`
   - ผู้ใช้รันเอง? (Double-click จาก Explorer)
   - Automated Script รัน?
2. ดู **Child Processes** / **Activities**:
   - ⚠️ Disable Windows Defender → **มัลแวร์ปิด Security**
   - ⚠️ แก้ไข Registry → อาจเป็น KMS Activation
   - ⚠️ Network Connection ไปภายนอก → **C2 / Download มัลแวร์เพิ่ม**
   - ⚠️ สร้างไฟล์ในหลายโฟลเดอร์ → **Ransomware Encryption**
3. **Screenshot** Storyline

### Step 5: ตรวจสอบการแพร่กระจาย
1. **Deep Visibility** → ค้นหา:
   ```
   FileName = "OInstall_x64.exe" OR FileName Contains "OInstall"
   ```
2. ค้นหาด้วย Hash
3. ค้นหาไฟล์ที่เกี่ยวข้อง:
   ```
   FileName In Contains ("KMSPico","KMSAuto","office-activator","C2R")
   ```
4. ถ้าพบหลายเครื่อง → **Isolate ทุกเครื่องที่พบ**

### Step 6: Remediation
1. **Remediate** ผ่าน SentinelOne:
   - **"Actions"** → **"Remediate"**
2. **Rollback** (ถ้ามีการเข้ารหัสไฟล์):
   - **"Actions"** → **"Rollback"**
   - SentinelOne ใช้ VSS Snapshot คืนค่าไฟล์ที่ถูกเข้ารหัส
   - ⚠️ **Rollback ใช้ได้เฉพาะเมื่อ SentinelOne Agent มี VSS enabled**
3. ตรวจสอบ **Persistence**:
   - ลบ Scheduled Tasks ที่เกี่ยวข้อง
   - ตรวจสอบ Services ที่ถูกเพิ่ม
   - ตรวจสอบ Registry Run Keys
4. ตรวจสอบว่า **Windows Defender** ถูก Enable กลับ
5. ตรวจสอบว่า **License ของ Office** เป็นอย่างไร:
   - ถ้าใช้ Pirated Office → ต้องให้ IT ติดตั้ง Office License ที่ถูกต้อง

### Step 7: Post-Remediation Check
1. รอ 15-30 นาที
2. ตรวจสอบ:
   - ไม่มี Alert ใหม่
   - ไฟล์ที่ถูกเข้ารหัส (ถ้ามี) ถูก Rollback สำเร็จ
   - Windows Defender ทำงานปกติ
   - ไม่มี Process ที่เกี่ยวข้องทำงาน
3. ปลด **Network Quarantine**
4. แจ้ง End User:
   - "เครื่องของคุณถูกพบซอฟต์แวร์ละเมิดลิขสิทธิ์ที่อาจเป็นอันตราย"
   - "ห้ามดาวน์โหลดหรือติดตั้งซอฟต์แวร์ที่ไม่ได้รับอนุญาต"

### Step 8: อัปเดต Verdict และปิด Incident
1. ตั้ง **Analyst Verdict** → **True Positive**
   - ⚠️ แม้จะเป็น Cracked Software ที่ไม่ใช่ Ransomware จริง ก็ถือเป็น True Positive เพราะเป็นซอฟต์แวร์อันตราย
2. สรุปใน Incident Ticket:
   - สาเหตุ: ผู้ใช้ดาวน์โหลดและรัน Pirated Office Installer
   - ผลกระทบ: (มี/ไม่มี) การเข้ารหัสไฟล์
   - การแก้ไข: Remediated + Rollback (ถ้ามี)
   - คำแนะนำ: ติดตั้ง Office License ที่ถูกต้อง
3. **แจ้ง IT Manager** เพื่อดำเนินการด้าน HR/Policy (ถ้าจำเป็น)
4. ปิด Ticket

---

## 3. Escalation Criteria

| สถานการณ์ | ดำเนินการ |
|-----------|----------|
| มีการเข้ารหัสไฟล์ (Ransomware Active) | 🔴 แจ้ง SOC Manager + IR Team **ทันที** |
| Rollback ไม่สำเร็จ | 🔴 แจ้ง SOC Manager + IT Backup Team |
| พบหลายเครื่อง | 🟠 แจ้ง SOC Manager |
| ข้อมูลสำคัญถูกเข้ารหัส | 🔴 แจ้ง SOC Manager + Management |

---

## 4. แนวทางป้องกัน

- **ห้ามใช้ Cracked/Pirated Software** อย่างเด็ดขาด — ตั้งเป็นนโยบายบริษัท
- ติดตั้ง **Microsoft Office License** ที่ถูกต้องให้ทุกเครื่อง
- ตั้ง **Application Control** Block ไฟล์ที่ชื่อ `OInstall`, `KMSPico`, `KMSAuto`
- ตั้ง SentinelOne Policy เป็น **Protect** mode
- **Enable VSS** ใน SentinelOne Agent เพื่อให้ Rollback ได้
- อบรมผู้ใช้เรื่อง **ความเสี่ยงของ Pirated Software**
