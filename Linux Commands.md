# Linux Commands — IT Tech Reference

Common terminal commands across most distros (Ubuntu/Debian commands
noted separately where package management differs from RHEL/CentOS).

## Networking

```bash
# IP configuration (modern replacement for ifconfig)
ip a
ip addr show eth0

# Set/bring an interface up or down
sudo ip link set eth0 up
sudo ip link set eth0 down

# Ping a host
ping google.com
ping -c 4 google.com   # 4 packets then stop

# Trace the route to a host
traceroute google.com

# DNS lookup
dig google.com
nslookup google.com

# Show listening ports and active connections (ss is the modern netstat)
ss -tulpn
sudo netstat -tulpn

# Show routing table
ip route
route -n

# Test if a specific port is reachable
nc -zv 192.168.1.10 22

# Flush/restart networking (systemd-based distros)
sudo systemctl restart NetworkManager
```

## File system and permissions

```bash
# List files, including hidden, with details
ls -la

# Change file/folder permissions (numeric: owner/group/other, read=4 write=2 exec=1)
chmod 755 script.sh
chmod -R 644 /var/www/html

# Change ownership
sudo chown user:group file.txt
sudo chown -R user:group /path/to/folder

# Find a file by name
find / -name "filename.txt" 2>/dev/null

# Find files modified in the last 24 hours
find /var/log -mtime -1

# Disk usage - overall
df -h

# Disk usage - by folder (find what's eating space)
du -sh /var/log/*
du -sh * | sort -rh | head -10

# Copy, including preserving permissions/ownership/timestamps
cp -a source/ destination/

# Robust sync (like robocopy on Windows)
rsync -avh --progress source/ destination/
```

## Processes and services

```bash
# List running processes (interactive, like Task Manager)
top
htop   # nicer version, often needs installing first

# List processes, full detail
ps aux

# Find a specific process
ps aux | grep nginx

# Kill a process by PID or name
kill -9 1234
killall nginx

# Service management (systemd - most modern distros)
sudo systemctl status nginx
sudo systemctl start nginx
sudo systemctl stop nginx
sudo systemctl restart nginx
sudo systemctl enable nginx    # start automatically on boot
sudo systemctl disable nginx

# List all services and their status
systemctl list-units --type=service
```

## Logs

```bash
# View the systemd journal (most modern distros)
journalctl -xe                  # recent errors
journalctl -u nginx             # logs for one service
journalctl -f                   # follow live, like tail -f
journalctl --since "1 hour ago"

# Classic log files still worth knowing
tail -f /var/log/syslog         # Debian/Ubuntu
tail -f /var/log/messages       # RHEL/CentOS
tail -f /var/log/auth.log       # login/SSH attempts (Debian/Ubuntu)
```

## Users and groups

```bash
# Add a new user
sudo useradd -m -s /bin/bash newuser
sudo passwd newuser

# Add a user to a group (e.g. sudo/admin access)
sudo usermod -aG sudo newuser    # Debian/Ubuntu
sudo usermod -aG wheel newuser   # RHEL/CentOS

# List a user's groups
groups newuser

# Delete a user (and their home folder)
sudo userdel -r newuser

# Who's currently logged in
who
w
```

## Package management

```bash
# Debian / Ubuntu
sudo apt update && sudo apt upgrade -y
sudo apt install package-name
sudo apt remove package-name
apt list --installed

# RHEL / CentOS / Fedora
sudo dnf update -y
sudo dnf install package-name
sudo dnf remove package-name
```

## Firewall

```bash
# UFW (Debian/Ubuntu default)
sudo ufw status
sudo ufw allow 22/tcp
sudo ufw allow from 192.168.1.0/24 to any port 3306
sudo ufw deny 23
sudo ufw enable

# firewalld (RHEL/CentOS default)
sudo firewall-cmd --list-all
sudo firewall-cmd --add-port=22/tcp --permanent
sudo firewall-cmd --reload
```

## SSH

```bash
# Connect to a remote server
ssh user@192.168.1.10
ssh user@192.168.1.10 -p 2222   # non-default port

# Copy a file to/from a remote server
scp file.txt user@192.168.1.10:/home/user/
scp user@192.168.1.10:/home/user/file.txt ./

# Generate an SSH key pair (for passwordless login)
ssh-keygen -t ed25519 -C "comment"

# Copy your public key to a remote server for passwordless login
ssh-copy-id user@192.168.1.10
```

## System info

```bash
# OS/distro version
cat /etc/os-release
uname -a

# Uptime and load average
uptime

# Memory usage
free -h

# Reboot / shutdown
sudo reboot
sudo shutdown -h now
```
