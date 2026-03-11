<h1 align="center">🛡️ SOC Playbooks — TW Site</h1>
<h3 align="center">คู่มือการตอบสนอง SentinelOne Alerts สำหรับทีม SOC</h3>

<p align="center">
  <img src="https://img.shields.io/badge/EDR-SentinelOne_XDR-6C2DC7?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Email-Symantec-FDB511?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Firewall-Fortigate-EE3124?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Firewall-PaloAlto-FA582D?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Playbooks-10-blue?style=for-the-badge" />
</p>

---

## เครื่องมือที่ใช้

| หมวด | เครื่องมือ | ใช้ทำอะไร |
|:-----|:---------|:---------|
| 🖥️ EDR / XDR | SentinelOne XDR | ตรวจจับ + ตอบสนอง, Deep Visibility, Remote Shell |
| 📧 Email Security | Symantec | กรอง Phishing, Block Sender, ลบ Email |
| 🔥 Firewall | Fortigate | Block C2, URL Filtering |
| 🔥 Firewall | Palo Alto Networks | Block C2, Threat Prevention |

---

## Playbook ทั้ง 10 ตัว

Playbook เรียงตาม Alert ที่พบบ่อยที่สุดของลูกค้า TW Site ครอบคลุมตั้งแต่ Ransomware ไปจนถึง Low-severity General Detection

| # | Alert | Severity | Link |
|:-:|:------|:--------:|:----:|
| 1 | `dllhostex.exe` detected as malicious | 🟠 High | [PB-01](playbooks/PB-01_dllhostex.md) |
| 2 | `spoolsv.exe` detected as Malware | 🟠 High | [PB-02](playbooks/PB-02_spoolsv.md) |
| 3 | Forbidden Spawn Execution detected | 🟠 High | [PB-03](playbooks/PB-03_forbidden_spawn.md) |
| 4 | `svchost.exe` detected as Malware | 🟠 High | [PB-04](playbooks/PB-04_svchost.md) |
| 5 | `rufus-3.13.exe` detected as Malware | 🟡 Medium | [PB-05](playbooks/PB-05_rufus.md) |
| 6 | `conres.dll` detected as Malware | 🟠 High | [PB-06](playbooks/PB-06_conres_dll.md) |
| 7 | `OInstall_x64.exe` detected as Ransomware | 🔴 Critical | [PB-07](playbooks/PB-07_oinstall_ransomware.md) |
| 8 | Writeable Process Creation detected | 🟠 High | [PB-08](playbooks/PB-08_writeable_process.md) |
| 9 | `Add_Defender_Exclusion.cmd` detected | 🔴 Critical | [PB-09](playbooks/PB-09_add_defender_exclusion.md) |
| 10 | `bwswfcfg.exe` detected as General | 🟢 Low | [PB-10](playbooks/PB-10_bwswfcfg.md) |

---

> [!TIP]
> **เอกสารเสริม:**
>
> ⚡ [Quick Reference Card](QUICK_REFERENCE.md) — สรุปทุก Alert ในหน้าเดียว พิมพ์แปะข้างจอได้
>
> 🔍 [Deep Visibility Queries](DEEP_VISIBILITY_QUERIES.md) — รวม Query สำเร็จรูป Copy-Paste ใช้ได้เลย
>
> 📚 [VirusTotal & TI Tools Guide](VIRUSTOTAL_GUIDE.md) — คู่มือตรวจ Hash, IP, URL สำหรับคนที่เพิ่งเริ่มใช้
>
> 🔐 [Exclusion Policy](EXCLUSION_POLICY.md) — แนวปฏิบัติการสร้าง Exclusion อย่างปลอดภัย

---

## เริ่มต้นใช้งาน

เวลาได้รับ Alert จาก SentinelOne Console:

1. ดูชื่อ Alert → เปิด Playbook ที่ตรงกับ Alert นั้น
2. ทำตามขั้นตอนทีละ Step ใน Playbook
3. บันทึกผลลัพธ์ทุกขั้นตอนลง Incident Ticket
4. ถ้าเข้าเงื่อนไข Escalation → แจ้งหัวหน้าตามที่ระบุ

ถ้าเจอ Alert ที่ไม่ตรงกับ Playbook ไหนเลย → ใช้แนวทางจาก **PB-10** (General) เป็นหลัก

---

## SLA ตาม Severity

| Severity | ตอบสนองภายใน | ตัวอย่าง |
|:--------:|:-----------:|:--------|
| 🔴 Critical | 15 นาที | Ransomware, มัลแวร์ปิด Defender |
| 🟠 High | 30 นาที | มัลแวร์ยืนยัน, Injection |
| 🟡 Medium | 1 ชั่วโมง | ไฟล์ต้องสงสัย, PUP |
| 🟢 Low | 4 ชั่วโมง | General Detection, อาจเป็น FP |

---

## โครงสร้างของ Playbook

ทุก Playbook ใช้รูปแบบเดียวกันเพื่อให้ Analyst ไม่ต้องปรับตัวเวลาเปลี่ยน Alert:

- **Quick Reference** — ดูปุ๊บรู้เลยว่า Alert นี้คืออะไร สำคัญแค่ไหน
- **Flowchart** — เห็นภาพรวมการตัดสินใจก่อนลงมือทำ
- **ขั้นตอนการตอบสนอง** — Step-by-step ละเอียด ทำตามได้เลย
- **Escalation** — เมื่อไหร่ต้องแจ้งหัวหน้า
- **การป้องกัน** — ทำอะไรได้บ้างเพื่อไม่ให้เจออีก

---

<p align="center">
  <i>SOC Team — TW Site | อัปเดตล่าสุด: มีนาคม 2026</i>
</p>
