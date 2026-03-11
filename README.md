# 📘 SOC Playbooks — TW Site

## ภาพรวม (Overview)
เอกสาร Playbook สำหรับทีม SOC ของ TW Site ใช้สำหรับตอบสนองต่อ Alert จาก **SentinelOne (EDR/XDR)**  
เนื้อหาเขียนในระดับ **Step-by-step** เพื่อให้ผู้ปฏิบัติงานที่มีความรู้พื้นฐานสามารถดำเนินการได้

## 📋 รายการ Playbook (Top 10 Alerts)

| # | Alert Name | ไฟล์ Playbook |
|---|-----------|--------------|
| 1 | dllhostex.exe detected as malicious | [PB-01](playbooks/PB-01_dllhostex.md) |
| 2 | spoolsv.exe detected as Malware | [PB-02](playbooks/PB-02_spoolsv.md) |
| 3 | Forbidden Spawn Execution detected | [PB-03](playbooks/PB-03_forbidden_spawn.md) |
| 4 | svchost.exe detected as Malware | [PB-04](playbooks/PB-04_svchost.md) |
| 5 | rufus-3.13.exe detected as Malware | [PB-05](playbooks/PB-05_rufus.md) |
| 6 | conres.dll detected as Malware | [PB-06](playbooks/PB-06_conres_dll.md) |
| 7 | OInstall_x64.exe detected as Ransomware | [PB-07](playbooks/PB-07_oinstall_ransomware.md) |
| 8 | Writeable Process Creation detected | [PB-08](playbooks/PB-08_writeable_process.md) |
| 9 | Add_Defender_Exclusion.cmd detected as Malware | [PB-09](playbooks/PB-09_add_defender_exclusion.md) |
| 10 | bwswfcfg.exe detected as General | [PB-10](playbooks/PB-10_bwswfcfg.md) |

## 🔰 วิธีใช้ Playbook
1. เมื่อได้รับ Alert จาก SentinelOne Console → ดูชื่อ Alert
2. เปิด Playbook ที่ตรงกับ Alert นั้น
3. ทำตามขั้นตอนทีละ Step
4. บันทึกผลลัพธ์ทุกขั้นตอนลง Incident Ticket

## 📌 ระดับความรุนแรง (Severity)
| ระดับ | ความหมาย | SLA ตอบสนอง |
|-------|---------|-----------|
| 🔴 Critical | ภัยรุนแรง เช่น Ransomware, Active C2 | ≤ 15 นาที |
| 🟠 High | Malware ยืนยัน, Lateral Movement | ≤ 30 นาที |
| 🟡 Medium | ไฟล์ต้องสงสัย, พฤติกรรมผิดปกติ | ≤ 1 ชั่วโมง |
| 🟢 Low / Informational | แจ้งเตือนทั่วไป, False Positive ที่เป็นไปได้ | ≤ 4 ชั่วโมง |

---
*สร้างโดย SOC Team — อัปเดตล่าสุด: มีนาคม 2026*
