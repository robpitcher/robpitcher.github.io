---
title: Windows Server Core Domain Controllers
date: '2024-05-17 11:46:00 -0400'
categories: [WindowsServer]
tags: [core, server, windows, domain controllers]     # TAG names should always be lowercase
---

The following steps describe how to add a new Windows Server Core VM as an additional domain controller to an existing domain. This tutorial used Windows Server Core 2022 with an existing domain running on Windows Server 2016 Forest and Domain functional levels.

## Requirements
- The server core VM must have line of sight / connectivity to an existing domain controller
- Console / remote connectivity to the server core VM
- A domain account with permission to promote the new server to a domain controller

### Configure DNS
1. Ensure the NIC DNS configuration for the server core VM is pointing to existing domain controller(s).

### Prep data disk for OS (If you're installing ADDS on a separate disk)
2. Console into the server core VM and choose option 15 to enter command line.
3. Run DISKPART commands. Be sure to change "X" in the lines below as needed for your environment
```
diskpart
list disk
select disk X
clean
convert gpt
create partition primary
list partition
select partition X
format fs=ntfs label="ADDS" quick
assign letter=X
exit
```

### Install Windows Features
4. Switch to powershell
```
powershell
```

5. Install Windows Features
```powershell
Install-WindowsFeature -Name "AD-Domain-Services"
```

### Promote the server to a domain controller
6. Specify the account credentials with permissions to perform domain controller promotion
```powershell
$credential = (Get-Credential "CORP\azureadmin")
```
7. Run the DC promotion command. Be sure to update the parameter values for your environment
```powershell
Install-ADDSDomainController -Credential $credential -DomainName "corp.robpitcher.com" -InstallDns -DatabasePath "F:\Windows\NTDS" -LogPath "F:\Windows\NTDS" -SysvolPath "F:\Windows\SYSVOL" -SiteName "Default-First-Site-Name"
```