# System and Services — PowerShell Commands

## Services

```powershell
# List all running services
Get-Service | Where-Object { $_.Status -eq "Running" }

# Check a specific service
Get-Service -Name "Spooler"

# Start / stop / restart a service
Start-Service -Name "Spooler"
Stop-Service -Name "Spooler" -Force
Restart-Service -Name "Spooler" -Force

# Set a service to start automatically
Set-Service -Name "Spooler" -StartupType Automatic
```

## Processes

```powershell
# Top 10 processes by memory use
Get-Process | Sort-Object WS -Descending | Select-Object -First 10 Name, Id, @{N="MemoryMB";E={[math]::Round($_.WS/1MB,1)}}

# Kill a stuck process
Stop-Process -Name "notepad" -Force
Stop-Process -Id 1234 -Force
```

## Windows Features / Roles (Server)

```powershell
# List installed roles/features
Get-WindowsFeature | Where-Object { $_.InstallState -eq "Installed" }

# Install a role/feature
Install-WindowsFeature -Name "DHCP" -IncludeManagementTools

# Remove a role/feature
Uninstall-WindowsFeature -Name "DHCP"
```

## Updates

```powershell
# Check installed update history (last 20)
Get-HotFix | Sort-Object InstalledOn -Descending | Select-Object -First 20

# Windows Update via PSWindowsUpdate module (needs Install-Module PSWindowsUpdate first)
Get-WindowsUpdate
Install-WindowsUpdate -AcceptAll -AutoReboot
```

## Disks and storage

```powershell
# Disk space on all drives
Get-PSDrive -PSProvider FileSystem | Select-Object Name, @{N="UsedGB";E={[math]::Round($_.Used/1GB,1)}}, @{N="FreeGB";E={[math]::Round($_.Free/1GB,1)}}

# Physical disk health
Get-PhysicalDisk | Select-Object FriendlyName, HealthStatus, OperationalStatus

# Volumes and their health
Get-Volume
```

## Restarts / remote management

```powershell
# Restart a remote computer
Restart-Computer -ComputerName "SERVER01" -Force

# Run a command against a remote computer
Invoke-Command -ComputerName "SERVER01" -ScriptBlock { Get-Service Spooler }

# Get basic system info
Get-ComputerInfo | Select-Object WindowsProductName, OsVersion, CsTotalPhysicalMemory
```
