# PowerShell Cheat Sheet — Windows Server

## Network
```powershell
Get-NetAdapter
Get-NetIPConfiguration
New-NetIPAddress -InterfaceAlias "Ethernet0" -IPAddress 192.168.1.2 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet0" -ServerAddresses 192.168.1.2,192.168.1.3
Disable-NetAdapterBinding -Name "Ethernet0" -ComponentID ms_tcpip6
Test-NetConnection 192.168.1.2 -Port 389
Resolve-DnsName final.local
```

## System
```powershell
Rename-Computer -NewName PDC -Restart
Add-Computer -DomainName final.local -Credential FINAL\Administrator -Restart
Get-WindowsFeature | Where Installed
Install-WindowsFeature <Name> -IncludeManagementTools [-Restart]
Enable-PSRemoting -Force
Enter-PSSession -ComputerName CORE
```

## Common Feature Names
| Role | `Install-WindowsFeature` name |
|---|---|
| Active Directory | `AD-Domain-Services` |
| DNS | `DNS` |
| DHCP | `DHCP` |
| IIS | `Web-Server` |
| FSRM | `FS-Resource-Manager` |
| Print Server | `Print-Server` |
| Hyper-V | `Hyper-V` |
| Windows Server Backup | `Windows-Server-Backup` |

## Active Directory
```powershell
Install-ADDSForest -DomainName final.local -InstallDns
Install-ADDSDomainController -DomainName final.local -InstallDns -Credential (Get-Credential)
Get-ADDomainController -Filter *
New-ADOrganizationalUnit -Name HR -Path "DC=final,DC=local"
New-ADGroup -Name HR-Group -GroupScope Global -Path "OU=Groups,DC=final,DC=local"
New-ADUser -Name "User1" -SamAccountName user1 -Path "OU=HR,DC=final,DC=local" -AccountPassword (Read-Host -AsSecureString) -Enabled $true
Add-ADGroupMember HR-Group -Members user1
Unlock-ADAccount user1
Set-ADAccountPassword user1 -Reset -NewPassword (Read-Host -AsSecureString)
Search-ADAccount -LockedOut
```

## DHCP / DNS
```powershell
Get-DhcpServerv4Scope
Get-DhcpServerv4Lease -ScopeId 192.168.1.0
Get-DhcpServerv4Failover
Get-DnsServerResourceRecord -ZoneName final.local
Add-DnsServerResourceRecordA -ZoneName final.local -Name www -IPv4Address 192.168.1.2
```

## Health Checks
```powershell
dcdiag /q
repadmin /replsummary
repadmin /syncall /AdeP
gpupdate /force
gpresult /r
netdom query fsmo
```
