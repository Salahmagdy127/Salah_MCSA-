# 09 — IIS Web Server + DNS Round Robin (Load Balancing)

**Machines:** PDC + CORE

## Steps

1. **IIS on PDC** — Add Roles and Features → ✅ **Web Server (IIS)** → Install.
2. **IIS on CORE** (PowerShell):
   ```powershell
   Install-WindowsFeature Web-Server -IncludeManagementTools
   ```
3. (Optional, for testing) edit the default page on each server so you can tell them apart:
   `C:\inetpub\wwwroot\index.html` → "Server 1 - PDC" / "Server 2 - CORE".
4. **DNS records** — PDC → Tools → **DNS** → Forward Lookup Zones → `final.local` → **New Host (A)**:
   - Name `www` → IP `192.168.1.2` → Add Host
   - Name `www` → IP `192.168.1.3` → Add Host
5. Make sure **Round robin** is enabled: DNS server → Properties → **Advanced** → ✅ Enable round robin (default on).

## PowerShell

```powershell
Add-DnsServerResourceRecordA -ZoneName final.local -Name www -IPv4Address 192.168.1.2
Add-DnsServerResourceRecordA -ZoneName final.local -Name www -IPv4Address 192.168.1.3

# CORE test page
Set-Content C:\inetpub\wwwroot\index.html "Server 2 - CORE"
```

## Verify

```powershell
nslookup www.final.local      # run several times → order of IPs rotates
ipconfig /flushdns            # on client, then browse http://www.final.local again
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **IIS**: ده الـ Web Server بتاع مايكروسوفت، اللي بيشغّل مواقع أو web apps جوه الشركة (Intranet).
- **نفس الاسم `www` على 2 IPs**: الـ DNS بيرد بالـ IPs الاتنين بس **بيبدّل ترتيبهم كل مرة** (Round Robin)، فنص الناس يروحوا للسيرفر الأول ونصهم للتاني، وكده الضغط بيتوزع (**Load Balancing**).
- **عيب الـ Round Robin**: مش بيعرف لو سيرفر وقع، وهيفضل يبعت ناس للسيرفر الواقع. ده load balancing بسيط ومجاني. في الشغل الحقيقي بيستخدموا **NLB** أو Load Balancer حقيقي (F5 / HAProxy) بيعمل health check.
- **flushdns**: الكلاينت بيعمل cache للـ DNS، فلازم تمسحه عشان تشوف الـ IP التاني وانت بتجرّب.

</div>
