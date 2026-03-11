<h1 align="center">⚡ Quick Reference Card</h1>
<h4 align="center">สรุปทุก Alert ในหน้าเดียว — พิมพ์แปะข้างจอ ดูได้ทันทีตอน Alert เข้า</h4>

---

## 🔴 CRITICAL — ทำทันที! (SLA 15 นาที)

### OInstall_x64.exe — Ransomware ([PB-07](playbooks/PB-07_oinstall_ransomware.md))
```
⚡ ทำทันที:  Kill → Quarantine → Isolate เครื่อง
🔍 ตรวจ:     มีไฟล์ถูกเข้ารหัส (.encrypted, .locked)?
               ├── มี  → Escalate ทันที + Rollback (VSS)
               └── ไม่  → ตรวจ VT → Remediate → ลบไฟล์ (เป็น Pirated SW)
📞 แจ้ง:     มีเข้ารหัสไฟล์ → SOC Manager + IR Team ทันที
```

### Add_Defender_Exclusion.cmd — ปิด Defender ([PB-09](playbooks/PB-09_add_defender_exclusion.md))
```
⚡ ทำทันที:  Kill → Quarantine → Isolate เครื่อง
🔍 ตรวจ:     Script เพิ่ม Exclusion อะไร? → Path นั้นคือจุดซ่อน Malware!
               ├── ค้นหามัลแวร์ใน Excluded Path
               ├── ลบ Exclusion: Remove-MpPreference -ExclusionPath "..."
               └── เปิด Defender กลับ: Set-MpPreference -DisableRealtimeMonitoring $false
📞 แจ้ง:     พบ Ransomware ซ่อน → SOC Manager + IR Team
```

---

## 🟠 HIGH — ดำเนินการเร็ว (SLA 30 นาที)

### dllhostex.exe — Cryptojacking / Backdoor ([PB-01](playbooks/PB-01_dllhostex.md))
```
ข้อเท็จจริง:  ไฟล์จริงชื่อ dllhost.exe (ไม่มี "ex") → เจอ dllhostex = มัลแวร์แน่
🔍 ตรวจ:     VT Hash → Storyline → Network Connection (C2? Mining Pool?)
🛡️ ทำ:       Kill → Quarantine → Remediate → Block Hash ที่ Firewall
```

### spoolsv.exe — ปลอมชื่อ / PrintNightmare ([PB-02](playbooks/PB-02_spoolsv.md))
```
จุดตัดสิน:    ดู File Path ก่อน!
               ├── System32 → อาจ FP / PrintNightmare → ตรวจ DLL + Signature
               └── Path อื่น → มัลแวร์ปลอมชื่อ → Kill + Quarantine
🛡️ ทำ:       PrintNightmare → Patch + Disable Spooler บน Server
```

### Forbidden Spawn — Phishing / Macro ([PB-03](playbooks/PB-03_forbidden_spawn.md))
```
จุดตัดสิน:    Office สร้าง cmd/powershell = ไม่ปกติ!
🔍 ตรวจ:     Command Line สำคัญที่สุด! → -enc Base64? certutil? mshta?
               ├── มาจาก Email? → Block Sender + ลบ Email ทุก Mailbox (Symantec)
               └── Disable VBA Macro ผ่าน GPO
🛡️ ทำ:       Kill Parent+Child → Quarantine เอกสาร → Remediate
```

### svchost.exe — Process Injection / ปลอมชื่อ ([PB-04](playbooks/PB-04_svchost.md))
```
จุดตัดสิน:    ดู File Path ก่อน!
               ├── System32 + Parent=services.exe + CmdLine มี -k → ตรวจ Injection
               └── Path อื่น หรือ Parent ผิด → ปลอมแน่นอน!
🔍 ตรวจ:     DLL จากนอก System32? Memory สูง? → Process Injection
🛡️ ทำ:       Kill → Quarantine → ถ้า Injection = Reboot เคลียร์ Memory
```

