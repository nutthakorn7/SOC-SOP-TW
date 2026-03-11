# PB-03: Forbidden Spawn Execution detected

| รายการ | รายละเอียด |
|--------|-----------|
| **Alert Name** | Forbidden Spawn Execution detected |
| **Severity** | 🟠 High |
| **MITRE ATT&CK** | T1059 (Command and Scripting Interpreter), T1203 (Exploitation for Client Execution) |
| **Platform** | SentinelOne EDR/XDR |
| **วันที่สร้าง** | มีนาคม 2026 |

---

## 1. ภาพรวมของ Alert

**Forbidden Spawn Execution** เป็น Alert ที่เกิดขึ้นเมื่อ SentinelOne ตรวจพบว่า **Process หนึ่ง สร้าง Child Process ที่ไม่ควรจะเกิดขึ้น** ตามปกติ

ตัวอย่าง:
- `winword.exe` (Word) → สร้าง `cmd.exe` หรือ `powershell.exe` ← **ไม่ปกติ!**
- `excel.exe` (Excel) → สร้าง `mshta.exe` ← **ไม่ปกติ!**
- `outlook.exe` → สร้าง `wscript.exe` ← **ไม่ปกติ!**

Alert นี้มักเกี่ยวข้องกับ:
- 📎 **ไฟล์แนบที่เป็น Malicious Document** (Phishing)
- 💣 **Macro Malware** (VBA Macro ใน Office)
- 🔗 **Exploit ผ่านเอกสาร** (เช่น Follina, DDE Attack)

---

## 2. ขั้นตอนการตอบสนอง (Response Steps)

### Step 1: รับ Alert และเปิด Incident Ticket
1. เข้า **SentinelOne Console** → **Incidents / Threats**
2. ค้นหา Alert: `Forbidden Spawn Execution detected`
3. จดบันทึก:
   - **Endpoint Name** และ **IP Address**
   - **Logged-in User**
   - **Parent Process** (เช่น `winword.exe`, `excel.exe`)
   - **Child Process** ที่ถูก Spawn (เช่น `cmd.exe`, `powershell.exe`)
   - **Command Line** ที่ใช้ ← **สำคัญมาก!**
   - **Timestamp**
4. เปิด Incident Ticket

### Step 2: วิเคราะห์ Attack Storyline
1. คลิก **"Attack Storyline"** ใน SentinelOne
2. ดูลำดับ Process:
   ```
   [Parent Process] → [Forbidden Child Process] → [ดำเนินการอะไรต่อ?]
   ```
3. ตรวจสอบ **Command Line Arguments** ของ Child Process:
   - ⚠️ `powershell.exe -enc <Base64>` → **Encoded Command (มัลแวร์ซ่อนคำสั่ง)**
   - ⚠️ `cmd.exe /c certutil -urlcache ...` → **ดาวน์โหลดไฟล์จากภายนอก**
   - ⚠️ `mshta.exe http://...` → **เรียก Script จาก URL**
   - ⚠️ `wscript.exe C:\Users\...\*.vbs` → **รัน VBScript**
4. ตรวจสอบว่ามี **Network Connection** จาก Child Process:
   - ถ้ามีการติดต่อ IP/Domain ภายนอก → **สงสัย Malware Download หรือ C2**
5. **Screenshot** Attack Storyline

### Step 3: ระบุแหล่งที่มา (Source Identification)
1. ดู Parent Process:

| Parent Process | แหล่งที่มาที่เป็นไปได้ |
|---------------|---------------------|
| `winword.exe` | เปิดไฟล์ Word ที่มี Macro |
| `excel.exe` | เปิดไฟล์ Excel ที่มี Macro |
| `outlook.exe` | เปิดไฟล์แนบจาก Email |
| `powerpnt.exe` | เปิด PowerPoint ที่มี Macro |
| `acrobat.exe` / `acrord32.exe` | เปิด PDF ที่มี Exploit |
| `iexplore.exe` / `msedge.exe` | เยี่ยมชมเว็บไซต์ที่เป็นอันตราย |

2. **ค้นหาไฟล์ต้นทาง**:
   - ถ้า Parent เป็น Office → หาไฟล์ `.docx`, `.docm`, `.xlsx`, `.xlsm` ที่ผู้ใช้เปิด
   - ดูใน Threat Details → **Indicators** → หา File Path ของเอกสาร
3. **ถ้ามาจาก Email** → ติดต่อ Email Team เพื่อ:
   - หา Email ต้นทาง
   - Block Sender
   - ลบ Email จากทุก Mailbox ที่ได้รับ

