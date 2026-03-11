<h1 align="center">🔍 Deep Visibility Query Cheatsheet</h1>
<h4 align="center">รวม Query สำเร็จรูป Copy-Paste ได้เลย — แก้แค่ค่าใน &lt; &gt;</h4>

<p align="center">
  <img src="https://img.shields.io/badge/Tool-Deep_Visibility-6C2DC7?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Queries-30+-blue?style=for-the-badge" />
</p>

---

## วิธีใช้

1. เข้า SentinelOne → **Visibility** → **Deep Visibility**
2. Copy Query จากเอกสารนี้ → Paste ลง Search Bar
3. แก้ค่าใน `< >` ให้ตรงกับ Case เช่น `<hash>` → ใส่ Hash จริง
4. กด Search

Deep Visibility เก็บข้อมูลย้อนหลังประมาณ **14 วัน** (ขึ้นกับ License)

---

## ค้นหาด้วย File Name / Hash

```
FileName = "<filename>"
```

```
FileSHA256 = "<hash>"
```

ค้นหาหลายชื่อพร้อมกัน:
```
FileName In Contains ("<name1>","<name2>","<name3>")
```

---

## Query ตาม Playbook

### PB-01: dllhostex.exe
```
FileName = "dllhostex.exe"
```

### PB-02: spoolsv.exe ปลอม
```
FileName = "spoolsv.exe" AND (NOT FilePath Contains "System32")
```

### PB-03: Office สร้าง Process ต้องสงสัย
```
SrcProcParentName In Contains ("winword","excel","outlook","powerpnt") AND TgtProcName In Contains ("cmd","powershell","mshta","wscript","cscript")
```

### PB-04: svchost.exe ปลอม
```
FileName = "svchost.exe" AND NOT FilePath = "C:\Windows\System32\svchost.exe"
```

### PB-05: Rufus
```
FileName Contains "rufus"
```

### PB-06: conres.dll
```
FileName = "conres.dll"
```

### PB-07: Pirated Software
```
FileName In Contains ("OInstall","KMSPico","KMSAuto","office-activator","C2R")
```

### PB-08: Writeable Process (ค้นหาด้วย Hash)
```
FileSHA256 = "<hash>"
```

### PB-09: Defender Exclusion Tampering
```
CmdLine Contains "Add-MpPreference -ExclusionPath"
```
```
CmdLine Contains "Set-MpPreference -DisableRealtimeMonitoring"
```

### PB-10: bwswfcfg.exe
```
FileName = "bwswfcfg.exe"
```

---

## Process ที่น่าสงสัย

System Process รันจาก Path ผิดปกติ:
```
SrcProcName In ("svchost.exe","spoolsv.exe","csrss.exe","lsass.exe") AND NOT SrcProcImagePath Contains "System32"
```

PowerShell Encoded Command (มักเป็นมัลแวร์ซ่อนคำสั่ง):
```
TgtProcName = "powershell.exe" AND TgtProcCmdLine Contains "-enc"
```

PowerShell ดาวน์โหลดไฟล์:
```
TgtProcName = "powershell.exe" AND TgtProcCmdLine In Contains ("Invoke-WebRequest","wget","curl","DownloadFile","DownloadString","IEX")
```

certutil ดาวน์โหลดไฟล์ (Living off the Land):
```
TgtProcName = "certutil.exe" AND TgtProcCmdLine Contains "urlcache"
```

mshta เรียก URL:
```
TgtProcName = "mshta.exe" AND TgtProcCmdLine Contains "http"
```

rundll32 รัน DLL จาก Path ผิดปกติ:
```
TgtProcName = "rundll32.exe" AND TgtProcCmdLine In Contains ("Temp","AppData","ProgramData","Users")
```

regsvr32 Silent Registration:
```
TgtProcName = "regsvr32.exe" AND TgtProcCmdLine Contains "/s"
```

---

## Network Activity

Process ติดต่อ IP ภายนอก:
```
NetConnStatus = "SUCCESS" AND SrcProcName = "<process_name>"
```

Connection ไปยัง IP ที่ระบุ:
```
NetDestinationIp = "<ip_address>"
```

Connection ไปยัง Domain:
```
DNS Contains "<domain>"
```

Port ที่น่าสงสัย (Reverse Shell, Backdoor):
```
NetDestinationPort In (4444,5555,8080,8443,1234,9999)
```

---

## File Operations

ไฟล์ที่ถูกสร้างใน Path ต้องสงสัย:
```
EventType = "File Creation" AND FilePath In Contains ("Temp","AppData\\Local","ProgramData")
```

ไฟล์ .exe ใหม่:
```
EventType = "File Creation" AND FilePath Contains ".exe"
```

ไฟล์ใน Excluded Path (สำหรับ PB-09):
```
FilePath Contains "<excluded_path>" AND EventType = "File Creation"
```

สัญญาณ Ransomware — ไฟล์ถูกแก้ไขจำนวนมาก:
```
EventType = "File Modification" AND SrcProcName = "<process_name>" | count by FilePath
```

---

## Persistence

Scheduled Task ถูกสร้าง:
```
EventType = "Task Register" OR (TgtProcName = "schtasks.exe" AND TgtProcCmdLine Contains "/create")
```

Registry Run Key ถูกแก้ไข:
```
EventType = "Registry Value Modified" AND RegistryPath Contains "CurrentVersion\\Run"
```

Service ใหม่:
```
EventType = "Service Installed" OR (TgtProcName = "sc.exe" AND TgtProcCmdLine Contains "create")
```

---

## Credential Access

เข้าถึง LSASS (Credential Dump):
```
TgtProcName = "lsass.exe" AND EventType = "Open Remote Process Handle"
```

เครื่องมือ Credential Dumping:
```
SrcProcName In Contains ("mimikatz","procdump","comsvcs") OR TgtProcCmdLine Contains "sekurlsa"
```

---

## Lateral Movement

PsExec / Remote Execution:
```
SrcProcName In Contains ("psexec","psexesvc") OR (TgtProcName = "cmd.exe" AND TgtProcCmdLine Contains "\\\\")
```

WMI Remote Execution:
```
SrcProcName = "wmiprvse.exe" AND TgtProcName In ("cmd.exe","powershell.exe")
```

---

## Tips

| Tip | คำอธิบาย |
|:----|:---------|
| `Contains` | ค้นหาบางส่วนได้ ไม่ต้องพิมพ์ครบ |
| `In Contains (...)` | ค้นหาหลายค่าพร้อมกัน |
| `NOT` | กรอง False Positive ออก |
| `AND` / `OR` | รวมเงื่อนไข |
| **Group By** | ดูเป็น Summary เช่น Group by Endpoint |
| **Time Range** | เลือกช่วงเวลาให้ตรงกับ Alert |

---

<p align="center"><i>SOC Team — TW Site | อัปเดตล่าสุด: มีนาคม 2026</i></p>
