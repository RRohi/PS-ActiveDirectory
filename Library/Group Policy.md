# A library of AD Group Policy-related cmdlets and scripts.

## Query Group Policies linked to OUs.
```powershell
(Get-ADOrganizationalUnit -Filter * | Get-GPInheritance).GpoLinks | Select-Object -Property Target, DisplayName, Enabled, Enforced, Order
```

## Add delegation to GPO.
### Delegate full privileges to all GPOs for full-group group.
```powershell
Get-GPO -All | ForEach-Object { Set-GPPermission -Guid $PSItem.Id -PermissionLevel GpoEditDeleteModifySecurity -TargetName 'test-group' -TargetType Group }
```
### Delegate read privileges to all GPOs that don't have admin in their names.
```powershell
Get-GPO -All | Where-Object DisplayName -INotMatch 'admin' | ForEach-Object { Set-GPPermission -Guid $PSItem.Id -PermissionLevel GpoRead -TargetName 'read-group' -TargetType Group }
```

## Disable User/Computer Settings in GPOs that have no User/Computer settings.
```powershell
Get-GPO -All | Where-Object { $PSItem.User.DSVersion -eq 0 -and $PSItem.Computer.DSVersion -gt 0 } | ForEach-Object { $PSItem.GpoStatus = 'UserSettingsDisabled' }
Get-GPO -All | Where-Object { $PSItem.User.DSVersion -gt 0 -and $PSItem.Computer.DSVersion -eq 0 } | ForEach-Object { $PSItem.GpoStatus = 'ComputerSettingsDisabled' }
```

## Generate a GPO summary.
```powershell
Get-GPO -All | Select-Object -Property `
    @{ Name = 'GPO Name';         Expression = { $PSItem.DisplayName } },
    @{ Name = 'ID';               Expression = { $PSItem.ID } },
    @{ Name = 'Settings Status';  Expression = { $PSItem.GpoStatus } },
    @{ Name = 'User Version';     Expression = { $PSItem.User.DSVersion } },
    @{ Name = 'Computer Version'; Expression = { $PSItem.Computer.DSVersion } },
    @{ Name = 'Description';      Expression = {
            If ($null -eq $PSItem.Description) {
                'N/A'
            }
            Else {
                $PSItem.Description
            }
        }
    },
    @{ Name = 'Owner';            Expression = { $PSItem.Owner } },
    @{ Name = 'WMI Filter';       Expression = {
            If ($null -eq $PSItem.WmiFilter.Name) {
                'N/A'
            }
            Else {
                $PSItem.WmiFilter.Name
            }
        }
    },
    @{ Name = 'Delegations';      Expression = {
            Get-GPPermission -Guid $GPO.Id -All -OutVariable GPOACL | Out-Null

            $ACLHT = @{}

            ForEach ($ACE in $GPOACL) {
                $ACLHT.Add($ACE.Trustee.Name, $ACE.Permission)
            }

            $ACLHT
        }
    }
```
