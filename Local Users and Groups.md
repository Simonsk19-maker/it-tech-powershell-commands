# Local Users and Groups — PowerShell Commands

For a machine's own local accounts (not domain accounts — see
[[Active Directory]] for those). Useful for standalone PCs, workgroup
machines, or local admin accounts on a domain-joined machine.

## Users

```powershell
# List all local users
Get-LocalUser

# Details on one user
Get-LocalUser -Name "Administrator"

# Create a new local user
$password = Read-Host -AsSecureString "Password"
New-LocalUser -Name "ITSupport" -Password $password -FullName "IT Support" -Description "Local support account"

# Set/reset a local user's password
Set-LocalUser -Name "ITSupport" -Password (Read-Host -AsSecureString "New password")

# Enable / disable a local account
Enable-LocalUser -Name "ITSupport"
Disable-LocalUser -Name "ITSupport"

# Rename a local user
Rename-LocalUser -Name "ITSupport" -NewName "SupportAdmin"

# Delete a local user
Remove-LocalUser -Name "OldAccount"

# Force password change at next logon / prevent expiry (via the older net user, still works)
net user ITSupport /logonpasswordchg:yes
net user ITSupport /expires:never
```

## Groups

```powershell
# List all local groups
Get-LocalGroup

# List members of a group (e.g. who's a local admin)
Get-LocalGroupMember -Group "Administrators"

# Add a user to the local Administrators group
Add-LocalGroupMember -Group "Administrators" -Member "ITSupport"

# Add a DOMAIN user/group to a local group (note the DOMAIN\ prefix)
Add-LocalGroupMember -Group "Remote Desktop Users" -Member "DOMAIN\jbloggs"

# Remove someone from a group
Remove-LocalGroupMember -Group "Administrators" -Member "ITSupport"

# Create a new local group
New-LocalGroup -Name "Local Support Staff" -Description "Local IT support accounts"
```

## Quick audits worth running

```powershell
# Who's actually a local admin on this machine right now
Get-LocalGroupMember -Group "Administrators" | Select-Object Name, PrincipalSource

# Any local accounts with passwords that never expire
Get-LocalUser | Where-Object { $_.PasswordExpires -eq $null -and $_.Enabled }

# Built-in Administrator/Guest account status (should usually be disabled)
Get-LocalUser -Name "Administrator", "Guest" | Select-Object Name, Enabled
```
