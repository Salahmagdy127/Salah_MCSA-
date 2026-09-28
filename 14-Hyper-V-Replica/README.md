# 14 — Hyper-V & Hyper-V Replica

**Machines:** PDC (primary) + CORE (replica server)
**Prerequisite:** Nested virtualization enabled in VMware → see [01 — Lab Setup](../01-Lab-Setup/README.md).

## Steps

1. **Hyper-V on PDC** — Add Roles and Features → ✅ **Hyper-V** →
   create a virtual switch (select the network adapter) → Install → restart.
2. **Hyper-V on CORE**:
   ```powershell
   Install-WindowsFeature Hyper-V -IncludeManagementTools -Restart
   ```
3. **Virtual Switch** (if not created in wizard) — Hyper-V Manager → **Virtual Switch Manager** →
   New → **External** → bind to the physical NIC → OK.
4. **Create a VM** on PDC — Hyper-V Manager → New → Virtual Machine (e.g. `Core VM`) →
   Generation, RAM, connect to the External switch, create VHDX, attach ISO.
5. **Enable CORE as Replica server** (on CORE — PowerShell, see below), or from PDC's Hyper-V Manager connected to CORE:
   Hyper-V Settings → **Replication Configuration** → ✅ Enable as Replica server →
   ✅ Kerberos (HTTP) port 80 → Allow replication from any authenticated server → storage path.
6. Open firewall rule on CORE: **Hyper-V Replica HTTP Listener (TCP-In)**.
7. **Enable replication** on PDC — right-click the VM → **Enable Replication** →
   Replica server: `CORE.final.local` → Kerberos (HTTP) → pick VHDs → replication frequency (e.g. 5 min) →
   Send initial copy **now** → Finish.

## PowerShell

```powershell
# On CORE (Replica server)
New-Item C:\Replica -ItemType Directory
Set-VMReplicationServer -ReplicationEnabled $true -AllowedAuthenticationType Kerberos `
    -ReplicationAllowedFromAnyServer $true -DefaultStorageLocation C:\Replica
Enable-NetFirewallRule -DisplayName "Hyper-V Replica HTTP Listener (TCP-In)"

# On PDC (Primary)
New-VMSwitch -Name External -NetAdapterName "Ethernet0" -AllowManagementOS $true
Enable-VMReplication -VMName "Core VM" -ReplicaServerName CORE.final.local -ReplicaServerPort 80 -AuthenticationType Kerberos
Start-VMInitialReplication -VMName "Core VM"
```

## Verify

```powershell
Get-VMReplication        # State: Replicating, Health: Normal
```
- CORE's Hyper-V Manager shows the replica VM (off).
- Right-click VM → Replication → **View Replication Health**.

---

<div dir="rtl">

## ليه عملنا كده؟

- **Hyper-V**: بيخليك تشغّل كذا سيرفر (VMs) على جهاز فيزيكال واحد، وده توفير في الأجهزة والكهربا والمساحة.
- **Virtual Switch أنواعه**:
  - **External** = الـ VM تطلع على الشبكة الحقيقية زي أي جهاز.
  - **Internal** = VM ↔ الهوست بس.
  - **Private** = الـ VMs مع بعض بس.
- **Hyper-V Replica**: نسخة من الـ VM بتتبعت للسيرفر التاني كل كام دقيقة. لو السيرفر الأساسي وقع، تشغّل النسخة من التاني (**Failover**). ده حل **Disaster Recovery** رخيص ومن غير shared storage.
- **Kerberos و Certificate**: الاتنين في نفس الدومين، فبنستخدم Kerberos (HTTP) لأنه أسهل. أما لو السيرفرات مش في دومين، فمحتاج Certificate (HTTPS).
- **الفايروال**: لازم تفتح الـ rule على سيرفر الـ Replica، وإلا الـ replication هيفشل. ودي أشهر غلطة.
- **ملاحظة للشغل الحقيقي**: مايكروسوفت **مش بتنصح** تحط Hyper-V على DC. إحنا عملنا كده هنا عشان اللاب بس.

</div>
