# Firewall Rules — PowerShell Commands

Windows Firewall with Advanced Security, via PowerShell.

## Viewing rules

```powershell
# All enabled rules
Get-NetFirewallRule -Enabled True | Select-Object DisplayName, Direction, Action

# Find a rule by name (partial match)
Get-NetFirewallRule -DisplayName "*Remote Desktop*"

# See the actual port/protocol behind a rule
Get-NetFirewallRule -DisplayName "Allow RDP" | Get-NetFirewallPortFilter

# See the program a rule applies to
Get-NetFirewallRule -DisplayName "Allow RDP" | Get-NetFirewallApplicationFilter

# All rules for a specific port
Get-NetFirewallRule | Get-NetFirewallPortFilter | Where-Object { $_.LocalPort -eq 3389 }
```

## Creating rules

```powershell
# Allow inbound on a specific TCP port
New-NetFirewallRule -DisplayName "Allow RDP" -Direction Inbound -Protocol TCP -LocalPort 3389 -Action Allow

# Allow inbound on a specific UDP port
New-NetFirewallRule -DisplayName "Allow DNS" -Direction Inbound -Protocol UDP -LocalPort 53 -Action Allow

# Allow a specific program through the firewall (any port it needs)
New-NetFirewallRule -DisplayName "Allow LOB App" -Direction Inbound -Program "C:\Program Files\App\app.exe" -Action Allow

# Allow only from a specific source IP/range
New-NetFirewallRule -DisplayName "Allow SQL from App Server" -Direction Inbound -Protocol TCP -LocalPort 1433 `
    -RemoteAddress 192.168.1.50 -Action Allow

# Block outbound to a specific IP
New-NetFirewallRule -DisplayName "Block Bad IP" -Direction Outbound -RemoteAddress 203.0.113.5 -Action Block

# Scope a rule to a specific profile only (Domain / Private / Public)
New-NetFirewallRule -DisplayName "Allow File Sharing (Domain only)" -Direction Inbound -Protocol TCP -LocalPort 445 `
    -Profile Domain -Action Allow
```

## Modifying / removing rules

```powershell
# Disable a rule (keeps it, just turns it off)
Disable-NetFirewallRule -DisplayName "Allow RDP"

# Re-enable it
Enable-NetFirewallRule -DisplayName "Allow RDP"

# Change an existing rule's port
Set-NetFirewallRule -DisplayName "Allow RDP" -LocalPort 3390

# Delete a rule entirely
Remove-NetFirewallRule -DisplayName "Old Test Rule"
```

## Firewall profiles

```powershell
# Check status of all 3 profiles (Domain, Private, Public)
Get-NetFirewallProfile | Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction

# Turn the firewall on/off for a specific profile (rarely a good idea for Public - be careful)
Set-NetFirewallProfile -Profile Domain -Enabled True
Set-NetFirewallProfile -Profile Public -Enabled False

# Set default inbound action for a profile
Set-NetFirewallProfile -Profile Domain -DefaultInboundAction Block -DefaultOutboundAction Allow
```

## Logging (useful for troubleshooting "is the firewall blocking this")

```powershell
# Enable logging of dropped packets
Set-NetFirewallProfile -Profile Domain,Private,Public -LogBlocked True `
    -LogFileName "%systemroot%\system32\LogFiles\Firewall\pfirewall.log" -LogMaxSizeKilobytes 4096

# Where the log lives by default
Get-NetFirewallProfile | Select-Object Name, LogFileName, LogBlocked
```

## Common real-world scenarios

```powershell
# Allow ping (ICMP) - off by default on some profiles
New-NetFirewallRule -DisplayName "Allow Ping" -Protocol ICMPv4 -IcmpType 8 -Direction Inbound -Action Allow

# Quickly check if a specific port is actually reachable from another machine
Test-NetConnection -ComputerName "192.168.1.10" -Port 3389

# Find what's actually listening on a port locally (is a service even running there)
Get-NetTCPConnection -LocalPort 3389 -State Listen
```

See also [[Networking Diagnostics]] for connectivity testing more broadly.
