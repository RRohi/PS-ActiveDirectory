# A library of AD accounts-related cmdlets and scripts.

## Query enabled accounts where password never expires.
```powershell
Get-ADUser -Filter { Enabled -eq "True" -and PasswordNeverExpires -eq "True" } -Properties PasswordNeverExpires | Select-Object SamAccountName | Sort-Object SamAccountName
```

## Get all AD groups in a custom view.
```powershell
Get-ADGroup -Filter * -Properties DisplayName, Description, ManagedBy, Members, whenCreated, whenChanged | Select-Object -Property `
	@{ Name = 'Name';         Expression = { $PSItem.Name } },
	@{ Name = 'DisplayName';  Expression = { $PSItem.DisplayName } },
	@{ Name = 'Description';  Expression = { $PSItem.Description } },
	@{ Name = 'Managed By';   Expression = { $PSItem.ManagedBy } },
	@{ Name = 'Member Count'; Expression = { $PSItem.Members.Count } },
	@{ Name = 'Category';     Expression = { $PSItem.GroupCategory } },
	@{ Name = 'Scope';        Expression = { $PSItem.GroupScope } },
	@{ Name = 'Created';      Expression = { $PSItem.whenCreated } },
	@{ Name = 'Changed';      Expression = { $PSItem.whenChanged } }
```

## Query enabled user accounts that have been inactive for 90 days.
```powershell
Search-ADAccount -AccountInactive -UsersOnly -TimeSpan (New-TimeSpan -Days 90) | Where-Object Enabled -eq $True | Sort-Object SamAccountName | Select-Object -Property `
    @{ Name = 'SamAccountName';          Expression = { $PSItem.SamAccountName } },
    @{ Name = 'Enabled';                 Expression = { $PSItem.Enabled } },
    @{ Name = 'Distinguished Name';      Expression = { $PSItem.DistinguishedName } },
    @{ Name = 'Last Logon Date';         Expression = { $PSItem.LastLogonDate } },
    @{ Name = 'Account Expiration Date'; Expression = { $PSItem.AccountExpirationDate } },
    @{ Name = 'OU';                      Expression = { $PSItem.DistinguishedName.Substring($PSItem.DistinguishedName.IndexOf("OU=")) } }
```

## Query disabled accounts that are not in disabled accounts OUs.
```powershell
Get-ADUser -Filter 'Enabled -eq "False"' | Where-Object DistinguishedName -notlike '*Disabled*' | Select-Object SamAccountName, Enabled, DistinguishedName
```

## Query expired accounts that are still enabled.
```powershell
Search-ADAccount -AccountExpired | Where-Object Enabled -eq $True | Sort-Object SamAccountName | Select-Object SamAccountName, DistinguishedName, LastLogonDate, AccountExpirationDate
```

## Get important account info.
```powershell
Get-ADUser -Identity user -Properties AccountExpirationDate, Created, Description, Enabled, LastBadPasswordAttempt, LastLogonDate, LockedOut, LogonWorkstations, Modified, PasswordExpired, PasswordLastSet, PasswordNeverExpires, PasswordNotRequired, SamAccountName, TrustedForDelegation
```

## Find enabled user accounts that don't have an email address that matches UPN.
```Powershell
Get-ADUser -Filter 'Enabled -eq "True"' -SearchBase 'OU=Users,DC=domain,DC=tld' -Properties proxyAddresses | Where-Object { "smtp:$($PSItem.UserPrincipalName)" -notin $PSItem.proxyAddresses } | Select-Object SamAccountName, proxyAddresses
```
