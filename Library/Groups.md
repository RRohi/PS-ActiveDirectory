# A library of AD group-related cmdlets and scripts.

## Get all AD Groups with a custom view.
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

## Query groups that have related info. This assumes that groups have something like 'Related resources: Ticket - IN-244' in the info attribute.
```powershell
Get-ADGroup -Filter 'info -like "*related*"' -Properties info | Select-Object Name, info | Format-List
```

## Get group memberships of a list of groups from a file.
```powershell
Get-Content -Path C:\temp\groups.txt -OutVariable Groups | Out-Null

ForEach ($Group in $Groups) {
    "Group: $Group ($((Get-ADGroupMember -Identity $Group).Count))"
    "Members: $((Get-ADGroupMember -Identity $Group | Select-Object -ExpandProperty SamAccountName) -join ', ')`r`n"
}
```

## Retrieve group memberships of all accounts ending with -adm.
```powershell
Get-ADUser -Filter 'SamAccountName -like "*-adm"' -OutVariable Admins | Out-Null

# Create an empty array.
$MembershipTable = @()

# Iterate found admin accounts.
ForEach ($Admin in $Admins) {
    ## Retrieve account principal group memberships.
    Get-ADPrincipalGroupMembership -Identity $Admin -Server dc-01.domain.tld -OutVariable Memberships | Out-Null

    ## Iterate found account principal group memberships.
    ForEach ($Membership in $Memberships) {
        ## Get AD Group Description.
        Get-ADGroup -Identity $Membership.SamAccountName -Properties Description -OutVariable Group | Out-Null

        ## Add membership data to the MembershipTable array.
        $MembershipTable += $Membership | Select-Object -Property `
            @{ Name = 'User SamAccountName';  Expression = { $Admin.SamAccountName } },
            @{ Name = 'Enabled';              Expression = { $Admin.Enabled } },
            @{ Name = 'Group SamAccountName'; Expression = { $Membership.SamAccountName } },
            @{ Name = 'Group DN';             Expression = { $Membership.distinguishedName } },
            @{ Name = 'Group Description';    Expression = { $Group.Description } }
    }
}

$MembershipTable
```

## Retrieve disabled accounts that have group memberships in admin groups.
```powershell
Get-ADUser -Filter 'Enabled -eq "False" -and SamAccountName -like "*.admin"' -OutVariable DotAdmins | Out-Null

$Report = ForEach ($Admin in $DotAdmins) {
    Get-ADPrincipalGroupMembership -Identity $Admin -Server dc-01.domain.tld | Select-Object -Property `
        @{ Name = 'Admin'; Expression = { $Admin.SamAccountName } },
        @{ Name = 'Group'; Expression = { $PSItem.SamAccountName } }
}

$Report
```

## Find empty groups.
```powershell
Get-ADGroup -Filter * -Properties Members -OutVariable Groups | Out-Null

ForEach ($Group in $Groups) {
    If ($Group.Members.Count -lt 1) {
        $Group.Name
    }
}
```
