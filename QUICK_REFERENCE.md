<h1 align="center">⚡ Quick Reference Card — SOC Alert Response</h1>
<h4 align="center">📌 พิมพ์แปะข้างจอ — ดูเร็วเมื่อ Alert เข้า</h4>

---

## 🔴 CRITICAL — ต้องทำทันที! (SLA ≤ 15 นาที)

### 🚨 OInstall_x64.exe — Ransomware
```
⚡ IMMEDIATE: Kill → Quarantine → Isolate เครื่อง
🔍 CHECK:    มีไฟล์ถูกเข้ารหัส (.encrypted, .locked)?
   ├── ✅ มี  → 🔴 Escalate ทันที + Rollback (VSS)
   └── ❌ ไม่ → ตรวจ VT → Remediate → ลบไฟล์ทุกกรณี (Pirated SW)
📞 ESCALATE: มีเข้ารหัส → SOC Manager + IR Team ทันที
```

### 🚨 Add_Defender_Exclusion.cmd — Defense Evasion
```
⚡ IMMEDIATE: Kill → Quarantine → Isolate เครื่อง
🔍 CHECK:    Script เพิ่ม Exclusion Path อะไร? → Path นั้นคือจุดซ่อน Malware
   ├── ค้นหา Malware ใน Excluded Path
   ├── ลบ Exclusion: Remove-MpPreference -ExclusionPath "..."
   └── เปิด Real-time: Set-MpPreference -DisableRealtimeMonitoring $false
📞 ESCALATE: พบ Ransomware ซ่อน → SOC Manager + IR Team
```

---

## 🟠 HIGH — ดำเนินการเร็ว (SLA ≤ 30 นาที)

### 🔔 dllhostex.exe — Cryptojacking / Backdoor
```
⚠️ FACT:     ไฟล์จริงของ Windows ชื่อ dllhost.exe (ไม่มี "ex") → มัลแวร์แน่นอน!
🔍 CHECK:    VT Hash → Storyline → Network Connection (สงสัย C2/Mining)
🛡️ ACTION:   Kill → Quarantine → Remediate → Block Hash ที่ Firewall
```

### 🔔 spoolsv.exe — Masquerading / PrintNightmare
```
⚠️ KEY:      ดู File Path ก่อน!
   ├── Path = System32  → อาจ FP / PrintNightmare → ตรวจ DLL + Signature
   └── Path ≠ System32  → 🔴 มัลแวร์ปลอมชื่อ → Kill + Quarantine ทันที
🛡️ ACTION:   PrintNightmare → Patch + Disable Spooler (บนเครื่องที่ไม่ใช้พิมพ์)
```

### 🔔 Forbidden Spawn — Phishing / Macro
```
⚠️ KEY:      Office (Word/Excel/Outlook) สร้าง cmd/powershell = ไม่ปกติ!
🔍 CHECK:    Command Line สำคัญที่สุด! → -enc Base64? certutil? mshta http?
   ├── มาจาก Email? → แจ้ง Email Team → Block Sender + ลบจากทุก Mailbox
   └── Disable VBA Macro ผ่าน Group Policy
🛡️ ACTION:   Kill Parent+Child → Quarantine เอกสาร → Remediate
```

### 🔔 svchost.exe — Process Injection / Masquerading
```
⚠️ KEY:      ดู File Path ก่อน!
   ├── Path = System32 + Parent = services.exe + CmdLine มี -k  → ตรวจ Injection เพิ่ม
   └── Path ≠ System32 หรือ Parent ≠ services.exe → 🔴 ปลอมแน่นอน!
🔍 CHECK:    DLL จากนอก System32? Memory Usage สูง? → Process Injection
🛡️ ACTION:   Kill → Quarantine → Injection = Reboot เคลียร์ Memory
```

### 🔔 conres.dll — DLL Injection / Hijacking
```
⚠️ FACT:     conres.dll ไม่ใช่ DLL ของ Windows → True Positive สูง!
🔍 CHECK:    โหลดโดย rundll32/regsvr32/svchost? → 🔴 น่าสงสัยมาก
   ├── ดู Path: Temp/AppData/ProgramData = 🔴 สูง
   └── ดู Storyline: มี C2? Credential Dump? → Escalate
🛡️ ACTION:   Kill Process ที่โหลด DLL → Quarantine → ลบ Persistence
```

### 🔔 Writeable Process Creation — Process Hollowing / Shellcode
```
⚠️ KEY:      Memory = Write+Execute (WX) ผิดปกติ!
🔍 CHECK:    Process Hollowing = Suspended → WriteMemory → Resume
   ├── System Process ถูก Hollow → 🔴 สูงมาก
   ├── Unknown EXE → 🔴 สูง
   └── Dev Tool / Security SW → อาจ FP
🛡️ ACTION:   Kill → Quarantine → Reboot เคลียร์ Memory → Full Scan
```

---

## 🟡 MEDIUM (SLA ≤ 1 ชั่วโมง)

### 🔔 rufus-3.13.exe — FP / Trojanized Tool / Policy Violation
```
⚠️ KEY:      เทียบ Hash กับ rufus.ie (Official)
   ├── Hash ตรง + อนุญาต     → FP → สร้าง Exclusion (Hash)
   ├── Hash ตรง + ไม่อนุญาต  → Policy Violation → Quarantine + แจ้ง Manager
   └── Hash ไม่ตรง           → 🔴 Trojanized → Kill + Quarantine + Remediate
```

---

## 🟢 LOW (SLA ≤ 4 ชั่วโมง)

### 🔔 bwswfcfg.exe — General / Unknown
```
⚠️ KEY:      "General" = AI ตรวจพบพฤติกรรมน่าสงสัย (ยังไม่มี Signature)
🔍 CHECK:    VT Hash → 0 detection + Signed = FP / >10 detection = 🔴 Malicious
   ├── ไม่มี Network/Child/Registry → FP → Exclusion
   └── มีพฤติกรรมผิดปกติ → 🔴 ยกระดับ Severity → Quarantine + Remediate
⚠️ NOTE:     อย่าละเลย! General อาจเป็น Zero-day
```

---

## 📞 Escalation Quick Guide

| เจอสิ่งนี้ | แจ้งใคร |
|:---------|:--------|
| 🔴 Ransomware / เข้ารหัสไฟล์ | SOC Manager + IR Team **ทันที** |
| 🔴 C2 Communication ยืนยัน | SOC Manager + IR Team |
| 🔴 Cobalt Strike / Meterpreter | SOC Manager + IR Team **ทันที** |
| 🔴 DC / Server โดน | SOC Manager + IT Team **ทันที** |
| 🟠 พบหลายเครื่อง (> 3) | SOC Manager |
| 🟠 Phishing Campaign | SOC Manager + Email Team |
| 🟡 วินิจฉัยไม่ได้ | Senior Analyst |

---

## 🔧 SentinelOne Quick Actions

| Action | วิธีทำ |
|:-------|:------|
| **Kill Process** | Threat → Actions → "Kill" |
| **Quarantine File** | Threat → Actions → "Quarantine" |
| **Isolate เครื่อง** | Sentinels → Actions → "Disconnect from Network" |
| **Remediate** | Threat → Actions → "Remediate" |
| **Rollback** | Threat → Actions → "Rollback" |
| **Full Scan** | Sentinels → Actions → "Initiate Scan" |
| **ปลด Isolation** | Sentinels → Actions → "Reconnect to Network" |

---

<p align="center">
  <b>SOC Team — TW Site</b><br/>
  <i>อัปเดตล่าสุด: มีนาคม 2026</i>
</p>
