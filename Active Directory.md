# Active Directory — PowerShell Commands

Needs the `ActiveDirectory` module (`RSAT-AD-PowerShell` feature).

## Users

```powershell
# Find a user and see key properties
Get-ADUser -Identity jbloggs -Properties * | Select-Object Name, EmailAddress, Enabled, LastLogonDate

# Search by name (partial match)
Get-ADUser -Filter "Name -like '*bloggs*'"

# Create a new user
New-ADUser -Name "Jo Bloggs" -SamAccountName jbloggs -UserPrincipalName jbloggs@domain.local `
    -Path "OU=Staff,DC=domain,DC=local" -AccountPassword (Read-Host -AsSecureString "Password") -Enabled $true

# Reset a password (force change at next logon)
Set-ADAccountPassword -Identity jbloggs -Reset -NewPassword (Read-Host -AsSecureString "New password")
Set-ADUser -Identity jbloggs -ChangePasswordAtLogon $true

# Unlock an account
Unlock-ADAccount -Identity jbloggs

# Disable / enable a user (leaver / returning staff)
Disable-ADAccount -Identity jbloggs
Enable-ADAccount -Identity jbloggs

# Find all locked-out accounts
Search-ADAccount -LockedOut

# Find accounts that haven't logged in for 90+ days (stale account check)
Search-ADAccount -AccountInactive -TimeSpan 90.00:00:00 -UsersOnly
```

## Groups

```powershell
# List members of a group
Get-ADGroupMember -Identity "Sales Team" | Select-Object Name, SamAccountName

# Add / remove a user from a group
Add-ADGroupMember -Identity "Sales Team" -Members jbloggs
Remove-ADGroupMember -Identity "Sales Team" -Members jbloggs -Confirm:$false

# Which groups is a user actually in
Get-ADPrincipalGroupMembership -Identity jbloggs | Select-Object Name
```

## Computers

```powershell
# List all computer objects, last logon date (find stale/dead machines)
Get-ADComputer -Filter * -Properties LastLogonDate | Select-Object Name, LastLogonDate | Sort-Object LastLogonDate

# Find a specific computer
Get-ADComputer -Identity "DESKTOP-01" -Properties IPv4Address, OperatingSystem
```

## Organizational Units

```powershell
# List all OUs
Get-ADOrganizationalUnit -Filter * | Select-Object Name, DistinguishedName

# Move a user to a different OU
Move-ADObject -Identity (Get-ADUser jbloggs).DistinguishedName -TargetPath "OU=Finance,DC=domain,DC=local"
```

## Domain / Forest health

```powershell
# Current FSMO role holders
netdom query fsmo

# Full DC health check
dcdiag /v

# Replication summary across all DCs
repadmin /replsummary

# List all domain controllers
Get-ADDomainController -Filter *
```