### conres.dll — DLL Injection / Hijacking ([PB-06](playbooks/PB-06_conres_dll.md))
```
ข้อเท็จจริง:  conres.dll ไม่ใช่ DLL ของ Windows → True Positive สูง!
🔍 ตรวจ:     โหลดโดย rundll32/regsvr32/svchost? → น่าสงสัยมาก
               ├── Path: Temp/AppData/ProgramData = เสี่ยงสูง
               └── Storyline: มี C2? Credential Dump? → Escalate
🛡️ ทำ:       Kill Process ที่โหลด DLL → Quarantine → ลบ Persistence
```

### Writeable Process — Process Hollowing / Shellcode ([PB-08](playbooks/PB-08_writeable_process.md))
```
ข้อเท็จจริง:  Memory = Write+Execute ผิดปกติ
🔍 ตรวจ:     Hollowing = Suspended → WriteMemory → Resume
               ├── System Process ถูก Hollow → เสี่ยงสูงมาก
               ├── Unknown EXE → เสี่ยงสูง
               └── Dev Tool / Security SW → อาจ FP
🛡️ ทำ:       Kill → Quarantine → Reboot เคลียร์ Memory → Full Scan
```

---

## 🟡 MEDIUM (SLA 1 ชั่วโมง)

### rufus-3.13.exe — FP / Policy Violation ([PB-05](playbooks/PB-05_rufus.md))
```
จุดตัดสิน:    เทียบ Hash กับ rufus.ie (Official)
               ├── Hash ตรง + อนุญาต     → FP → Exclusion (ด้วย Hash)
               ├── Hash ตรง + ไม่อนุญาต  → Policy Violation → แจ้ง Manager
               └── Hash ไม่ตรง           → Trojanized → Kill + Remediate
```

---

## 🟢 LOW (SLA 4 ชั่วโมง)

### bwswfcfg.exe — General / Unknown ([PB-10](playbooks/PB-10_bwswfcfg.md))
```
ข้อเท็จจริง:  "General" = AI ตรวจพบพฤติกรรมน่าสงสัย (ยังไม่มี Signature)
🔍 ตรวจ:     VT Hash → 0 detection + Signed = น่าจะ FP / >10 = Malicious
               ├── ไม่มี Network/Child/Registry → FP → Exclusion
               └── มีพฤติกรรมผิดปกติ → ยกระดับ → Quarantine + Remediate
อย่าละเลย!   General อาจเป็น Zero-day ที่ยังไม่มี Signature
```

---

## Escalation — แจ้งใครเมื่อไหร่

| เจอสิ่งนี้ | แจ้งใคร |
|:---------|:--------|
| Ransomware / เข้ารหัสไฟล์ | SOC Manager + IR Team **ทันที** |
| C2 Communication ยืนยัน | SOC Manager + IR Team |
| Cobalt Strike / Meterpreter | SOC Manager + IR Team **ทันที** |
| DC / Server โดน | SOC Manager + IT Team **ทันที** |
| พบหลายเครื่อง (> 3) | SOC Manager |
| Phishing Campaign | SOC Manager + Symantec Email Team |
| วินิจฉัยไม่ได้ | Senior Analyst |

---

## SentinelOne Quick Actions

| ทำอะไร | ทำอย่างไร |
|:-------|:---------|
| Kill Process | Threat → Actions → "Kill" |
| Quarantine File | Threat → Actions → "Quarantine" |
| Isolate เครื่อง | Sentinels → "Disconnect from Network" |
| Remediate | Threat → Actions → "Remediate" |
| Rollback | Threat → Actions → "Rollback" |
| Full Scan | Sentinels → Actions → "Initiate Scan" |
| ปลด Isolation | Sentinels → "Reconnect to Network" |

---

<p align="center"><i>SOC Team — TW Site | อัปเดตล่าสุด: มีนาคม 2026</i></p>
