# Networking Diagnostics — PowerShell Commands

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

## Firewall

```powershell
# List all firewall rules that are enabled
Get-NetFirewallRule -Enabled True | Select-Object DisplayName, Direction, Action

# Open a port
New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow

# Check if Windows Firewall is even on
Get-NetFirewallProfile | Select-Object Name, Enabled
```

## Shares / network paths

```powershell
# Test if a UNC path is reachable
Test-Path "\\server\share"

# Map a network drive
New-PSDrive -Name "S" -PSProvider FileSystem -Root "\\server\share" -Persist
```
