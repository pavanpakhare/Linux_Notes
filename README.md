# Linux_Notes

## Table of Contents
1. [Introduction to Linux](#introduction-to-linux)
2. [Getting Started](#getting-started)
3. [File System Basics](#file-system-basics)
4. [Essential Commands](#essential-commands)
5. [User and Permission Management](#user-and-permission-management)
6. [Process Management](#process-management)
7. [Networking](#networking)
8. [Shell Scripting](#shell-scripting)
9. [System Administration](#system-administration)
10. [Advanced Topics](#advanced-topics)

## Introduction to Linux

### What is Linux?
Linux is an open-source, Unix-like operating system kernel first released by Linus Torvalds in 1991. Combined with GNU utilities, it forms a complete operating system.

### Linux Distributions
Popular distributions include:
- Ubuntu
- Debian
- CentOS/RHEL
- Fedora
- Arch Linux
- openSUSE

### Why Use Linux?
- Free and open-source
- Highly customizable
- Stable and secure
- Excellent for development
- Powers most servers and cloud infrastructure

## Getting Started

### Accessing Linux
1. Install natively (dual boot)
2. Use a virtual machine (VirtualBox, VMware)
3. Cloud instances (AWS, GCP, Azure)
4. Windows Subsystem for Linux (WSL)

### Basic Terminal Navigation
```bash
whoami        # Show current user
pwd           # Print working directory
ls            # List directory contents
cd            # Change directory
clear         # Clear screen
```

### Getting Help
```bash
man <command>     # Manual pages
<command> --help  # Brief help
info <command>    # Detailed info
```

## File System Basics

### Linux Directory Structure
```
/           # Root directory
/bin        # Essential binaries
/etc        # Configuration files
/home       # User home directories
/var        # Variable data (logs, etc.)
/tmp        # Temporary files
/usr        # User programs
/opt        # Optional software
```

### File Operations
```bash
touch file.txt    # Create empty file
mkdir dir         # Create directory
cp file1 file2    # Copy file
mv file1 file2    # Move/rename file
rm file           # Remove file
rm -r dir         # Remove directory recursively
```

### File Viewing
```bash
cat file          # Display file content
less file         # View file page by page
head -n 5 file    # Show first 5 lines
tail -n 5 file    # Show last 5 lines
tail -f logfile   # Follow log file in real-time
```

## Essential Commands

### Text Processing
```bash
grep "pattern" file    # Search for pattern
sed 's/old/new/g' file # Replace text
awk '{print $1}' file  # Print first column
sort file              # Sort lines
uniq file             # Filter duplicate lines
wc file               # Word count
```

### Compression and Archiving
```bash
gzip file            # Compress file
gunzip file.gz       # Decompress
tar -czvf archive.tar.gz dir/  # Create tar.gz
tar -xzvf archive.tar.gz       # Extract tar.gz
zip archive.zip file           # Create zip
unzip archive.zip              # Extract zip
```

### System Information
```bash
uname -a            # System information
df -h               # Disk space
free -h            # Memory usage
uptime             # System uptime
top                # Process monitor (q to quit)
htop               # Enhanced process viewer
```

## User and Permission Management

### User Accounts
```bash
sudo adduser newuser    # Add user
sudo deluser olduser    # Remove user
passwd username        # Change password
su - username          # Switch user
sudo <command>         # Run as root
```

### File Permissions
Permissions are represented as:
- r (read) = 4
- w (write) = 2
- x (execute) = 1

```bash
chmod 755 file      # rwxr-xr-x
chmod +x script.sh  # Add execute permission
chown user:group file  # Change owner/group
```

### Special Permissions
```bash
chmod +s file       # Set SUID/SGID
chmod +t dir        # Sticky bit
```

## Process Management

### Viewing Processes
```bash
ps aux             # List all processes
ps -ef | grep nginx  # Find specific process
pstree             # Process tree
```

### Managing Processes
```bash
kill -9 PID        # Force kill process
killall process    # Kill all processes by name
pkill pattern      # Kill by pattern
nice -n 10 command # Run with low priority
renice 15 -p PID   # Change priority
```

### Background Jobs
```bash
command &          # Run in background
jobs               # List background jobs
fg %1              # Bring job 1 to foreground
bg %1              # Continue job in background
ctrl+z             # Suspend current job
```

## Networking

### Basic Networking Commands
```bash
ping example.com       # Test connectivity
ifconfig               # Network interfaces (deprecated)
ip addr                # Modern alternative
netstat -tuln          # Listening ports
ss -tuln               # Newer alternative
traceroute example.com # Network path
mtr example.com        # Advanced traceroute
```

### Remote Access
```bash
ssh user@host          # Secure shell
scp file user@host:path  # Secure copy
rsync -avz src/ dest/  # Efficient file sync
```

### Firewall (UFW)
```bash
sudo ufw enable        # Enable firewall
sudo ufw allow 22      # Allow SSH
sudo ufw deny 80       # Block HTTP
sudo ufw status        # View rules
```

## Shell Scripting

### Basic Script
```bash
#!/bin/bash
# This is a comment
echo "Hello, $USER!"
```

### Variables
```bash
name="Linux"
echo "Welcome to $name"
```

### Conditionals
```bash
if [ $1 -gt 100 ]; then
    echo "Large number"
elif [ $1 -eq 0 ]; then
    echo "Zero"
else
    echo "Small number"
fi
```

### Loops
```bash
# For loop
for i in {1..5}; do
    echo "Number $i"
done

# While loop
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done
```

### Functions
```bash
greet() {
    echo "Hello, $1!"
}
greet "Alice"
```

## System Administration

### Package Management (Ubuntu/Debian)
```bash
sudo apt update        # Update package list
sudo apt upgrade      # Upgrade packages
sudo apt install package  # Install package
sudo apt remove package  # Remove package
sudo apt autoremove   # Remove unused packages
```

### Services
```bash
sudo systemctl start service    # Start service
sudo systemctl stop service     # Stop service
sudo systemctl restart service  # Restart service
sudo systemctl status service   # Check status
sudo systemctl enable service   # Enable at boot
```

### Logs
```bash
sudo journalctl -xe            # System logs
sudo tail -f /var/log/syslog   # System log file
dmesg                          # Kernel messages
```

### Cron Jobs
```bash
crontab -e            # Edit cron jobs
# Example: Run daily at 3am
0 3 * * * /path/to/script.sh
```

## Advanced Topics

### Kernel Modules
```bash
lsmod                  # List loaded modules
modinfo module         # Module information
sudo modprobe module   # Load module
sudo rmmod module      # Remove module
```

### SELinux/AppArmor
```bash
sestatus               # Check SELinux status
getenforce             # Get enforcement mode
sudo setenforce 1      # Enable enforcing mode
```

### Performance Monitoring
```bash
vmstat 1              # System performance
iostat 1              # Disk I/O
sar                   # System activity reporter
perf stat command     # Performance statistics
```

### LVM (Logical Volume Management)
```bash
pvcreate /dev/sdb1    # Create physical volume
vgcreate vg0 /dev/sdb1 # Create volume group
lvcreate -L 10G -n lv0 vg0 # Create logical volume
```

### Containers and Virtualization
```bash
# Docker
sudo docker ps
sudo docker images

# Podman (Docker alternative)
podman run -it ubuntu

# KVM/QEMU
sudo virt-manager
```

### Automation Tools
```bash
# Ansible
ansible-playbook playbook.yml

# Puppet
puppet apply manifest.pp

# Chef
chef-client
```



