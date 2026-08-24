# File Shares and Permissions — PowerShell Commands

## SMB Shares

```powershell
# List all shares on this server (excluding admin shares like C$)
Get-SmbShare | Where-Object { $_.Name -notmatch '\$$' }

# Create a new share
New-SmbShare -Name "Finance" -Path "D:\Shares\Finance" -Description "Finance department files"

# Remove a share (doesn't delete the underlying files, just unshares them)
Remove-SmbShare -Name "OldShare" -Force

# See who's currently connected to a share
Get-SmbSession
Get-SmbOpenFile
```

## Share-level permissions

```powershell
# See who has access to a share, and what level
Get-SmbShareAccess -Name "Finance"

# Grant access
Grant-SmbShareAccess -Name "Finance" -AccountName "DOMAIN\Finance Team" -AccessRight Change -Force

# Remove access
Revoke-SmbShareAccess -Name "Finance" -AccountName "DOMAIN\Everyone" -Force
```

## NTFS permissions (the folder/file level, separate from share permissions)

```powershell
# View the current NTFS permissions on a folder
Get-Acl "D:\Shares\Finance" | Format-List

# Grant NTFS permission to a user/group
$acl = Get-Acl "D:\Shares\Finance"
$rule = New-Object System.Security.AccessControl.FileSystemAccessRule("DOMAIN\Finance Team", "Modify", "ContainerInherit,ObjectInherit", "None", "Allow")
$acl.SetAccessRule($rule)
Set-Acl "D:\Shares\Finance" $acl

# Copy a whole folder structure preserving NTFS permissions, owner, and timestamps (great for server migrations)
robocopy "D:\OldShare" "D:\NewShare" /MIR /COPYALL /SEC /ZB /MT:16 /R:2 /W:5 /LOG:"C:\robocopy.log"
```

## Finding what's using disk space

```powershell
# Largest folders under a path (top-level only, quick check)
Get-ChildItem "D:\Shares" -Directory | ForEach-Object {
    [PSCustomObject]@{ Folder = $_.Name; SizeGB = [math]::Round((Get-ChildItem $_.FullName -Recurse -File -ErrorAction SilentlyContinue | Measure-Object Length -Sum).Sum / 1GB, 2) }
} | Sort-Object SizeGB -Descending
```
