# DHCP — PowerShell Commands

Needs the `DhcpServer` module.

## Scopes

```powershell
# List all scopes and their state
Get-DhcpServerv4Scope | Select-Object ScopeId, Name, StartRange, EndRange, State

# Create a new scope
Add-DhcpServerv4Scope -Name "Main Office" -StartRange 192.168.1.100 -EndRange 192.168.1.200 `
    -SubnetMask 255.255.255.0 -State Active

# Deactivate / activate a scope (e.g. during a server cutover)
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State InActive
Set-DhcpServerv4Scope -ScopeId 192.168.1.0 -State Active

# Set scope options (e.g. DNS server, default gateway)
Set-DhcpServerv4OptionValue -ScopeId 192.168.1.0 -DnsServer 192.168.1.1 -Router 192.168.1.254
```

## Leases and reservations

```powershell
# List current leases in a scope
Get-DhcpServerv4Lease -ScopeId 192.168.1.0

# Reserve an IP for a specific device (by MAC address)
Add-DhcpServerv4Reservation -ScopeId 192.168.1.0 -IPAddress 192.168.1.50 `
    -ClientId "00-11-22-33-44-55" -Name "Reception PC"

# List all reservations in a scope
Get-DhcpServerv4Reservation -ScopeId 192.168.1.0
```

## Server-level / migration

```powershell
# Authorize this server as a DHCP server in AD (required before it can hand out leases)
Add-DhcpServerInDC -DnsName "server.domain.local"

# List authorized DHCP servers in AD
Get-DhcpServerInDC

# Export the entire DHCP configuration (scopes, reservations, leases, options)
Export-DhcpServer -File "C:\DHCPExport\export.xml" -Leases

# Import it on another server
Import-DhcpServer -File "C:\DHCPExport\export.xml" -BackupPath "C:\DHCPImportBackup"
```
