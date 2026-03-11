<h1 align="center">🛡️ SOC Playbooks — TW Site</h1>
<h3 align="center">Standard Operating Procedures for SentinelOne EDR/XDR Alerts</h3>

<p align="center">
  <img src="https://img.shields.io/badge/EDR-SentinelOne_XDR-6C2DC7?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Email-Symantec-FDB511?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Firewall-Fortigate-EE3124?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Firewall-PaloAlto-FA582D?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Playbooks-10-blue?style=for-the-badge" />
</p>

---

## 🏗️ TW Site — Tool Stack

| หมวด | เครื่องมือ | การใช้งานหลัก |
|:-----|:---------|:------------|
| 🖥️ **EDR / XDR** | SentinelOne XDR | ตรวจจับ + Response ที่ Endpoint, Deep Visibility, Remote Shell |
| 📧 **Email Security** | Symantec Email Security | กรอง Phishing, Block Sender, ลบ Email จาก Mailbox |
| 🔥 **Firewall** | Fortigate | Block C2 IP/Domain, Network Segmentation |
| 🔥 **Firewall** | Palo Alto Networks | Block C2 IP/Domain, Threat Prevention, URL Filtering |

---

## 📋 รายการ Playbook (Top 10 Alerts)

| # | Alert Name | Severity | Playbook |
|:-:|:-----------|:--------:|:--------:|
| 1 | `dllhostex.exe` detected as malicious | 🟠 High | [📖 PB-01](playbooks/PB-01_dllhostex.md) |
| 2 | `spoolsv.exe` detected as Malware | 🟠 High | [📖 PB-02](playbooks/PB-02_spoolsv.md) |
| 3 | Forbidden Spawn Execution detected | 🟠 High | [📖 PB-03](playbooks/PB-03_forbidden_spawn.md) |
| 4 | `svchost.exe` detected as Malware | 🟠 High | [📖 PB-04](playbooks/PB-04_svchost.md) |
| 5 | `rufus-3.13.exe` detected as Malware | 🟡 Medium | [📖 PB-05](playbooks/PB-05_rufus.md) |
| 6 | `conres.dll` detected as Malware | 🟠 High | [📖 PB-06](playbooks/PB-06_conres_dll.md) |
| 7 | `OInstall_x64.exe` detected as Ransomware | 🔴 Critical | [📖 PB-07](playbooks/PB-07_oinstall_ransomware.md) |
| 8 | Writeable Process Creation detected | 🟠 High | [📖 PB-08](playbooks/PB-08_writeable_process.md) |
| 9 | `Add_Defender_Exclusion.cmd` detected | 🔴 Critical | [📖 PB-09](playbooks/PB-09_add_defender_exclusion.md) |
| 10 | `bwswfcfg.exe` detected as General | 🟢 Low | [📖 PB-10](playbooks/PB-10_bwswfcfg.md) |

---

> [!TIP]
> 📌 **ต้องการดูสรุปทุก Alert ในหน้าเดียว?** → [⚡ Quick Reference Card](QUICK_REFERENCE.md) — พิมพ์แปะข้างจอได้!
>
> 🔍 **ต้องการ Query สำเร็จรูปสำหรับ Deep Visibility?** → [🔍 Query Cheatsheet](DEEP_VISIBILITY_QUERIES.md) — Copy-Paste ได้เลย!
>
> 📚 **ไม่เคยใช้ VirusTotal?** → [📚 VirusTotal Guide](VIRUSTOTAL_GUIDE.md) — คู่มือวิเคราะห์ Hash/IP/URL สำหรับมือใหม่

---

## 🔰 วิธีใช้ Playbook

```
1. ได้รับ Alert จาก SentinelOne Console
        ↓
2. ดูชื่อ Alert → เปิด Playbook ที่ตรงกัน
        ↓
3. ทำตามขั้นตอนทีละ Step
        ↓
4. บันทึกผลลัพธ์ทุกขั้นตอนลง Incident Ticket
```

---

## 📌 ระดับความรุนแรง (Severity) & SLA

| ระดับ | ความหมาย | ⏱️ SLA |
|:-----:|:---------|:------:|
| 🔴 **Critical** | Ransomware, Active C2, Defense Evasion | ≤ **15 นาที** |
| 🟠 **High** | Malware ยืนยัน, Lateral Movement | ≤ **30 นาที** |
| 🟡 **Medium** | ไฟล์ต้องสงสัย, พฤติกรรมผิดปกติ | ≤ **1 ชั่วโมง** |
| 🟢 **Low** | แจ้งเตือนทั่วไป, Possible FP | ≤ **4 ชั่วโมง** |

---

## 📐 โครงสร้างของแต่ละ Playbook

ทุก Playbook มีโครงสร้างเดียวกันเพื่อความสม่ำเสมอ:

| Section | เนื้อหา |
|:--------|:--------|
| 🎯 **Quick Reference** | สรุปข้อมูล Alert + Severity + SLA |
| 📊 **Flowchart** | Mermaid Diagram แสดง Decision Tree |
| 📋 **ขั้นตอนการตอบสนอง** | Step-by-step ละเอียด พร้อม Icon |
| 🚨 **Escalation Criteria** | เมื่อไหร่ต้องแจ้งหัวหน้า |
| 🛡️ **แนวทางป้องกัน** | Prevention Recommendations |

---

<p align="center"><i>📅 สร้างโดย SOC Team — อัปเดตล่าสุด: มีนาคม 2026</i></p>
