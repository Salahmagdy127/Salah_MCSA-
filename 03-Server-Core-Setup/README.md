# 03 — Server Core: sconfig, IP, Join Domain

**Machine:** CORE (no GUI — everything from `sconfig` / PowerShell)

## Steps

1. Log in → type `sconfig` (opens automatically on newer versions).
2. **Option 2 — Computer name** → `CORE` → restart when asked.
3. **Option 8 — Network settings** → pick adapter →
   - Set address → **Static** → IP `192.168.1.3` · Mask `255.255.255.0` · Gateway `192.168.1.1`
   - Set DNS → `192.168.1.2` (the PDC)
4. **Option 4 — Remote management** → Enable, and **enable ping response**.
5. **Option 1 — Domain/workgroup** → **D** (domain) → `final.local` →
   enter `FINAL\Administrator` + password → restart.

## PowerShell

```powershell
Rename-Computer -NewName CORE -Restart

New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.1.3 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.1.2

# Allow ping (ICMPv4 echo)
Enable-NetFirewallRule -Name FPS-ICMP4-ERQ-In

# Remote management (WinRM)
Enable-PSRemoting -Force

Add-Computer -DomainName final.local -Credential FINAL\Administrator -Restart
```

## Verify

```powershell
ping 192.168.1.2
Test-ComputerSecureChannel     # True = joined correctly
(Get-CimInstance Win32_ComputerSystem).Domain
```

---

<div dir="rtl">

## ليه عملنا كده؟

- **sconfig**: واجهة نصية بالأرقام، فيها أغلب الإعدادات الأساسية للـ Core من غير ما تحفظ أوامر.
- **DNS = 192.168.1.2**: لازم الجهاز يسأل الـ DNS بتاع الدومين (الـ PDC) عشان يلاقي `final.local`. لو سبته على DNS بره زي 8.8.8.8، **الـ Join هيفشل**، ودي أشهر غلطة.
- **Ping**: الفايروال بيقفل الـ ping افتراضياً، وإحنا فتحناه عشان نعمل تست للاتصال بسهولة.
- **Remote Management**: عشان نقدر ندير الـ Core من Server Manager اللي على الـ PDC، لأن الـ Core مفيهوش GUI، فبنديره عن بُعد.
- **Join Domain**: عشان يبقى عضو في الدومين، وبعدها نقدر نرقّيه لـ Additional DC (موديول 08).

</div>
