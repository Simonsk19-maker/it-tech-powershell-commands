# Networking Diagnostics — PowerShell Commands

See [[Firewall Rules]] for creating/managing Windows Firewall rules.

## Connectivity

```powershell
# Modern ping replacement, with more detail
Test-Connection -ComputerName "192.168.1.10" -Count 4

# Check if a specific port is open (e.g. is SQL/RDP/HTTP actually reachable)
Test-NetConnection -ComputerName "192.168.1.10" -Port 3389

# Trace the route to a host
Test-NetConnection -ComputerName "8.8.8.8" -TraceRoute
```

## IP configuration

```powershell
# Full IP config (replacement for ipconfig /all)
Get-NetIPConfiguration -Detailed

# Just the IP addresses
Get-NetIPAddress -AddressFamily IPv4

# Release / renew a DHCP lease
Remove-NetIPAddress -InterfaceAlias "Ethernet" -Confirm:$false
Restart-NetAdapter -Name "Ethernet"

# Set a static IP
New-NetIPAddress -InterfaceAlias "Ethernet" -IPAddress 192.168.1.10 -PrefixLength 24 -DefaultGateway 192.168.1.1
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.1.1
```

## Shares / network paths

```powershell
# Test if a UNC path is reachable
Test-Path "\\server\share"

# Map a network drive
New-PSDrive -Name "S" -PSProvider FileSystem -Root "\\server\share" -Persist
```
