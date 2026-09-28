# 08 — Promote CORE to Additional Domain Controller (ADC)

**Machines:** PDC (managing) + CORE

## Steps (from PDC Server Manager)

1. Server Manager → **Manage → Add Servers** → Active Directory → Find Now → add `CORE`.
2. Add Roles and Features → select server **CORE** → ✅ **Active Directory Domain Services** → Install.
3. ⚠️ flag → **Promote this server to a domain controller**:
   - **Add a domain controller to an existing domain** → `final.local`
   - Credentials: `FINAL\Administrator`
   - DNS Server ✅, Global Catalog ✅
   - DSRM password
   - Replicate from: **PDC.final.local** → Install → CORE restarts.
4. Fix DNS on both DCs (point to each other first, itself second):
   - PDC → Preferred `192.168.1.3`, Alternate `127.0.0.1`
   - CORE → Preferred `192.168.1.2`, Alternate `127.0.0.1`

## PowerShell (on CORE)

```powershell
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSDomainController -DomainName final.local -InstallDns -Credential (Get-Credential FINAL\Administrator)

Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.1.2,127.0.0.1
```

## Verify

```powershell
Get-ADDomainController -Filter * | Select Name, IPv4Address
repadmin /replsummary        # 0 fails
dcdiag /q                    # no errors = healthy
```
- Create a test user on PDC → it appears on CORE after replication.

---

<div dir="rtl">

## ليه عملنا كده؟

- **Additional DC**: لو الـ PDC وقع، **مفيش حد هيعرف يعمل login** والشبكة كلها تقف. التاني نسخة كاملة من قاعدة بيانات الـ AD، فبيكمّل شغل مكانه (**High Availability / Redundancy**).
- **Replication**: أي تعديل على أي DC (يوزر جديد، باسورد اتغير) بيتنسخ للتاني أوتوماتيك.
- **Load**: الاتنين بيعملوا authentication، فالحِمل بيتوزع عليهم.
- **DNS + GC على الـ ADC**: عشان لو الأول وقع، الـ DNS كمان يفضل شغال، لأن من غير DNS مفيش دومين.
- **كل DC يشاور على التاني في الـ DNS**: عشان مايبقاش الـ DC معتمد على نفسه بس. وده بيساعد الـ replication يشتغل صح بعد الـ restart.
- **على فكرة**: اسم "PDC" هنا مجرد اسم. في الويندوز الحديث كل الـ DCs متساويين (multi-master)، والفرق الوحيد في الـ **FSMO Roles** اللي بتبقى على أول DC.

</div>
