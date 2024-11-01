# A library of different uncategorized cmdlets and scriptlets.

### Get an operating systems brakedown of the computers in AD
```powershell
Get-ADComputer -Filter 'Enabled -eq "True"' -Properties OperatingSystem | Group-Object OperatingSystem -NoElement | Sort-Object Count -Descending
```

### Query for failed logins for a specified account from a specified DC.
```powershell
$Start = (Get-Date).AddDays(-7)
Get-WinEvent -FilterHashtable @{ LogName = 'Security'; Id = 4625; StartTime = $Start; Data = 'username' } -ComputerName dc.fqdn
```

### Get OU tree.
```powershell
Get-ADOrganizationalUnit -Filter * -SearchScope Subtree | ForEach-Object {
    [PSCustomObject]@{
        Name = $PSItem.Name
        Level = ((($PSItem.DistinguishedName -replace "OU=$($PSItem.Name),",'') -split ',') -match 'OU=').Count 
        Path = $PSItem.DistinguishedName -replace "OU=$($PSItem.Name),",''
        GPO = $PSItem.LinkedGroupPolicyObjects -ireplace 'cn=','' -ireplace ',policies,system,DC=DOMAIN,DC=TLD','' -join ','
    }
} | Sort-Object Level | Export-Csv -Path C:\TEMP\datetime_OU_Tree.csv -Force -Encoding Unicode -Delimiter ';' -NoTypeInformation
```
## Create a new group Managed Service Account (gMSA).
```powershell
New-ADServiceAccount -Name 'SVC-ST$' -Description 'Service Account for SRV1 server high-privileged Scheduled Tasks.' -DisplayName 'Scheduled Task Service Account - High' -DNSHostName 'SRV1.DOMAIN.TLD' -Enabled $True -Path 'OU=Service Accounts,DC=DOMAIN,DC=TLD' -PrincipalsAllowedToRetrieveManagedPassword SRV1$ -SamAccountName 'SVC-ST'
```

## Test the gMSA on the SRV1 server and install it if the test succeeds.
```powershell
Test-ADServiceAccount -Identity 'SVC-ST$'
Install-ADServiceAccount -Identity 'SVC-ST$'
```

## Get OU from a DN.
```powershell
$ADRA = Get-ADUser -Identity radi.ator
$ADRA.DistinguishedName.Substring($ADRA.DistinguishedName.IndexOf('OU='))
```
