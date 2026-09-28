# Troubleshooting

| Problem | Cause | Fix |
|---|---|---|
| Client/Core **can't join domain** ("domain could not be contacted") | DNS on the machine isn't the DC | Set DNS to `192.168.1.2` → `ipconfig /flushdns` → `ping final.local` |
| Join fails with wrong credentials | Wrong username format | Use `FINAL\Administrator` or `administrator@final.local` |
| Windows 11 has no "Domain" option | Windows 11 **Home** | Use Pro / Enterprise |
| **Hyper-V install fails** ("processor doesn't have required virtualization") | Nested virtualization off in VMware | Power off VM → Settings → Processors → ✅ Virtualize Intel VT-x/EPT or AMD-V/RVI |
| `sconfig` slow / not showing | Services still loading after boot | Wait, or use PowerShell directly |
| GPO not applying | Computer/user still in default container, or not refreshed | Move object to the linked OU → `gpupdate /force` → check `gpresult /r` |
| Password policy on OU not working | Domain account password policy only works at **domain** level | Put it in **Default Domain Policy** (or use Fine-Grained Password Policy) |
| DHCP not giving IPs | Server not **authorized** / scope not active | DHCP console → right-click server → Authorize; activate scope |
| Hyper-V Replica fails to connect | Firewall on replica server | `Enable-NetFirewallRule -DisplayName "Hyper-V Replica HTTP Listener (TCP-In)"` |
| Can't ping a server | ICMP blocked by firewall | `Enable-NetFirewallRule -Name FPS-ICMP4-ERQ-In` |
| Round robin always returns same IP | Client DNS cache | `ipconfig /flushdns` |

<div dir="rtl">

## القاعدة الذهبية

**90% من مشاكل الدومين سببها الـ DNS.** أول حاجة تعملها في أي مشكلة: `ipconfig /all`، واتأكد إن الـ DNS بيشاور على الـ DC.

</div>
