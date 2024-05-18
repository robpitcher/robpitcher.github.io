---
title: Azure Domain Controllers on Windows Server Core
date: '2024-05-17 11:46:00 -0400'
categories: [Azure]
tags: [azure, server core, windows, domain controllers]     # TAG names should always be lowercase
---

The following steps describe how to add a new Azure VM running Windows Server Core 2022 as an additional domain controller to an existing domain. The scope of this article covers the basic steps you would take after deploying the infrastructure. Architecture, infrastructure deployment, and domain administration are not covered here. For assistance with those items, please head over to [Microsoft Learn](https://learn.microsoft.com/en-us/).

### Prerequisites
- The server core VM must have line of sight to an existing domain controller
- Console or remote access to the server core VM
- A domain account with permission to promote the new server to a domain controller
- Ensure the NIC DNS configuration for the server core VM is pointing to existing domain controller(s).

### Prep data disk for OS
1. Console into the server core VM and choose option 15 to enter command line.
2. Run DISKPART commands. Be sure to change "X" in the lines below as needed for your environment
```console
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
3. Switch to powershell
```console
powershell
```

4. Install Windows Features
```powershell
Install-WindowsFeature -Name "AD-Domain-Services"
```

### Promote the server to a domain controller
> Update the parameter values below for your environment
{: .prompt-warning }
5. Specify the account credentials with permissions to perform domain controller promotion
```powershell
$credential = (Get-Credential "CORP\azureadmin")
```
6. Run the DC promotion command.
```powershell
Install-ADDSDomainController -Credential $credential -DomainName "corp.robpitcher.com" -InstallDns -DatabasePath "F:\Windows\NTDS" -LogPath "F:\Windows\NTDS" -SysvolPath "F:\Windows\SYSVOL" -SiteName "Default-First-Site-Name"
```