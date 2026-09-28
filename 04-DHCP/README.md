# 04 — DHCP Server & Scope

**Machine:** PDC

## Steps

1. Server Manager → Add Roles and Features → ✅ **DHCP Server** → Install.
2. Click the ⚠️ flag → **Complete DHCP configuration** → Commit (authorizes the server in AD).
3. Tools → **DHCP** → expand `pdc.final.local` → right-click **IPv4** → **New Scope**:
   | Field | Value |
   |---|---|
   | Name | `Scope 192.168.1.x` |
   | Start / End IP | `192.168.1.40` → `192.168.1.230` |
   | Mask | `255.255.255.0` (/24) |
   | Exclusions | `192.168.1.80` → `192.168.1.85` |
   | Lease duration | **10 days** |
   | Configure options now | Yes |
   | Router (Gateway) | `192.168.1.1` |
   | DNS | `192.168.1.2` (parent domain `final.local`) |
   | WINS | skip |
   | Activate scope | Yes |

## PowerShell

```powershell
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName pdc.final.local -IPAddress 192.168.1.2
Add-DhcpServerSecurityGroup ; Restart-Service DHCPServer

Add-DhcpServerv4Scope -Name "Scope 192.168.1.x" -StartRange 192.168.1.40 -EndRange 192.168.1.230 `
    -SubnetMask 255.255.255.0 -LeaseDuration 10.00:00:00 -State Active
Add-DhcpServerv4ExclusionRange -ScopeId 192.168.1.0 -StartRange 192.168.1.80 -EndRange 192.168.1.85
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -Router 192.168.1.1 -DnsServer 192.168.1.2 -DnsDomain final.local
```

## Verify

- DHCP console: scope is **Active**, green ✔ on IPv4.
- On client: `ipconfig /release` → `ipconfig /renew` → gets IP between .40 and .230.
- DHCP console → **Address Leases** shows the client.

---

<div dir="rtl">

## ليه عملنا كده؟

- **DHCP**: بدل ما تروح لكل جهاز تكتبله IP بإيدك، السيرفر بيوزّع IP وGateway وDNS أوتوماتيك على كل الأجهزة.
- **Authorize in AD**: عشان مايجيش حد يشغّل DHCP server "مضروب" (Rogue) على الشبكة ويوزّع IPs غلط. في الدومين، الـ DHCP لازم يبقى متصرّح بيه من الـ AD عشان يشتغل.
- **الرينج يبدأ من 40**: سايبين الـ IPs من 1 لـ 39 للأجهزة الـ Static (راوتر، سيرفرات، طابعات...) عشان مايحصلش IP conflict.
- **Exclusion 80–85**: IPs جوه الرينج بس مش عايزين الـ DHCP يوزعها، زي إنها تكون محجوزة لأجهزة معينة (طابعات / كاميرات).
- **Lease 10 أيام**: المدة اللي الجهاز يفضل ماسك فيها الـ IP قبل ما يجدد. مدة طويلة تنفع للشبكة الثابتة (أجهزة مكتب)، ومدة قصيرة تنفع لما يبقى فيه أجهزة كتير داخلة وخارجة (Wi-Fi ضيوف).
- **DNS option = الـ DC**: عشان الكلاينت يلاقي الدومين ويعرف يعمل Join.

</div>
