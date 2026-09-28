# Project Checklist

Use this to rebuild the lab (or a real small-company setup) from scratch.

### Foundation
- [ ] VMs created, same network, nested virtualization on PDC + CORE
- [ ] PDC: static IP `192.168.1.2`, IPv6 off, renamed `PDC`
- [ ] AD DS installed, new forest `final.local`
- [ ] CORE: renamed, static IP `192.168.1.3`, DNS → PDC, ping + remote mgmt enabled, joined domain

### Services
- [ ] DHCP installed + authorized, scope .40–.230, exclude .80–.85, lease 10 days, options set
- [ ] OUs: HR, Sales, Dev, IT, Groups
- [ ] Groups created and users added
- [ ] Windows 11 client joined, moved to correct OU
- [ ] GPO: password + lockout policy (Default Domain Policy)
- [ ] GPO: RDP enabled + firewall
- [ ] GPO: block Task Manager, Control Panel, USB
- [ ] GPO: drive maps
- [ ] CORE promoted to Additional DC, DNS cross-pointed, `dcdiag` clean
- [ ] IIS on both servers + `www` A records on both IPs (round robin)
- [ ] Shares HR / Dev / Public with NTFS + Deny Delete
- [ ] FSRM: block audio/video/exe on HR + Dev, 2 GB quota on Public
- [ ] Print server: B&W only, 9 AM–4 PM, deployed via GPO
- [ ] AD Recycle Bin enabled + tested
- [ ] Hyper-V on both, external switch, VM replicating PDC → CORE
- [ ] Windows Server Backup: full server, daily 11 PM → `\\CORE\Backup`
- [ ] DHCP failover PDC ↔ CORE, tested by stopping DHCP on PDC
