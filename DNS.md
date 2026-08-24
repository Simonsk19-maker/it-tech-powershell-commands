# DNS — PowerShell Commands

Needs the `DnsServer` module (runs on a DNS server, or remotely with
`-ComputerName`).

## Zones

```powershell
# List all zones and whether they're AD-integrated
Get-DnsServerZone | Select-Object ZoneName, ZoneType, IsDsIntegrated

# Create a new AD-integrated forward lookup zone
Add-DnsServerPrimaryZone -Name "domain.local" -ReplicationScope Domain

# Export a zone to a file (useful before making changes, or for non-AD-integrated zone migration)
Export-DnsServerZone -Name "domain.local" -FileName "domain.local.export.txt"
```

## Records

```powershell
# List all records in a zone
Get-DnsServerResourceRecord -ZoneName "domain.local"

# Add an A record
Add-DnsServerResourceRecordA -ZoneName "domain.local" -Name "fileserver" -IPv4Address "192.168.1.10"

# Add a CNAME (alias) record
Add-DnsServerResourceRecordCName -ZoneName "domain.local" -Name "www" -HostNameAlias "webserver.domain.local"

# Remove a record
Remove-DnsServerResourceRecord -ZoneName "domain.local" -Name "oldserver" -RRType A -Force

# Find a specific record
Get-DnsServerResourceRecord -ZoneName "domain.local" -Name "fileserver"
```

## Cache / troubleshooting

```powershell
# Clear the DNS server's cache (server-side)
Clear-DnsServerCache -Force

# Clear the local machine's resolver cache (client-side)
Clear-DnsClientCache

# Test resolution against a specific DNS server
Resolve-DnsName -Name "fileserver.domain.local" -Server 192.168.1.1

# Show which DNS servers a network adapter is actually using
Get-DnsClientServerAddress
```
