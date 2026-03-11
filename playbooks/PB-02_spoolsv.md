# PB-02: spoolsv.exe detected as Malware

| รายการ | รายละเอียด |
|--------|-----------|
| **Alert Name** | spoolsv.exe detected as Malware |
| **Severity** | 🟠 High |
| **MITRE ATT&CK** | T1036.005 (Masquerading: Match Legitimate Name), T1543.003 (Create/Modify System Process: Windows Service) |
| **Platform** | SentinelOne EDR/XDR |
| **วันที่สร้าง** | มีนาคม 2026 |

---

## 1. ภาพรวมของ Alert

**spoolsv.exe** เป็น Windows Print Spooler Service ที่ **เป็นไฟล์ของ Windows จริง**  
อย่างไรก็ตาม มัลแวร์หลายตระกูลมักปลอมชื่อเป็น `spoolsv.exe` หรือใช้ช่องโหว่ของ Print Spooler (เช่น **PrintNightmare — CVE-2021-34527**) ในการโจมตี

**สิ่งสำคัญ**: ต้องแยกแยะระหว่าง:
- ✅ `spoolsv.exe` ตัวจริง → อยู่ที่ `C:\Windows\System32\spoolsv.exe`
- ❌ `spoolsv.exe` ปลอม → อยู่ในโฟลเดอร์อื่น เช่น `C:\Users\`, `C:\Temp\`, `AppData`

---

## 2. ขั้นตอนการตอบสนอง (Response Steps)

### Step 1: รับ Alert และเปิด Incident Ticket
1. เข้าสู่ **SentinelOne Console** → **Incidents** หรือ **Threats**
2. ค้นหา Alert: `spoolsv.exe detected as Malware`
3. จดบันทึก:
   - **Endpoint Name**
   - **IP Address**
   - **Logged-in User**
   - **Timestamp**
   - **File Path** ของ `spoolsv.exe` ← **สำคัญมาก!**
   - **SHA256 Hash**
4. เปิด Incident Ticket

### Step 2: ตรวจสอบ File Path (จุดตัดสินสำคัญ)
1. ดู **File Path** ใน Threat Details:

| File Path | การวินิจฉัย |
|-----------|------------|
| `C:\Windows\System32\spoolsv.exe` | อาจเป็น **False Positive** หรือ PrintNightmare Exploit |
| `C:\Users\<username>\...` | **True Positive** — มัลแวร์แน่นอน |
| `C:\Temp\` หรือ `C:\ProgramData\` | **True Positive** — มัลแวร์แน่นอน |
| ที่อื่นนอกเหนือจาก System32 | **True Positive** — มัลแวร์แน่นอน |

2. **ถ้า Path ไม่ใช่ System32** → ข้ามไป **Step 4** (ยืนยัน Malicious ทันที)
3. **ถ้า Path อยู่ใน System32** → ทำ Step 3 ต่อ

### Step 3: ตรวจสอบ spoolsv.exe ใน System32 (กรณี Legitimate Path)
> ⚠️ ทำ Step นี้เฉพาะกรณีไฟล์อยู่ใน `C:\Windows\System32\`

1. ตรวจสอบ **Digital Signature**:
   - ใน Threat Details → ดู **Signer** → ควรเป็น `Microsoft Windows`
   - ถ้า **ไม่มี Signature** หรือ Signer ไม่ใช่ Microsoft → **Malicious**
2. ตรวจสอบ **File Size**:
   - `spoolsv.exe` ตัวจริงมีขนาดประมาณ **65-80 KB**
   - ถ้าขนาดต่างจากนี้มาก → **น่าสงสัย**
3. ตรวจสอบ **Process Behavior**:
   - เข้า Attack Storyline → ดู Parent Process:
     - ✅ ปกติ: Parent = `services.exe`
     - ❌ ผิดปกติ: Parent = `cmd.exe`, `powershell.exe`, `explorer.exe`
4. ตรวจสอบว่าเกี่ยวกับ **PrintNightmare** หรือไม่:
   - ถ้ามี DLL ถูกโหลดจากนอก `C:\Windows\System32\` → **สงสัย PrintNightmare**
   - ถ้ามี `spoolsv.exe` สร้าง `rundll32.exe` → **สงสัย PrintNightmare**

### Step 4: ตรวจสอบ Hash ด้วย Threat Intelligence
1. คัดลอก **SHA256 Hash**
2. ค้นหาใน **VirusTotal** (https://www.virustotal.com)
3. ตรวจสอบ:
   - Detection Rate → > 10 engines = **Malicious**
   - Family Name → ดูว่าเป็นมัลแวร์ตระกูลอะไร
   - First Seen Date → ถ้าเป็นไฟล์ใหม่มาก → น่าสงสัย
4. บันทึกผลลง Ticket

### Step 5: ตรวจสอบ Attack Storyline อย่างละเอียด
1. คลิก **"Attack Storyline"** ใน SentinelOne
2. ตรวจสอบ:
   - **Network Connections**: มี Connection ไปหา IP ภายนอกหรือไม่
   - **File Drops**: มีการสร้างไฟล์อื่นหรือไม่
   - **Registry Changes**: มีการแก้ไข Registry เพื่อ Persistence หรือไม่
   - **Lateral Movement**: มีการพยายามเข้าเครื่องอื่นหรือไม่
3. **Screenshot** Attack Storyline

### Step 6: ตรวจสอบการแพร่กระจาย (Scope Analysis)
1. ไปที่ **Deep Visibility** → ค้นหา:
   ```
   FileName = "spoolsv.exe" AND (NOT FilePath Contains "System32")
   ```
2. ค้นหาด้วย Hash เพื่อดูว่ามีเครื่องอื่นที่พบหรือไม่
3. ถ้าพบหลายเครื่อง → **Escalate เป็น Critical**

### Step 7: Containment (กักกัน)
1. **Network Quarantine** เครื่องที่โดน:
   - SentinelOne → **Sentinels** → เลือกเครื่อง → **"Actions"** → **"Disconnect from Network"**
2. **Kill Process**:
   - ⚠️ ถ้าเป็น `spoolsv.exe` ใน System32 → การ Kill จะทำให้การพิมพ์ไม่ทำงาน (ยอมรับได้)
   - Threat Details → **"Actions"** → **"Kill"**
3. **Quarantine File**:
   - **"Actions"** → **"Quarantine"**

### Step 8: Remediation (แก้ไข)
1. คลิก **"Actions"** → **"Remediate"**
2. ถ้าเป็น PrintNightmare:
   - ต้องติดตั้ง Windows Security Patch ที่เกี่ยวกับ Print Spooler
   - พิจารณา Disable Print Spooler Service ถ้าเครื่องไม่ต้องใช้พิมพ์:
     - SentinelOne Remote Shell → `sc config spooler start= disabled`
3. ถ้าเป็นมัลแวร์ปลอมชื่อ:
   - Remediate + Rollback ตามปกติ
   - ตรวจสอบ Persistence Mechanism (Scheduled Task, Registry Run Key)

### Step 9: ตรวจสอบหลัง Remediation
1. รอ 15-30 นาที → ตรวจสอบว่าไม่มี Alert ใหม่
2. ตรวจสอบว่า Print Spooler Service ทำงานปกติ (ถ้าจำเป็น)
3. ปลด Network Quarantine
4. แจ้ง End User

### Step 10: อัปเดต Verdict และปิด Incident
1. Analyst Verdict → **True Positive** หรือ **False Positive**
2. ถ้าเป็น False Positive:
   - สร้าง **Exclusion** โดยใช้ Path + Hash
   - ⚠️ อย่า Exclude ด้วย Path อย่างเดียว ต้องใช้ Hash ด้วยเสมอ
3. สรุปผลใน Incident Ticket แล้วปิด

---

## 3. Escalation Criteria

| สถานการณ์ | ดำเนินการ |
|-----------|----------|
| ยืนยัน PrintNightmare Exploit | แจ้ง SOC Manager + IT Patch Team |
| มัลแวร์พบหลายเครื่อง | แจ้ง SOC Manager + Incident Response Team |
| มี Lateral Movement | แจ้ง SOC Manager ทันที |

---

## 4. แนวทางป้องกัน

- **ติดตั้ง Patch** สำหรับ Print Spooler Vulnerability ทุกเครื่อง
- **Disable Print Spooler** บนเครื่องที่ไม่จำเป็นต้องพิมพ์ (เช่น Server)
- ตั้ง SentinelOne Policy เป็น **Protect** mode
- Monitor ด้วย Deep Visibility Query หา `spoolsv.exe` ที่ไม่อยู่ใน System32
