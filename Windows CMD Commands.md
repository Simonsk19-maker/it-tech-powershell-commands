# Windows CMD Commands — IT Tech Reference

Classic Command Prompt (`cmd.exe`) commands — still faster to reach for
than PowerShell for a lot of quick diagnostic work, and some of these
don't have a direct PowerShell equivalent worth typing out.

## Networking

```cmd
:: Full IP configuration
ipconfig /all

:: Release and renew DHCP lease
ipconfig /release
ipconfig /renew

:: Flush the local DNS resolver cache
ipconfig /flushdns

:: Register this machine's DNS record again (domain-joined machines)
ipconfig /registerdns

:: Ping a host (add -t for continuous, Ctrl+C to stop)
ping 192.168.1.1
ping -t 192.168.1.1

:: Trace the route to a host
tracert google.com

:: DNS lookup
nslookup google.com
nslookup google.com 8.8.8.8

:: Show active connections and listening ports
netstat -ano

:: Show which process owns a specific port (pair with netstat -ano, match the PID)
netstat -ano | findstr :3389

:: Show the ARP table (MAC-to-IP mappings on the local network)
arp -a

:: Show/flush/reset the routing table
route print
netsh int ip reset

:: Test a network share is reachable and map a drive letter
net use Z: \\server\share /persistent:yes
net use Z: /delete
```

## System info and diagnostics

```cmd
:: Full system summary (OS version, patches, memory, etc.)
systeminfo

:: Who am I logged in as, and what groups
whoami
whoami /groups

:: List running processes
tasklist

:: Kill a process by name or PID
taskkill /IM notepad.exe /F
taskkill /PID 1234 /F

:: List installed hotfixes/updates
wmic qfe list

:: Check disk for errors (schedules on next reboot if the drive is in use)
chkdsk C: /f

:: Repair Windows system files
sfc /scannow

:: Repair the Windows component store (if sfc alone doesn't fix it)
DISM /Online /Cleanup-Image /RestoreHealth

:: Force a Group Policy update
gpupdate /force

:: Show applied Group Policy results for troubleshooting
gpresult /r
gpresult /h C:\gpresult.html
```

## Files and disks

```cmd
:: Robust file/folder copy - the real workhorse for migrations and backups
robocopy C:\Source D:\Destination /MIR /COPYALL /R:2 /W:5

:: List directory contents with hidden/system files
dir /a

:: Find a file by name across a drive
dir C:\ /s /b | findstr /i "filename.txt"

:: Show disk space
wmic logicaldisk get caption,freespace,size
```

## User/account management

```cmd
:: List local users
net user

:: Details on a specific user
net user jbloggs

:: Create a local user
net user jbloggs P@ssw0rd123 /add

:: Add a user to the local Administrators group
net localgroup administrators jbloggs /add

:: Force password change at next logon
net user jbloggs /logonpasswordchg:yes

:: Disable/enable an account
net user jbloggs /active:no
net user jbloggs /active:yes
```

## Services

```cmd
:: List all services and their status
sc query

:: Query a specific service
sc query spooler

:: Start / stop / configure a service
net start spooler
net stop spooler
sc config spooler start= auto
```
