# 02 — PDC: Static IP, Rename, AD DS, Promote to DC

**Machine:** PDC (GUI)

## Steps

1. **Static IP** — `ncpa.cpl` → adapter → Properties → **IPv4**:
   - IP `192.168.1.2` · Mask `255.255.255.0` · Gateway `192.168.1.1`
   - Preferred DNS `127.0.0.1` (itself)
2. **Disable IPv6** — same Properties window → uncheck **Internet Protocol Version 6 (TCP/IPv6)**.
3. **Rename** — `sysdm.cpl` → Change → Computer name `PDC` → **Restart**.
4. **Install AD DS** — Server Manager → Manage → **Add Roles and Features** →
   Role-based → select server → ✅ **Active Directory Domain Services** → Add Features → Install.
5. **Promote** — click the ⚠️ flag in Server Manager → **Promote this server to a domain controller**:
   - **Add a new forest** → Root domain name: `final.local`
   - DNS Server ✅, Global Catalog ✅
   - Set **DSRM password**
   - NetBIOS name: `FINAL` (auto) → Next → Install → server restarts automatically.
6. Log in as `FINAL\Administrator`.

## PowerShell

```powershell
# IP + DNS
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.1.2 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 127.0.0.1

# Disable IPv6 on the adapter
Disable-NetAdapterBinding -Name "Ethernet0" -ComponentID ms_tcpip6

# Rename
Rename-Computer -NewName PDC -Restart

# AD DS + new forest
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
Install-ADDSForest -DomainName final.local -DomainNetbiosName FINAL -InstallDns
```

> `Get-NetAdapter` to see your real adapter name.

## Verify

```powershell
Get-ADDomain            # shows final.local
Get-Service NTDS, DNS   # both Running
nslookup final.local
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **Static IP**: أي سيرفر بيقدّم خدمة (DC / DNS / DHCP) لازم الـ IP بتاعه ثابت، لأن كل الأجهزة هتدوّر عليه بالـ IP ده. لو اتغير، الدومين كله يقع.
- **DNS = نفسه (127.0.0.1)**: الـ DC هو نفسه الـ DNS server بتاع الدومين. الأكتيف دايركتوري **قايم على الـ DNS**، والأجهزة بتلاقي الـ DC عن طريق SRV records في الـ DNS.
- **إلغاء IPv6**: أغلب الشبكات مش بتستخدمه، وسيبانه ممكن يعمل بطء أو لخبطة في الـ name resolution. (ملاحظة: مايكروسوفت رسمياً مش بتنصح تقفله، بس في اللاب ده عادي.)
- **Rename قبل الـ Promote**: اسم الـ DC بيتسجل في الأكتيف دايركتوري والـ DNS، وتغييره بعد ما يبقى DC وجع دماغ. فسمّي الجهاز الأول وبعدين رقّيه.
- **الفرق بين Domain و Domain Controller**:
  - **Domain** = الكيان المنطقي (`final.local`) اللي بيجمع اليوزرز والأجهزة.
  - **Domain Controller** = السيرفر الفعلي اللي شايل قاعدة البيانات `NTDS.dit` وبيعمل الـ authentication.
- **New Forest**: ده أول دومين خالص، فلازم نعمل Forest جديد، والدومين ده يبقى الـ root.
- **DSRM Password**: باسورد وضع الصيانة (Directory Services Restore Mode) اللي بتدخله لو قاعدة بيانات الـ AD باظت وعايز تعمل restore. **احفظه كويس.**

</div>
