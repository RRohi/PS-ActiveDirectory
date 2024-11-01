# Installation of an Active Directory (AD) forest and and a secondary Domain Controller (DC).

The following samples are a basic/minimal way of how to deploy the forest and a DC. To see a complete list of parameters for each cmdlet, use ```Get-Help``` or see Microsoft Docs.

## Install AD role binaries.

The following cmdlet will install the required AD binaries along with all the additional tools and features.

```powershell
Install-WindowsFeature -Name AD-Domain-Services -IncludeAllSubFeature -IncludeManagementTools
```

## Install AD Forest.

This cmdlet will deploy the first DC as domain forest.

The **WinThreshold** value in **ForestMode** or **DomainMode** parameters indicates the latest domain Forest Functional Level (FFL) and Domain Functional Level (DFL).

**DomainName** is expected to be in Fully Qualified Distinguished Name (FQDN) format: **mycorp.com**

**DomainNetbiosName** should be a single label version of the FQDN you provided in **DomainName**: **mycorp**

If you have additional drives on the DC, you could store database, logs and SYSVOL on other drives.

```powershell
Install-ADDSForest -ForestMode 'WinThreshold' -DomainName 'domainname.tld' -DomainNetbiosName 'domainname' -DomainMode 'WinThreshold' -DatabasePath 'C:\Windows\NTDS' -LogPath 'C:\Windows\NTDS' -SysvolPath 'C:\Windows\SYSVOL' -InstallDns:$True -SafeModeAdministratorPassword (Read-Host -Prompt 'Enter DSRM Password' -AsSecureString) -Force:$True
```

## Install a DC.

The following cmdlet assumes you're going to deploy the new DC into the default site.

Decide whether this particular DC needs to be a **Global Catalog** server. The Global Catalog server owns the **Infrastructure master** Flexible Single Master Operation (FSMO) role and there can be only one Infrastructure master in a domain. In this example, the role is being left to the first DC.

```powershell
Install-ADDSDomainController -DomainName 'domainname.tld' -Sitename 'Default-First-Site-Name' -NoGlobalCatalog:$True -InstallDns:$True -DatabasePath 'C:\Windows\NTDS' -LogPath 'C:\Windows\NTDS' -SysvolPath 'C:\Windows\SYSVOL' -SafeModeAdministratorPassword (Read-Host -Prompt 'Enter DSRM Password' -AsSecureString) -Credential (Get-Credential -Message 'Enter Domain Administrator Credentials') -Force:$True
```
