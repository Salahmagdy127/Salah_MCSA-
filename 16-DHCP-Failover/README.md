# 16 — DHCP Failover

**Machines:** PDC (existing DHCP) + CORE (partner)

## Steps

1. **Install DHCP on CORE** (from PDC Server Manager → Add Roles → server CORE → DHCP Server,
   or PowerShell below) → then **authorize** it in AD.
   > Don't create a scope on CORE — failover will copy it.
2. On PDC → **DHCP** console → right-click **IPv4** (or the scope) → **Configure Failover...**
3. Select the scope(s) → Next → **Partner server**: `CORE.final.local` (Add Server).
4. Failover settings:
   | Field | Value |
   |---|---|
   | Relationship name | auto (e.g. `pdc.final.local-core.final.local`) |
   | Mode | **Hot standby** (or Load balance) |
   | Role of partner | **Standby** |
   | Reserve addresses for standby | 5% (default) |
   | Maximum Client Lead Time | 1 hour (default) |
   | Enable Message Authentication | ✅ Shared secret: (type one) |
5. Next → Finish → all steps show **Successful**.

## PowerShell

```powershell
# On CORE
Install-WindowsFeature DHCP -IncludeManagementTools
Add-DhcpServerInDC -DnsName core.final.local -IPAddress 192.168.1.3

# On PDC
Add-DhcpServerv4Failover -Name "PDC-CORE" -PartnerServer core.final.local -ScopeId 192.168.1.0 `
    -ServerRole Active -ReservePercent 5 -SharedSecret "YourSecret"
# Load balance instead:  -LoadBalancePercent 50  (without -ServerRole)
```

## Verify

```powershell
Get-DhcpServerv4Failover
```
- CORE's DHCP console now shows the same scope.
- Test: stop DHCP on PDC (`Stop-Service DHCPServer`) → client `ipconfig /renew` → still gets IP from CORE.

---

<div dir="rtl">

## ليه عملنا كده؟

- **DHCP Failover**: لو سيرفر الـ DHCP وقع، الأجهزة الجديدة (أو اللي الـ lease بتاعها خلص) **مش هتاخد IP**، فمش هتعرف تدخل الشبكة خالص. الـ Failover بيخلي سيرفرين يتشاركوا نفس الـ scope ويعملوا sync للـ leases.
- **الوضعين**:
  - **Hot Standby** = سيرفر شغال (Active) والتاني مستني (Standby)، ويشتغل بس لو الأول وقع. ده بيتعمل غالباً لما السيرفر التاني يبقى في فرع تاني أو backup.
  - **Load Balance** = الاتنين شغالين مع بعض ومقسّمين الطلبات (50/50 مثلاً). ده بيتعمل لما الاتنين في نفس المكان.
- **ماتعملش Scope على الـ CORE بإيدك**: الـ Failover هو اللي بينسخ الـ scope والإعدادات. لو عملته يدوي، ممكن الاتنين يوزّعوا نفس الـ IPs، فيحصل **IP Conflict**.
- **Shared Secret**: باسورد بين السيرفرين عشان يتأكدوا إنهم بيكلموا بعض فعلاً ومحدش بيتدخّل.
- **Authorize**: زي ما شرحنا في موديول 04، أي DHCP في الدومين لازم يتعمله authorize.

</div>