### Step 4: ตรวจสอบ Threat Intelligence
1. ถ้ามี **SHA256 Hash** ของไฟล์ต้นทาง (เอกสาร) → ค้นหาใน VirusTotal
2. ถ้ามี **URL/IP** ที่ถูกติดต่อ → ค้นหาใน:
   - VirusTotal
   - AbuseIPDB (https://www.abuseipdb.com)
3. ถ้ามี **Command Line** ที่น่าสงสัย → ค้นหาใน Google/VirusTotal ว่าเป็น Known Attack Pattern หรือไม่
4. บันทึกผลลง Ticket

### Step 5: การตัดสินใจ (True Positive vs False Positive)

| เงื่อนไข | ผลวินิจฉัย |
|---------|-----------|
| Office Process → `powershell.exe` ด้วย Encoded Command | ✅ True Positive |
| Office Process → `cmd.exe` ดาวน์โหลดไฟล์ | ✅ True Positive |
| Office Process → `mshta.exe` + URL | ✅ True Positive |
| ซอฟต์แวร์ที่รู้จัก (เช่น Update Agent) สร้าง Process ปกติ | ❌ Possible False Positive |
| Script ของ IT Admin ที่ใช้เป็นประจำ | ❌ Possible False Positive |

- ถ้าเป็น **True Positive** → ทำ Step 6 ต่อ
- ถ้าเป็น **False Positive** → ข้ามไป Step 9

### Step 6: Containment (กักกัน)
1. **Network Quarantine**:
   - SentinelOne → **Sentinels** → เลือกเครื่อง → **"Disconnect from Network"**
2. **Kill Process Chain**:
   - Kill ทั้ง Parent และ Child Process
   - Threat Details → **"Actions"** → **"Kill"**
3. **Quarantine Malicious Files**:
   - Quarantine ไฟล์เอกสารต้นทาง
   - Quarantine ไฟล์ที่ถูก Download (ถ้ามี)

### Step 7: Remediation (แก้ไข)
1. **Remediate** ผ่าน SentinelOne:
   - **"Actions"** → **"Remediate"**
2. ตรวจสอบ **Persistence**:
   - ใน Attack Storyline ดูว่ามีการสร้าง:
     - Scheduled Task
     - Registry Run Key
     - Startup Folder Entry
   - ถ้ามี → ต้องลบออกด้วย
3. ถ้ามีการ Download Malware เพิ่ม:
   - ค้นหาไฟล์ที่ Download มา → Quarantine ทั้งหมด
4. เปลี่ยนรหัสผ่านผู้ใช้ (ถ้า Credential อาจถูกขโมย)

### Step 8: ตรวจสอบการแพร่กระจาย
1. **Deep Visibility** → ค้นหา:
   ```
   SrcProcParentName In Contains ("winword","excel","outlook","powerpnt") AND TgtProcName In Contains ("cmd","powershell","mshta","wscript","cscript")
   ```
2. ถ้า Email เป็นแหล่งที่มา → ตรวจสอบว่ามีผู้ใช้คนอื่นเปิด Email เดียวกันหรือไม่
3. ถ้าพบหลายเครื่อง → **Escalate**

### Step 9: อัปเดต Verdict และปิด Incident
1. ตรวจสอบหลัง Remediation (รอ 15-30 นาที)
2. ปลด Network Quarantine
3. ตั้ง Analyst Verdict:
   - **True Positive** → ปิด Incident พร้อมสรุป
   - **False Positive** → สร้าง Exclusion Rule:
     - ใช้ **Parent Process Path + Child Process + Hash** เป็นเงื่อนไข
     - ⚠️ อย่า Exclude กว้างเกินไป
4. ปิด Incident Ticket

---

## 3. Escalation Criteria

| สถานการณ์ | ดำเนินการ |
|-----------|----------|
| มี C2 Communication ยืนยัน | แจ้ง SOC Manager + IR Team |
| มีการ Download มัลแวร์เพิ่ม | แจ้ง SOC Manager |
| Phishing Campaign พบหลายเครื่อง | แจ้ง SOC Manager + Email Team |
| ข้อมูลสำคัญอาจถูกขโมย | แจ้ง SOC Manager + Management |

---

## 4. แนวทางป้องกัน

- **Disable VBA Macro** ใน Office ผ่าน Group Policy (ถ้าไม่จำเป็นต้องใช้)
- **ตั้ง Attack Surface Reduction (ASR) Rules** สำหรับ Office:
  - Block Office apps from creating child processes
  - Block Office apps from creating executable content
- **อบรมผู้ใช้** เรื่อง Phishing Awareness
- **ใช้ Email Security Gateway** กรอง Malicious Attachments
- ตั้ง SentinelOne Policy เป็น **Protect** mode
