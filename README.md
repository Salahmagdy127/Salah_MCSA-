# Windows Server Administration — Full Project (MCSA Lab)

Hands-on lab notes for a full Windows Server project: build a domain from zero and add
the core services a small company needs.

> Source: *Full Windows Server Project — Windows Server Administration* by **Mohamed Zohdy** (YouTube, ~1h23m).
> Each module has: **Steps** (English, short) → **PowerShell** (same thing from CLI) → **Verify** → **ليه عملنا كده؟** (Arabic).

---

## Lab Topology

```
                     Gateway 192.168.1.1
                            |
        ---------------------------------------------  192.168.1.0/24
        |                      |                    |
   [ PDC ]  GUI           [ CORE ] Server Core   [ CLIENT ] Windows 11
   192.168.1.2            192.168.1.3            DHCP (192.168.1.40 - .230)
   - AD DS (first DC)     - AD DS (additional DC)
   - DNS                  - DNS
   - DHCP (primary)       - DHCP (failover partner)
   - IIS                  - IIS
   - File Server + FSRM   - Hyper-V (Replica server)
   - Print Server         - Backup target share
   - Hyper-V (primary)
   - Windows Server Backup
```

| Item | Value |
|---|---|
| Domain | `final.local` |
| PDC | `PDC` — 192.168.1.2 /24, GW .1, DNS 127.0.0.1 → later .3 |
| ADC | `CORE` — 192.168.1.3 /24, GW .1, DNS 192.168.1.2 |
| DHCP scope | 192.168.1.40 – 192.168.1.230, exclude .80 – .85, lease 10 days |
| OUs | HR, Sales, Dev, IT, Groups |
| Groups | HR-Group, Sales-Group, Dev-Group, IT-Group |
| Hypervisor | VMware Workstation (nested virtualization for Hyper-V) |

---

## Modules (in order)

| # | Module | Machine |
|---|---|---|
| 01 | [Lab Setup (VMware)](01-Lab-Setup/README.md) | Host |
| 02 | [PDC — IP, Rename, AD DS, Promote](02-PDC-Domain-Controller/README.md) | PDC |
| 03 | [Server Core — sconfig & Join Domain](03-Server-Core-Setup/README.md) | CORE |
| 04 | [DHCP Server & Scope](04-DHCP/README.md) | PDC |
| 05 | [OUs, Users & Groups](05-AD-Users-Groups/README.md) | PDC |
| 06 | [Join Windows 11 Client](06-Client-Join/README.md) | CLIENT |
| 07 | [Group Policy (GPO)](07-Group-Policy/README.md) | PDC |
| 08 | [Additional Domain Controller](08-Additional-DC/README.md) | CORE |
| 09 | [IIS + DNS Round Robin](09-IIS-DNS-Round-Robin/README.md) | PDC + CORE |
| 10 | [Shared Folders & NTFS Permissions](10-File-Shares-Permissions/README.md) | PDC |
| 11 | [FSRM — File Screening & Quota](11-FSRM/README.md) | PDC |
| 12 | [Print Server](12-Print-Server/README.md) | PDC |
| 13 | [AD Recycle Bin](13-AD-Recycle-Bin/README.md) | PDC |
| 14 | [Hyper-V & Replica](14-Hyper-V-Replica/README.md) | PDC + CORE |
| 15 | [Windows Server Backup](15-Windows-Server-Backup/README.md) | PDC |
| 16 | [DHCP Failover](16-DHCP-Failover/README.md) | PDC + CORE |

Extras: [PowerShell Cheat Sheet](PowerShell-Cheatsheet.md) · [Troubleshooting](Troubleshooting.md) · [Project Checklist](CHECKLIST.md)

---

<div dir="rtl">

## ⚠️ ملاحظة مهمة

المحتوى ده اتجمع من الفيديو بمساعدة AI (Gemini)، وبعدين اتظبط واتكمل بأوامر PowerShell وقيم مظبوطة.
حاجات لازم تراجعها من الفيديو نفسه:

- **الباسورد** اللي استخدمها في اللاب (Gemini كتب `P@ssw0rd`، مش متأكد).
- **ترتيب الخطوات**: الترتيب هنا منطقي (زي ما المفروض يتعمل)، وممكن يكون مختلف شوية عن ترتيب الفيديو.
- **وضع الـ DHCP Failover** (Hot Standby ولا Load Balance).
- تفاصيل الطابعة (نوع الـ driver اللي استخدمه).

</div>
