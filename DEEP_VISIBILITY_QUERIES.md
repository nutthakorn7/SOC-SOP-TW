<h1 align="center">🔍 Deep Visibility Query Cheatsheet</h1>
<h4 align="center">รวม Query สำเร็จรูปสำหรับ SentinelOne Deep Visibility — Copy-Paste ได้เลย!</h4>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Deep_Visibility-6C2DC7?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Queries-30+-blue?style=for-the-badge" />
</p>

---

## 📌 วิธีใช้

1. เข้า **SentinelOne Console** → **Visibility** → **Deep Visibility**
2. Copy Query จากเอกสารนี้ → Paste ลงใน Search Bar
3. แก้ไขค่าใน `< >` ให้ตรงกับ Case (เช่น `<Hash>` → ใส่ Hash จริง)
4. กด **Search**

> [!NOTE]
> Deep Visibility เก็บข้อมูลย้อนหลังประมาณ **14 วัน** (ขึ้นอยู่กับ License)

---

## 1️⃣ ค้นหาด้วย File Name / Hash

### ค้นหาไฟล์ตามชื่อ
```
FileName = "<filename>"
```

### ค้นหาไฟล์ด้วย SHA256 Hash
```
FileSHA256 = "<hash>"
```

### ค้นหาไฟล์ในหลายชื่อพร้อมกัน
```
FileName In Contains ("<name1>","<name2>","<name3>")
```

---

## 2️⃣ Query เฉพาะแต่ละ Alert

### 🔴 PB-01: dllhostex.exe
```
FileName = "dllhostex.exe"
```

### 🔴 PB-02: spoolsv.exe ปลอม (นอก System32)
```
FileName = "spoolsv.exe" AND (NOT FilePath Contains "System32")
```

### 🔴 PB-03: Forbidden Spawn (Office → cmd/powershell)
```
SrcProcParentName In Contains ("winword","excel","outlook","powerpnt") AND TgtProcName In Contains ("cmd","powershell","mshta","wscript","cscript")
```

### 🔴 PB-04: svchost.exe ปลอม
```
FileName = "svchost.exe" AND NOT FilePath = "C:\Windows\System32\svchost.exe"
```

### 🟡 PB-05: Rufus
```
FileName Contains "rufus"
```

### 🔴 PB-06: conres.dll
```
FileName = "conres.dll"
```

### 🔴 PB-07: OInstall + Pirated SW
```
FileName In Contains ("OInstall","KMSPico","KMSAuto","office-activator","C2R")
```

### 🔴 PB-08: Writeable Process (ค้นหาด้วย Hash)
```
FileSHA256 = "<hash>"
```

### 🔴 PB-09: Defender Exclusion Tampering
```
CmdLine Contains "Add-MpPreference -ExclusionPath"
```
```
CmdLine Contains "Set-MpPreference -DisableRealtimeMonitoring"
```

### 🟢 PB-10: bwswfcfg.exe
```
FileName = "bwswfcfg.exe"
```

---

## 3️⃣ ค้นหา Process ที่น่าสงสัย

### Process ที่รันจาก Path ผิดปกติ
```
SrcProcName In ("svchost.exe","spoolsv.exe","csrss.exe","lsass.exe") AND NOT SrcProcImagePath Contains "System32"
```

### PowerShell Encoded Command
```
TgtProcName = "powershell.exe" AND TgtProcCmdLine Contains "-enc"
```

### PowerShell ดาวน์โหลดไฟล์
```
TgtProcName = "powershell.exe" AND TgtProcCmdLine In Contains ("Invoke-WebRequest","wget","curl","DownloadFile","DownloadString","IEX")
```

### certutil ดาวน์โหลดไฟล์
```
TgtProcName = "certutil.exe" AND TgtProcCmdLine Contains "urlcache"
```

### mshta เรียก URL
```
TgtProcName = "mshta.exe" AND TgtProcCmdLine Contains "http"
```

### rundll32 รัน DLL จาก Path ผิดปกติ
```
TgtProcName = "rundll32.exe" AND TgtProcCmdLine In Contains ("Temp","AppData","ProgramData","Users")
```

### regsvr32 Silent Registration
```
TgtProcName = "regsvr32.exe" AND TgtProcCmdLine Contains "/s"
```

---

## 4️⃣ ค้นหา Network Activity

### Process ที่ติดต่อ IP ภายนอก
```
NetConnStatus = "SUCCESS" AND SrcProcName = "<process_name>"
```

### ค้นหา Connection ไปยัง IP ที่ระบุ
```
NetDestinationIp = "<ip_address>"
```

### ค้นหา Connection ไปยัง Domain
```
DNS Contains "<domain>"
```

### ค้นหา Port ที่น่าสงสัย
```
NetDestinationPort In (4444,5555,8080,8443,1234,9999)
```

---

## 5️⃣ ค้นหา File Operations

### ไฟล์ที่ถูกสร้างใน Path ที่น่าสงสัย
```
EventType = "File Creation" AND FilePath In Contains ("Temp","AppData\\Local","ProgramData")
```

### ไฟล์ .exe ที่ถูกสร้างใหม่
```
EventType = "File Creation" AND FilePath Contains ".exe"
```

### ไฟล์ที่ถูกสร้างใน Excluded Path (PB-09)
```
FilePath Contains "<excluded_path>" AND EventType = "File Creation"
```

### สัญญาณ Ransomware — ไฟล์ที่ถูกแก้ไขจำนวนมาก
```
EventType = "File Modification" AND SrcProcName = "<process_name>" | count by FilePath
```

---

## 6️⃣ ค้นหา Persistence

### Scheduled Task ถูกสร้าง
```
EventType = "Task Register" OR (TgtProcName = "schtasks.exe" AND TgtProcCmdLine Contains "/create")
```

### Registry Run Key ถูกแก้ไข
```
EventType = "Registry Value Modified" AND RegistryPath Contains "CurrentVersion\\Run"
```

### Service ถูกสร้างใหม่
```
EventType = "Service Installed" OR (TgtProcName = "sc.exe" AND TgtProcCmdLine Contains "create")
```

---

## 7️⃣ ค้นหา Credential Access

### เข้าถึง LSASS
```
TgtProcName = "lsass.exe" AND EventType = "Open Remote Process Handle"
```

### เครื่องมือ Credential Dumping
```
SrcProcName In Contains ("mimikatz","procdump","comsvcs") OR TgtProcCmdLine Contains "sekurlsa"
```

---

## 8️⃣ ค้นหา Lateral Movement

### PsExec / Remote Execution
```
SrcProcName In Contains ("psexec","psexesvc") OR (TgtProcName = "cmd.exe" AND TgtProcCmdLine Contains "\\\\")
```

### WMI Remote Execution
```
SrcProcName = "wmiprvse.exe" AND TgtProcName In ("cmd.exe","powershell.exe")
```

---

## 🔧 Tips สำหรับ Deep Visibility

| Tip | คำอธิบาย |
|:----|:---------|
| ใช้ `Contains` | ค้นหาบางส่วน — ไม่ต้องพิมพ์ครบ |
| ใช้ `In Contains (...)` | ค้นหาหลายค่าพร้อมกัน |
| ใช้ `NOT` | กรอง False Positive ออก |
| ใช้ `AND` / `OR` | รวมเงื่อนไข |
| กด **Group By** | ดูเป็น Summary (เช่น Group by Endpoint) |
| เลือก **Time Range** | เลือกช่วงเวลาให้ตรงกับ Alert |

---

<p align="center">
  <b>🛡️ SOC Team — TW Site</b><br/>
  <i>อัปเดตล่าสุด: มีนาคม 2026</i>
</p>
