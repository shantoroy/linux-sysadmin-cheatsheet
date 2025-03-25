# Linux Command Scenarios - Part 1

This document provides practical command solutions for common Linux administration scenarios, organized by category. Each scenario includes a description and the complete command or sequence of commands needed to accomplish the task.

## Table of Contents
- [YUM Package Management](#yum-package-management)
- [Text Manipulation Scenarios](#text-manipulation-scenarios)
- [File Management Scenarios](#file-management-scenarios)
- [Log Analysis and Management](#log-analysis-and-management)
- [User Management Scenarios](#user-management-scenarios)
- [Process Management Scenarios](#process-management-scenarios)
- [Disk Management Scenarios](#disk-management-scenarios)
- [Networking Scenarios](#networking-scenarios)
- [Security Scenarios](#security-scenarios)
- [SSL/TLS Certificate Management](#ssltls-certificate-management)
- [System Monitoring Scenarios](#system-monitoring-scenarios)
- [Database Server Management](#database-server-management)
- [Web Server Management](#web-server-management)

## YUM Package Management

### Repository Management

#### List All Configured Repositories
```bash
# List all enabled and disabled repositories
yum repolist all

# List only enabled repositories
yum repolist enabled

# List only disabled repositories
yum repolist disabled
```

#### Get Repository Information
```bash
# Get detailed information about a specific repository
yum repoinfo base

# Get verbose information about all repositories
yum repoinfo -v
```

#### Add a New Repository
```bash
# Add a repository manually by creating a .repo file
cat > /etc/yum.repos.d/example.repo << EOF
[example]
name=Example Repository
baseurl=http://example.com/repo
enabled=1
gpgcheck=1
gpgkey=http://example.com/key.asc
EOF

# Add a repository using rpm
rpm -Uvh https://example.com/example-repo.rpm
```

#### Enable/Disable Repositories
```bash
# Disable a repository
yum-config-manager --disable example

# Enable a repository
yum-config-manager --enable example

# Install yum-utils if yum-config-manager is not available
yum install yum-utils
```

#### Clean Repository Cache
```bash
# Clean all cached packages and metadata
yum clean all

# Clean just the cached packages
yum clean packages

# Clean just the metadata
yum clean metadata

# Rebuild the yum cache
yum makecache
```

### Package Queries and Information

#### Search for Packages
```bash
# Search for packages by name
yum search nginx

# Search package names and summaries for a keyword
yum search all "web server"

# List packages matching a specific name pattern
yum list "php*"

# List available packages that can be installed
yum list available

# List installed packages
yum list installed

# List packages that have updates available
yum list updates

# Check if a specific package is installed
yum list installed httpd
```

#### Get Package Information
```bash
# Get detailed information about a package
yum info nginx

# Get information about a specific installed package
yum info installed httpd

# List all versions of a package available
yum --showduplicates list httpd
```

#### Find Which Package Provides a File
```bash
# Find which package provides a specific file
yum provides /usr/bin/htpasswd

# Alternative syntax
yum whatprovides */htpasswd
```

#### List Package Dependencies
```bash
# Show dependencies for a package
yum deplist httpd

# Show packages that require a specific package
yum repoquery --requires httpd
```

#### List Files in a Package
```bash
# List all files included in an installed package
rpm -ql httpd

# List all files in a package (not installed)
repoquery -l httpd
```

#### Show Package Change History
```bash
# Show transaction history
yum history

# Get details about a specific transaction
yum history info 25

# Show history for a specific package
yum history list httpd
```

### Package Installation and Management

#### Install Packages
```bash
# Install a single package
yum install httpd

# Install multiple packages
yum install httpd php php-mysql

# Install a specific version of a package
yum install httpd-2.4.6-93.el7

# Install a local RPM package with dependency resolution
yum localinstall ./package.rpm
```

#### Update Packages
```bash
# Check for updates
yum check-update

# Update a specific package
yum update httpd

# Update all packages
yum update

# Apply only security updates
yum update --security
```

#### Remove Packages
```bash
# Remove a package
yum remove httpd

# Remove a package without removing dependencies
yum remove --noautoremove httpd
```

#### Install Package Groups
```bash
# List available package groups
yum group list

# Get information about a package group
yum group info "Development Tools"

# Install a package group
yum group install "Development Tools"

# Update a package group
yum group update "Development Tools"

# Remove a package group
yum group remove "Development Tools"
```

#### Downgrade Packages
```bash
# Downgrade a package to a previous version
yum downgrade httpd
```

#### Reinstall Packages
```bash
# Reinstall a package
yum reinstall httpd
```

### Transaction Management

#### Perform Dry Run
```bash
# See what would happen for any yum command without actually doing it
yum install nginx --assumeno
```

#### Download Only
```bash
# Download a package without installing it
yum install --downloadonly --downloaddir=/tmp/packages httpd
```

#### Rollback Transactions
```bash
# Undo a transaction by ID
yum history undo 42

# Rollback to a specific transaction ID
yum history rollback 36
```

#### Work with Multiple Packages
```bash
# Install all packages needed for a specific task
yum install @lamp-server

# See what package group provides a functionality
yum group list | grep -i database
```

### Advanced YUM Operations

#### Configure YUM Settings
```bash
# List all YUM configuration settings
yum-config-manager --dump

# Set a specific configuration option
yum-config-manager --save --setopt=timeout=30
```

#### Work with Yum Plugins
```bash
# List installed YUM plugins
yum list installed "yum-plugin-*"

# Install a YUM plugin
yum install yum-plugin-fastestmirror
```

#### Lock Package Versions
```bash
# Prevent a package from being updated (requires versionlock plugin)
yum install yum-plugin-versionlock
yum versionlock add httpd

# Display version-locked packages
yum versionlock list

# Remove a version lock
yum versionlock delete httpd
```

#### Work with Security Updates
```bash
# List available security updates
yum updateinfo list security

# Apply security updates rated as critical
yum update --security --sec-severity=Critical
```

#### Package Verification
```bash
# Verify integrity of an installed package
rpm -V httpd

# Verify all installed packages
rpm -Va
```

#### Troubleshooting Package Issues
```bash
# Fix package dependencies issues
yum clean all
yum distro-sync

# Check for conflicting files
rpm -Va | grep -i missing
```

### Switching to DNF (YUM's Successor)

#### Basic DNF Usage (Similar to YUM)
```bash
# Install DNF
yum install dnf

# Basic operations with DNF (syntax is very similar to YUM)
dnf install httpd
dnf update
dnf remove httpd
```

#### DNF-Specific Features
```bash
# Automatic removal of no longer needed dependencies
dnf autoremove

# List available modules
dnf module list

# Enable a specific module stream
dnf module enable php:7.4

# Install a specific module
dnf module install php:7.4
```

### Practical Scenarios

#### Find All Installed Security-Related Packages
```bash
yum list installed | grep -E 'security|firewall|ssl|crypto'
```

#### Find Large Installed Packages
```bash
rpm -qa --queryformat '%{size} %{name}-%{version}\n' | sort -rn | head -20
```

#### Clean Up Unused Dependencies
```bash
package-cleanup --leaves
package-cleanup --orphans
```

#### Find What Package a Command Belongs To
```bash
# Find what package installed the 'top' command
rpm -qf $(which top)
```

#### Upgrade to Latest Available Kernel
```bash
yum update kernel
```

#### Find Configuration Files for a Package
```bash
rpm -qc httpd
```

#### Trace Package Installation Process
```bash
yum install -v httpd
```

#### Export List of Installed Packages for Reinstallation
```bash
rpm -qa > installed_packages.txt

# Later, to reinstall packages from this list
yum install $(cat installed_packages.txt)
```

These commands cover most package management tasks that a system administrator would need to perform on Red Hat-based systems, allowing for efficient management of software installations, updates, and repositories.

## Text Manipulation Scenarios

### Find and Replace Text in Files

#### Replace a string in a single file
```bash
# Replace all occurrences of "old_string" with "new_string" in file.txt
sed -i 's/old_string/new_string/g' file.txt
```

#### Replace a string in multiple files
```bash
# Using sed with find to replace in all .conf files
find /path/to/dir -type f -name "*.conf" -exec sed -i 's/old_string/new_string/g' {} \;
```

#### Replace a string only on line 10
```bash
# Replace "old_string" with "new_string" only on line 10
sed -i '10s/old_string/new_string/g' file.txt
```

#### Replace a string with special characters
```bash
# Replace a path like /var/log with /opt/log
sed -i 's/\/var\/log/\/opt\/log/g' file.txt

# Alternative with different delimiter
sed -i 's|/var/log|/opt/log|g' file.txt
```

#### Replace only the first occurrence on each line
```bash
# Replace only the first occurrence of "old_string" per line
sed -i 's/old_string/new_string/' file.txt
```

#### Replace with case insensitive matching
```bash
# Replace with case insensitive matching
sed -i 's/old_string/new_string/gi' file.txt
```

#### Replace whole word only
```bash
# Replace only whole words matching "old"
sed -i 's/\bold\b/new/g' file.txt
```

#### Replace a string across multiple lines
```bash
# Replace string across multiple lines (GNU sed)
sed -i ':a;N;$!ba;s/old_string\n/new_string\n/g' file.txt
```

#### Replace IP address format
```bash
# Replace IP address 192.168.1.100 with 10.0.0.50
sed -i 's/192\.168\.1\.100/10\.0\.0\.50/g' file.txt
```

#### Add a line after a specific pattern
```bash
# Add a new line after lines containing "pattern"
sed -i '/pattern/a\New line text goes here' file.txt
```

#### Add a line before a specific pattern
```bash
# Add a new line before lines containing "pattern"
sed -i '/pattern/i\New line text goes here' file.txt
```

#### Delete lines matching a pattern
```bash
# Delete all lines containing "pattern"
sed -i '/pattern/d' file.txt
```

#### Delete empty lines
```bash
# Delete all empty lines
sed -i '/^$/d' file.txt
```

#### Delete lines with only whitespace
```bash
# Delete lines with only whitespace
sed -i '/^\s*$/d' file.txt
```

#### Comment out lines containing a pattern
```bash
# Comment out lines containing "pattern"
sed -i '/pattern/s/^/#/' file.txt
```

#### Uncomment lines containing a pattern
```bash
# Uncomment lines (remove leading # from lines containing "pattern")
sed -i '/pattern/s/^#//' file.txt
```

### Extract Information from Files

#### Extract lines between two patterns
```bash
# Extract lines between START and END patterns (inclusive)
sed -n '/START/,/END/p' file.txt
```

#### Extract specific columns from CSV files
```bash
# Extract 1st and 3rd columns from CSV file
awk -F, '{print $1,$3}' file.csv

# Extract and reformat with a different delimiter
awk -F, '{print $1"|"$3}' file.csv
```

#### Extract the Nth word from each line
```bash
# Extract the 3rd word from each line
awk '{print $3}' file.txt
```

#### Extract lines matching multiple patterns
```bash
# Extract lines containing either "error" or "warning"
grep -E 'error|warning' file.log
```

#### Extract blocks of text between empty lines
```bash
# Print paragraphs (text blocks separated by empty lines) containing "pattern"
awk -v RS="" '/pattern/' file.txt
```

#### Extract text between XML tags
```bash
# Extract content between <tag> and </tag>
grep -oP '(?<=<tag>).*(?=</tag>)' file.xml
```

#### Extract all IP addresses from a file
```bash
# Extract IPv4 addresses
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file.txt

# More precise IPv4 extraction (better but more complex)
grep -oE '\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b' file.txt
```

#### Extract all email addresses from a file
```bash
# Extract email addresses
grep -oE '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' file.txt
```

#### Extract all URLs from a file
```bash
# Extract URLs
grep -oE 'https?://[^ ]+' file.txt
```

#### Extract specific lines by line number
```bash
# Extract lines 10-20
sed -n '10,20p' file.txt
```

## File Management Scenarios

### Finding and Deleting Files

#### Find and delete files older than X days
```bash
# Find and delete files older than 15 days in /var/log
find /var/log -type f -mtime +15 -exec rm {} \;

# Safer version with confirmation
find /var/log -type f -mtime +15 -exec rm -i {} \;

# List files that would be deleted
find /var/log -type f -mtime +15 -ls
```

#### Find and delete empty files
```bash
# Find and delete empty files
find /path/to/dir -type f -empty -delete
```

#### Find and delete empty directories
```bash
# Find and delete empty directories
find /path/to/dir -type d -empty -delete
```

#### Find and delete files by size
```bash
# Find and delete files larger than 100MB
find /path/to/dir -type f -size +100M -exec rm {} \;

# Find and delete files smaller than 10KB
find /path/to/dir -type f -size -10k -exec rm {} \;
```

#### Find and delete files with specific extensions
```bash
# Find and delete .tmp and .bak files
find /path/to/dir -type f \( -name "*.tmp" -o -name "*.bak" \) -delete
```

#### Clean up specific log files over X size
```bash
# Find log files over 100MB and truncate them
find /var/log -name "*.log" -size +100M -exec sh -c 'echo "" > "{}"' \;

# Alternative using truncate
find /var/log -name "*.log" -size +100M -exec truncate -s 0 {} \;
```

#### Remove duplicate files
```bash
# Find and list duplicate files based on content (requires fdupes)
fdupes -r /path/to/dir

# Find and delete duplicate files (keeping one copy)
fdupes -r -d -N /path/to/dir
```

### Advanced File Operations

#### Find and compress old log files
```bash
# Find log files older than 30 days and compress them
find /var/log -name "*.log" -type f -mtime +30 -exec gzip {} \;
```

#### Batch rename files
```bash
# Rename all .txt files to .bak
find /path/to/dir -name "*.txt" -exec sh -c 'mv "$1" "${1%.txt}.bak"' sh {} \;

# Using rename (perl version) to replace spaces with underscores
rename 's/ /_/g' *.txt

# Rename files with sequential numbers
i=1; for f in *.jpg; do mv "$f" "image_$(printf '%03d' $i).jpg"; ((i++)); done
```

#### Recursively change file permissions
```bash
# Change all directories to 755 and files to 644
find /path/to/dir -type d -exec chmod 755 {} \; 
find /path/to/dir -type f -exec chmod 644 {} \;

# Change file ownership recursively
find /path/to/dir -exec chown user:group {} \;
```

#### Find files modified in the last X minutes
```bash
# Find files modified in the last 60 minutes
find /path/to/dir -type f -mmin -60
```

#### Synchronize two directories
```bash
# Mirror source directory to destination
rsync -avz --delete /source/dir/ /destination/dir/

# Backup with exclusions
rsync -avz --exclude='*.tmp' --exclude='cache/' /source/dir/ /destination/dir/
```

#### Find and archive old files
```bash
# Archive files older than 90 days, then remove them
find /path/to/dir -type f -mtime +90 -exec tar -rvf old_files.tar {} \; -exec rm {} \;
```

#### Handle files with unusual names (spaces, special chars)
```bash
# Find and fix files with problematic names
find /path/to/dir -name "* *" -exec rename 's/ /_/g' {} \;

# Handle files with newlines or special characters
find /path/to/dir -type f -print0 | xargs -0 some_command
```

#### Set creation time to match modification time
```bash
# Make access/creation time match modification time
find /path/to/dir -type f -exec touch -am {} \;
```

## Log Analysis and Management

### Analyzing Log Files

#### Find errors in log files
```bash
# Search for ERROR, error, or Error in log files
grep -i "error" /var/log/*.log

# Show 2 lines before and after each error
grep -i -B2 -A2 "error" /var/log/*.log

# Count errors by type
grep -i "error" /var/log/*.log | awk '{print $NF}' | sort | uniq -c | sort -nr
```

#### Find failed login attempts
```bash
# Check for failed logins in auth log
grep "Failed password" /var/log/auth.log

# Count failed logins by username
grep "Failed password" /var/log/auth.log | awk '{print $(NF-5)}' | sort | uniq -c | sort -nr

# Count failed logins by IP address
grep "Failed password" /var/log/auth.log | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -nr
```

#### Extract and summarize web server access logs
```bash
# Count HTTP status codes
awk '{print $9}' /var/log/apache2/access.log | sort | uniq -c | sort -nr

# Top 10 IP addresses accessing your site
awk '{print $1}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -10

# Top requested pages
awk '{print $7}' /var/log/apache2/access.log | sort | uniq -c | sort -nr | head -10

# Count requests by HTTP method
awk '{print $6}' /var/log/apache2/access.log | tr -d '"' | sort | uniq -c | sort -nr
```

#### Extract time-specific log entries
```bash
# Get log entries between specific times
awk '$4 >= "[01/Mar/2023:10:00:00" && $4 <= "[01/Mar/2023:11:00:00"' /var/log/apache2/access.log

# Get today's log entries
grep "`date +"%b %d"`" /var/log/syslog
```

#### Calculate average response time from logs
```bash
# For logs with response time in the last field
awk '{sum+=$NF; count++} END {print "Average:",sum/count,"ms"}' /var/log/response_times.log
```

#### Analyze sudo usage
```bash
# List all sudo commands executed
grep "sudo:" /var/log/auth.log | grep "COMMAND"

# List sudo commands by user
grep "sudo:" /var/log/auth.log | grep "COMMAND" | awk '{print $6,$NF}' | sort | uniq -c
```

### Log Rotation and Maintenance

#### Manually rotate a log file
```bash
# Create a timestamped backup and reset the log file
cp access.log "access.log.$(date +%Y%m%d)"
cat /dev/null > access.log
```

#### Create a custom logrotate configuration
```bash
# Create a logrotate config for a custom application log
cat > /etc/logrotate.d/myapp << EOF
/var/log/myapp/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 myapp myapp
    sharedscripts
    postrotate
        systemctl reload myapp
    endscript
}
EOF
```

#### Force log rotation
```bash
# Force immediate rotation for all configurations
logrotate -f /etc/logrotate.conf

# Force rotation for a specific configuration
logrotate -f /etc/logrotate.d/myapp
```

#### Set up log file size limits
```bash
# Add size limit to a logrotate configuration
cat > /etc/logrotate.d/size-limit << EOF
/var/log/myapp/*.log {
    size 100M
    rotate 5
    compress
    missingok
    notifempty
}
EOF
```

## User Management Scenarios

### User Account Management

#### Create a batch of users from a file
```bash
# Create users listed in a file (one per line) with default settings
while read username; do useradd -m "$username"; done < userlist.txt

# Create users with specific groups, shells, and home directories
while read username; do useradd -m -g staff -G developers -s /bin/bash -d /home/dev/"$username" "$username"; done < userlist.txt
```

#### Set password expiry for multiple users
```bash
# Set 90-day password expiration for all users in a group
getent group developers | cut -d: -f4 | tr ',' '\n' | xargs -I{} chage -M 90 {}

# Force password change at next login for inactive users
lastlog | grep "**Never logged in**" | awk '{print $1}' | xargs -I{} chage -d 0 {}
```

#### Bulk modify user attributes
```bash
# Add all users to a new supplementary group
getent passwd | awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' | xargs -I{} usermod -aG newgroup {}

# Change shell for all users in a group
getent group developers | cut -d: -f4 | tr ',' '\n' | xargs -I{} usermod -s /bin/zsh {}
```

#### Find and disable inactive user accounts
```bash
# Find users who haven't logged in for over 90 days
lastlog | awk -F' +' 'NR>1 && $5!="**Never" && systime()-mktime($6" "$5" "$9" "$7" "$8)>90*24*60*60 {print $1}' | xargs -I{} usermod -L {}

# Alternative using lastlog for inactive users
for user in $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd); do
  if [[ $(date -d "$(chage -l $user | grep 'Last password change' | cut -d: -f2)" +%s) -lt $(date -d "-90 days" +%s) ]]; then
    usermod -L $user
    echo "Disabled account: $user"
  fi
done
```

#### Find users with sudo privileges
```bash
# List all users with sudo privileges
grep -Po '^[^#]+\s+ALL=(?:\([^)]+\))?\s+ALL' /etc/sudoers | cut -d ' ' -f1

# Alternative, including group membership checks
for user in $(getent passwd | awk -F: '{print $1}'); do
  groups $user | grep -q '\bsudo\b' && echo "$user has sudo via sudo group"
  groups $user | grep -q '\bwheel\b' && echo "$user has sudo via wheel group"
done
grep -v '^#' /etc/sudoers | grep ALL | grep -v '%' | awk '{print $1" has sudo via sudoers file"}'
```

#### Reset a forgotten user password using root
```bash
# Reset password for user
passwd username

# Force user to change password at next login
passwd -e username
```

#### Create a system account
```bash
# Create a system account for a service
useradd -r -s /sbin/nologin myservice

# Create a system account with custom home directory
useradd -r -s /sbin/nologin -d /opt/myapp myapp
```

## Process Management Scenarios

### Monitoring and Controlling Processes

#### Find and kill processes by name
```bash
# Find and kill all processes matching "processname"
pkill processname

# Find and kill with a more precise match
pgrep -f "exact process string" | xargs kill

# Kill processes more aggressively
pkill -9 processname
```

#### Find processes consuming high resources
```bash
# Find top 5 CPU-consuming processes
ps aux --sort=-%cpu | head -6

# Find top 5 memory-consuming processes
ps aux --sort=-%mem | head -6

# Find processes using over 50% CPU
ps aux | awk '$3 > 50.0'
```

#### Kill all processes of a specific user
```bash
# Kill all processes owned by a user
pkill -u username

# Kill more aggressively
pkill -9 -u username
```

#### Run a process that continues after logout
```bash
# Run a command in the background that continues after logout
nohup long_running_command &

# Alternative using screen
screen -dmS sessionname command

# Alternative using tmux
tmux new-session -d 'command'
```

#### Limit resource usage of a process
```bash
# Limit CPU and memory
systemd-run --user --scope -p CPUQuota=50% -p MemoryLimit=1G command

# Traditional way, limiting CPU niceness (-20 to 19, higher = less priority)
nice -n 10 command

# Limit IO priority (0-7, lower = higher priority, requires ionice command)
ionice -c2 -n7 command
```

#### Monitor processes in real time
```bash
# Use top with specific sorting
top -o %CPU

# Use htop with tree view
htop -t

# Monitor specific processes by name
watch "ps aux | grep [p]rocessname"
```

#### Auto-restart a crashed process
```bash
# Simple monitoring script
cat > /usr/local/bin/process_monitor.sh << 'EOF'
#!/bin/bash
while true; do
  if ! pgrep -f "process_name" > /dev/null; then
    /path/to/process_command &
    echo "Process restarted at $(date)" >> /var/log/process_monitor.log
  fi
  sleep 60
done
EOF
chmod +x /usr/local/bin/process_monitor.sh

# Run the monitor as a systemd service
cat > /etc/systemd/system/process-monitor.service << EOF
[Unit]
Description=Process Monitoring Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/process_monitor.sh
Restart=always

[Install]
WantedBy=multi-user.target
EOF

systemctl enable process-monitor.service
systemctl start process-monitor.service
```

## Disk Management Scenarios

### Space Management and Analysis

#### Find directories consuming the most space
```bash
# Show top 10 directories by size
du -hx --max-depth=1 /path | sort -hr | head -10

# Alternative using ncdu for interactive exploration
ncdu /path
```

#### Clean up disk space
```bash
# Clear package manager cache (example for apt)
apt-get clean

# Alternative for dnf/yum
dnf clean all

# Clear systemd journal logs older than 3 days
journalctl --vacuum-time=3d

# Empty all trash folders for all users
rm -rf /home/*/.local/share/Trash/*/** /root/.local/share/Trash/*/**
```

#### Find largest files
```bash
# Find top 20 largest files in a path
find /path -type f -exec du -h {} \; | sort -hr | head -20

# Find large files (>100MB) only
find /path -type f -size +100M -exec ls -lh {} \; | sort -k5,5hr
```

#### Clean temporary files
```bash
# Clean /tmp files older than 10 days
find /tmp -type f -atime +10 -delete

# Clean specific temporary file patterns
find /tmp -name "*.tmp" -o -name "*.temp" -o -name "*.bak" -delete
```

#### Check disk usage by file types
```bash
# Check disk usage by file extension in current directory
find . -type f | grep -v "^\./\." | sed 's/.*\.//' | sort | uniq -c | sort -nr
```

### Filesystem Management

#### Create a new filesystem
```bash
# Create and mount an ext4 filesystem
mkfs.ext4 /dev/sdb1
mkdir -p /mnt/newdisk
mount /dev/sdb1 /mnt/newdisk

# Add to fstab for persistent mounting
echo "/dev/sdb1 /mnt/newdisk ext4 defaults 0 2" >> /etc/fstab
```

#### Extend a logical volume and filesystem
```bash
# Extend a logical volume and resize its filesystem
lvextend -L +10G /dev/mapper/vg-lv
resize2fs /dev/mapper/vg-lv

# Alternative with a single command (for ext4)
lvextend -r -L +10G /dev/mapper/vg-lv

# Resize XFS filesystem (only grow, not shrink)
lvextend -L +10G /dev/mapper/vg-lv
xfs_growfs /mnt/xfs_mount
```

#### Set up disk quotas
```bash
# Install quota tools (example for Debian/Ubuntu)
apt-get install quota

# Edit fstab to enable quotas
# Add "usrquota,grpquota" to mount options
nano /etc/fstab
# Example line:
# /dev/sda1 / ext4 defaults,usrquota,grpquota 0 1

# Remount and initialize quota databases
mount -o remount /
quotacheck -cmug /

# Turn on quotas
quotaon -v /

# Set quota for a user (soft: 5GB, hard: 6GB)
setquota -u username 5000000 6000000 0 0 /
```

#### Set up disk monitoring alerts
```bash
# Create a simple disk space alert script
cat > /usr/local/bin/disk_alert.sh << 'EOF'
#!/bin/bash
THRESHOLD=90
MAILTO="admin@example.com"

df -h | grep -vE '^Filesystem|tmpfs|cdrom' | awk '{ print $5 " " $1 }' | while read -r output;
do
  usep=$(echo "$output" | awk '{ print $1}' | cut -d'%' -f1)
  partition=$(echo "$output" | awk '{ print $2 }')
  if [ $usep -ge $THRESHOLD ]; then
    echo "WARNING: Running out of space \"$partition ($usep%)\" on $(hostname) as on $(date)" | 
    mail -s "Disk Space Alert: $usep% on $(hostname)" "$MAILTO"
  fi
done
EOF
chmod +x /usr/local/bin/disk_alert.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 * * * * /usr/local/bin/disk_alert.sh") | crontab -
```




## Networking Scenarios

### Network Configuration and Troubleshooting

#### Quickly check interface status
```bash
# Show all interfaces and their status
ip -br addr show

# Show specific interface details
ip addr show dev eth0

# Check interface statistics
ip -s link show dev eth0
```

#### Configure a static IP address
```bash
# Temporary static IP (until reboot)
ip addr add 192.168.1.100/24 dev eth0
ip route add default via 192.168.1.1

# Using NetworkManager (permanent)
nmcli con mod "Wired connection 1" ipv4.addresses 192.168.1.100/24 ipv4.gateway 192.168.1.1 ipv4.dns "8.8.8.8,8.8.4.4" ipv4.method manual
nmcli con up "Wired connection 1"

# For Debian/Ubuntu (permanent via interfaces file)
cat > /etc/network/interfaces.d/eth0 << EOF
auto eth0
iface eth0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 8.8.8.8 8.8.4.4
EOF
systemctl restart networking

# For RHEL/CentOS (permanent via ifcfg file)
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE=eth0
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
DNS1=8.8.8.8
DNS2=8.8.4.4
ONBOOT=yes
TYPE=Ethernet
EOF
systemctl restart network
```

#### Set up network bonding (link aggregation)
```bash
# Create a bond interface (RHEL/CentOS)
cat > /etc/sysconfig/network-scripts/ifcfg-bond0 << EOF
DEVICE=bond0
TYPE=Bond
NAME=bond0
BONDING_MASTER=yes
BOOTPROTO=static
IPADDR=192.168.1.100
NETMASK=255.255.255.0
GATEWAY=192.168.1.1
BONDING_OPTS="mode=802.3ad miimon=100"
ONBOOT=yes
EOF

# Configure slave interfaces
cat > /etc/sysconfig/network-scripts/ifcfg-eth0 << EOF
DEVICE=eth0
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
ONBOOT=yes
EOF

cat > /etc/sysconfig/network-scripts/ifcfg-eth1 << EOF
DEVICE=eth1
TYPE=Ethernet
MASTER=bond0
SLAVE=yes
BOOTPROTO=none
ONBOOT=yes
EOF

# Ensure bonding module is loaded
echo "bonding" > /etc/modules-load.d/bonding.conf

# Restart networking
systemctl restart network
```

#### Configure a bridge interface
```bash
# Create a bridge (Debian/Ubuntu)
apt-get install bridge-utils
cat > /etc/network/interfaces.d/br0 << EOF
auto br0
iface br0 inet static
    address 192.168.1.100
    netmask 255.255.255.0
    gateway 192.168.1.1
    bridge_ports eth0
    bridge_stp on
    bridge_fd 0
EOF
systemctl restart networking

# Using NetworkManager
nmcli con add type bridge ifname br0
nmcli con mod bridge-br0 ipv4.addresses 192.168.1.100/24 ipv4.method manual ipv4.gateway 192.168.1.1
nmcli con add type ethernet slave-type bridge con-name br0-port1 ifname eth0 master br0
nmcli con up bridge-br0
```

#### Network troubleshooting sequence
```bash
# Check interface status
ip addr

# Check routing table
ip route
route -n

# Check DNS resolution
nslookup google.com
dig google.com

# Check default gateway connectivity
ping -c4 $(ip route | grep default | awk '{print $3}')

# Check external connectivity
ping -c4 8.8.8.8

# Check ports in use
ss -tuln

# Check for listening services
netstat -tulnp

# Trace route to destination
traceroute google.com
mtr google.com

# Check for packet loss
ping -c 100 google.com | grep -E 'transmitted|loss'

# Check if firewall is blocking
iptables -L -n -v
firewall-cmd --list-all

# Test specific port connectivity
nc -zv host.example.com 22
telnet host.example.com 80
```

#### Configure network traffic shaping
```bash
# Limit bandwidth on an interface to 1Mbit
tc qdisc add dev eth0 root tbf rate 1mbit burst 32kbit latency 400ms

# Clear traffic control settings
tc qdisc del dev eth0 root

# Prioritize SSH traffic
tc qdisc add dev eth0 root handle 1: prio
tc filter add dev eth0 protocol ip parent 1: prio 1 u32 match ip dport 22 0xffff flowid 1:1
```

#### Port forwarding with iptables
```bash
# Forward external port 8080 to internal 80
iptables -t nat -A PREROUTING -p tcp --dport 8080 -j REDIRECT --to-port 80

# Forward port 80 to internal server
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
iptables -t nat -A POSTROUTING -j MASQUERADE

# Save rules (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4

# Save rules (RHEL/CentOS)
service iptables save
```

#### Set up IP masquerading (NAT)
```bash
# Enable IP forwarding
echo 1 > /proc/sys/net/ipv4/ip_forward
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
sysctl -p

# Set up NAT rules
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT

# Save the rules
iptables-save > /etc/iptables/rules.v4
```

#### Capture and analyze network traffic
```bash
# Basic packet capture
tcpdump -i eth0

# Save output to a file
tcpdump -i eth0 -w capture.pcap

# Filter for specific traffic (example: HTTP)
tcpdump -i eth0 port 80

# Filter by source/destination
tcpdump -i eth0 src 192.168.1.100 and dst 192.168.1.200

# Analyze a specific protocol
tcpdump -i eth0 icmp

# Read a saved capture file
tcpdump -r capture.pcap

# Display verbose output and packet payload
tcpdump -i eth0 -vvv -X port 80
```

### DNS Configuration

#### Configure local DNS resolution
```bash
# Add hosts to /etc/hosts
echo "192.168.1.10 server1.example.com server1" >> /etc/hosts

# Flush DNS cache (systemd-resolved)
systemd-resolve --flush-caches

# Flush DNS cache (nscd)
service nscd restart
```

#### Set up a local DNS server (using dnsmasq)
```bash
# Install dnsmasq
apt-get install dnsmasq

# Configure dnsmasq
cat > /etc/dnsmasq.conf << EOF
# Listen on this interface
interface=eth0
# Don't forward short names
domain-needed
# Don't forward addresses in non-routed address spaces
bogus-priv
# Use Google DNS
server=8.8.8.8
server=8.8.4.4
# Local domain
local=/example.local/
domain=example.local
# DHCP range
dhcp-range=192.168.1.50,192.168.1.150,12h
# Add some fixed hosts
dhcp-host=00:11:22:33:44:55,server1,192.168.1.10
# Set gateway
dhcp-option=3,192.168.1.1
# Set DNS server
dhcp-option=6,192.168.1.1
EOF

# Start dnsmasq
systemctl enable dnsmasq
systemctl restart dnsmasq
```

#### DNS lookup tools
```bash
# DNS lookup using different tools
dig example.com
host example.com
nslookup example.com

# Query specific record types
dig MX example.com
dig TXT example.com
dig NS example.com

# Query specific DNS server
dig @8.8.8.8 example.com

# Reverse DNS lookup
dig -x 8.8.8.8
```

## Security Scenarios

### Firewall Configuration

#### Configure firewalld (RHEL/CentOS/Fedora)
```bash
# Install firewalld if not already installed
dnf install firewalld

# Start and enable firewalld
systemctl enable firewalld
systemctl start firewalld

# Check status
firewall-cmd --state

# View current configuration
firewall-cmd --list-all

# Add a service
firewall-cmd --permanent --add-service=http

# Add a port
firewall-cmd --permanent --add-port=8080/tcp

# Configure port forwarding
firewall-cmd --permanent --add-forward-port=port=80:proto=tcp:toport=8080

# Configure masquerading
firewall-cmd --permanent --add-masquerade

# Create a custom service
cat > /etc/firewalld/services/myapp.xml << EOF
<?xml version="1.0" encoding="utf-8"?>
<service>
  <short>MyApp</short>
  <description>My custom application</description>
  <port protocol="tcp" port="12345"/>
</service>
EOF
firewall-cmd --permanent --add-service=myapp

# Reload firewall to apply changes
firewall-cmd --reload
```

#### Configure UFW (Ubuntu/Debian)
```bash
# Install UFW if not already installed
apt-get install ufw

# Set default policies
ufw default deny incoming
ufw default allow outgoing

# Allow specific services
ufw allow ssh
ufw allow http
ufw allow https

# Allow specific ports
ufw allow 8080/tcp

# Allow from specific IP address
ufw allow from 192.168.1.100

# Allow specific IP to access a specific port
ufw allow from 192.168.1.100 to any port 22

# Rate limiting (brute force protection)
ufw limit ssh

# Enable UFW
ufw enable

# Check status
ufw status verbose
```

#### Configure iptables directly
```bash
# Flush existing rules
iptables -F
iptables -X
iptables -t nat -F
iptables -t nat -X
iptables -t mangle -F
iptables -t mangle -X

# Set default chain policies
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback traffic
iptables -A INPUT -i lo -j ACCEPT
iptables -A OUTPUT -o lo -j ACCEPT

# Allow established and related connections
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH
iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# Allow HTTP and HTTPS
iptables -A INPUT -p tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp --dport 443 -j ACCEPT

# Allow ICMP (ping)
iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT

# Log dropped packets
iptables -A INPUT -j LOG --log-prefix "IPTables-Dropped: " --log-level 4

# Save rules (Debian/Ubuntu)
iptables-save > /etc/iptables/rules.v4

# Save rules (RHEL/CentOS)
service iptables save
```

### Securing SSH

#### Secure SSH configuration
```bash
# Backup the original configuration
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak

# Edit SSH configuration
cat > /etc/ssh/sshd_config.d/secure.conf << EOF
# Disable password authentication
PasswordAuthentication no

# Allow only key-based authentication
PubkeyAuthentication yes

# Disable root login
PermitRootLogin no

# Allow only specific users
AllowUsers user1 user2

# Allow only specific groups
AllowGroups sshusers wheel

# Limit login attempts
MaxAuthTries 3

# Change SSH port (optional)
Port 2222

# Disable empty passwords
PermitEmptyPasswords no

# Disconnect when idle
ClientAliveInterval 300
ClientAliveCountMax 2

# Disable X11 forwarding
X11Forwarding no

# Use strong ciphers and MACs
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
EOF

# Check configuration syntax
sshd -t

# Restart SSH service
systemctl restart sshd
```

#### Set up SSH key authentication
```bash
# Generate SSH key pair (on client)
ssh-keygen -t ed25519 -f ~/.ssh/mykey -C "my-ssh-key"

# Copy public key to server (interactive method)
ssh-copy-id -i ~/.ssh/mykey.pub user@server

# Copy public key to server (manual method)
cat ~/.ssh/mykey.pub | ssh user@server "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"

# Configure the client to use the key
cat >> ~/.ssh/config << EOF
Host myserver
    HostName server.example.com
    User myuser
    Port 22
    IdentityFile ~/.ssh/mykey
    IdentitiesOnly yes
EOF
chmod 600 ~/.ssh/config
```

#### Set up SSH with 2FA
```bash
# Install Google Authenticator
apt-get install libpam-google-authenticator

# Configure PAM
echo "auth required pam_google_authenticator.so" >> /etc/pam.d/sshd

# Update SSH config
sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/' /etc/ssh/sshd_config
sed -i 's/UsePAM no/UsePAM yes/' /etc/ssh/sshd_config
echo "AuthenticationMethods publickey,keyboard-interactive" >> /etc/ssh/sshd_config

# Restart SSH
systemctl restart sshd

# Setup for user (run as the user)
google-authenticator
```

### Intrusion Detection and Prevention

#### Configure Fail2Ban
```bash
# Install Fail2Ban
apt-get install fail2ban

# Create a custom jail for SSH
cat > /etc/fail2ban/jail.d/sshd.conf << EOF
[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 3600
findtime = 600
EOF

# Create custom jail for Apache
cat > /etc/fail2ban/jail.d/apache.conf << EOF
[apache]
enabled = true
port = http,https
filter = apache-auth
logpath = /var/log/apache2/error.log
maxretry = 3
bantime = 3600
EOF

# Restart Fail2Ban
systemctl restart fail2ban

# Check status
fail2ban-client status
fail2ban-client status sshd
```

#### Set up rootkit scanning with rkhunter
```bash
# Install rkhunter
apt-get install rkhunter

# Update database
rkhunter --update

# Initial system scan
rkhunter --propupd
rkhunter --check

# Setup automatic daily scans
cat > /etc/cron.daily/rkhunter-scan << EOF
#!/bin/bash
/usr/bin/rkhunter --check --skip-keypress --report-warnings-only | mail -s "RKHunter Scan $(date)" admin@example.com
EOF
chmod +x /etc/cron.daily/rkhunter-scan
```

#### Audit system with Lynis
```bash
# Install Lynis
apt-get install lynis

# Run an audit
lynis audit system

# Create a scheduled audit
cat > /etc/cron.weekly/lynis-audit << EOF
#!/bin/bash
/usr/bin/lynis audit system --cronjob > /var/log/lynis-report.$(date +%Y%m%d).log
EOF
chmod +x /etc/cron.weekly/lynis-audit
```

## SSL/TLS Certificate Management

### Self-Signed Certificates

#### Create a self-signed SSL certificate
```bash
# Generate private key and certificate
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/server.key -out /etc/ssl/certs/server.crt -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Set proper permissions
chmod 600 /etc/ssl/private/server.key
chmod 644 /etc/ssl/certs/server.crt
```

#### Create a certificate with Subject Alternative Names (SAN)
```bash
# Create OpenSSL config
cat > openssl.cnf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = State
L = City
O = Organization
CN = example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = mail.example.com
IP.1 = 192.168.1.10
EOF

# Generate certificate with SANs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt -config openssl.cnf -extensions v3_req

# Verify the certificate
openssl x509 -in server.crt -text -noout
```

### Lets Encrypt Certificates

#### Set up certbot and obtain certificates
```bash
# Install certbot (Debian/Ubuntu)
apt-get install certbot

# For Apache
apt-get install python3-certbot-apache

# For Nginx
apt-get install python3-certbot-nginx

# Obtain certificate with Apache plugin
certbot --apache -d example.com -d www.example.com

# Obtain certificate with Nginx plugin
certbot --nginx -d example.com -d www.example.com

# Obtain certificate standalone (stop web server first)
certbot certonly --standalone -d example.com -d www.example.com

# Obtain certificate using webroot
certbot certonly --webroot -w /var/www/html -d example.com -d www.example.com

# Test auto-renewal
certbot renew --dry-run
```

### Certificate Management

#### Verify and test certificates
```bash
# Check certificate information
openssl x509 -in certificate.crt -text -noout

# Verify certificate chain
openssl verify -CAfile chain.pem certificate.crt

# Check certificate expiration
openssl x509 -in certificate.crt -noout -enddate

# Check SSL/TLS of a website
openssl s_client -connect example.com:443 -servername example.com

# Quick script to check expiration of multiple domains
cat > check_ssl_expiry.sh << 'EOF'
#!/bin/bash
for domain in example.com www.example.org mail.example.net; do
  echo -n "$domain: "
  echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
  openssl x509 -noout -enddate | cut -d= -f2
done
EOF
chmod +x check_ssl_expiry.sh
```

#### Convert certificate formats
```bash
# Convert PEM to DER
openssl x509 -in cert.pem -outform der -out cert.der

# Convert DER to PEM
openssl x509 -in cert.der -inform der -outform pem -out cert.pem

# Convert PEM to PKCS#12 (PFX)
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -certfile ca-chain.crt

# Convert PKCS#12 (PFX) to PEM
openssl pkcs12 -in certificate.pfx -out certificate.pem -nodes
```

#### Create a Certificate Signing Request (CSR)
```bash
# Generate private key
openssl genrsa -out private.key 2048

# Create CSR
openssl req -new -key private.key -out request.csr -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# Create CSR with SANs using config file
cat > csr.cnf << EOF
[req]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no

[req_distinguished_name]
C = US
ST = State
L = City
O = Organization
CN = example.com

[v3_req]
keyUsage = keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = example.com
DNS.2 = www.example.com
DNS.3 = mail.example.com
EOF

openssl req -new -key private.key -out request.csr -config csr.cnf
```

## System Monitoring Scenarios

### Performance Monitoring

#### Monitor system load and resources
```bash
# Monitor CPU, memory, disk I/O and network in real-time
dstat -cdngy

# Monitor top processes
htop

# Watch specific process resource usage
ps -p $(pgrep -f process_name) -o %cpu,%mem,cmd

# Continuous monitoring of specific process
watch -n 1 "ps -p $(pgrep -f process_name) -o %cpu,%mem,cmd"

# Monitor disk I/O
iostat -xz 5

# Watch network traffic
iftop -i eth0

# Watch overall system performance
glances
```

#### Create a system resource monitoring script
```bash
cat > /usr/local/bin/system_monitor.sh << 'EOF'
#!/bin/bash

echo "=== System Monitor Report ==="
date
echo

echo "=== System Load ==="
uptime
echo

echo "=== Memory Usage ==="
free -h
echo

echo "=== Disk Usage ==="
df -h | grep -v "tmpfs\|udev"
echo

echo "=== Top 5 Processes by CPU ==="
ps aux --sort=-%cpu | head -6
echo

echo "=== Top 5 Processes by Memory ==="
ps aux --sort=-%mem | head -6
echo

echo "=== Network Connections ==="
ss -tuln
echo

echo "=== Recent Login Activity ==="
last | head -5
echo

echo "=== System Errors/Warnings ==="
journalctl -p err..emerg --since "1 hour ago" | tail -10
EOF

chmod +x /usr/local/bin/system_monitor.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 * * * * /usr/local/bin/system_monitor.sh > /var/log/system_monitor_\$(date +\%Y\%m\%d_\%H).log") | crontab -
```

#### Setup system load alerts
```bash
cat > /usr/local/bin/load_alert.sh << 'EOF'
#!/bin/bash

THRESHOLD=4
MAILTO="admin@example.com"

LOAD=$(cat /proc/loadavg | cut -d " " -f 1)
LOAD_INT=$(echo "$LOAD * 100" | bc | cut -d "." -f 1)
THRESHOLD_INT=$(echo "$THRESHOLD * 100" | bc | cut -d "." -f 1)

if [ $LOAD_INT -ge $THRESHOLD_INT ]; then
  echo "ALERT: Load average is $LOAD on $(hostname) at $(date)" | 
  mail -s "High Load Alert: $LOAD on $(hostname)" "$MAILTO"
  
  # Append top processes for investigation
  echo -e "\n\nTop processes by CPU usage:" > /tmp/load_report.txt
  ps aux --sort=-%cpu | head -10 >> /tmp/load_report.txt
  echo -e "\n\nTop processes by memory usage:" >> /tmp/load_report.txt
  ps aux --sort=-%mem | head -10 >> /tmp/load_report.txt
  
  mail -s "High Load Process Report for $(hostname)" "$MAILTO" < /tmp/load_report.txt
  rm /tmp/load_report.txt
fi
EOF

chmod +x /usr/local/bin/load_alert.sh
(crontab -l 2>/dev/null; echo "*/5 * * * * /usr/local/bin/load_alert.sh") | crontab -
```

### System Maintenance 

#### Automate system updates
```bash
# Create update script (Debian/Ubuntu)
cat > /usr/local/bin/auto_update.sh << 'EOF'
#!/bin/bash
LOG="/var/log/auto_update.log"

echo "=== Auto Update Run: $(date) ===" >> $LOG
apt-get update >> $LOG 2>&1
apt-get -y -d upgrade >> $LOG 2>&1
apt-get -y --with-new-pkgs upgrade >> $LOG 2>&1
apt-get -y autoremove >> $LOG 2>&1
apt-get clean >> $LOG 2>&1
EOF

chmod +x /usr/local/bin/auto_update.sh

# Add to crontab (run at 3am every Sunday)
(crontab -l 2>/dev/null; echo "0 3 * * 0 /usr/local/bin/auto_update.sh") | crontab -

# Alternative for RHEL/CentOS
cat > /usr/local/bin/auto_update.sh << 'EOF'
#!/bin/bash
LOG="/var/log/auto_update.log"

echo "=== Auto Update Run: $(date) ===" >> $LOG
dnf -y update >> $LOG 2>&1
EOF

chmod +x /usr/local/bin/auto_update.sh
(crontab -l 2>/dev/null; echo "0 3 * * 0 /usr/local/bin/auto_update.sh") | crontab -
```

#### Schedule system maintenance tasks
```bash
# Create a maintenance script
cat > /usr/local/bin/maintenance.sh << 'EOF'
#!/bin/bash
LOG="/var/log/maintenance.log"

echo "=== Maintenance Run: $(date) ===" >> $LOG

# Update package cache
apt-get update >> $LOG 2>&1

# Clean old package files
apt-get clean >> $LOG 2>&1
apt-get autoremove -y >> $LOG 2>&1

# Clean journal logs
journalctl --vacuum-time=7d >> $LOG 2>&1

# Clean temp files
find /tmp -type f -atime +10 -delete
find /var/tmp -type f -atime +10 -delete

# Clean old logs
find /var/log -name "*.gz" -type f -mtime +30 -delete

# Update locate database
updatedb >> $LOG 2>&1

# Check disk usage
df -h >> $LOG 2>&1

# Check for failed systemd services
systemctl --failed >> $LOG 2>&1

# Run SMART tests on all disks
for disk in $(lsblk -d -o name | grep -v NAME); do
  smartctl -t short /dev/$disk >> $LOG 2>&1
done

echo "=== Maintenance Completed: $(date) ===" >> $LOG
EOF

chmod +x /usr/local/bin/maintenance.sh

# Add to crontab (run at 4am every Saturday)
(crontab -l 2>/dev/null; echo "0 4 * * 6 /usr/local/bin/maintenance.sh") | crontab -
```

## Database Server Management 

### MySQL/MariaDB Administration 

#### MySQL performance tuning
```bash
# Create a MySQL configuration with performance optimizations
cat > /etc/mysql/conf.d/performance.cnf << EOF
[mysqld]
# InnoDB settings
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
innodb_flush_method = O_DIRECT
innodb_flush_log_at_trx_commit = 2
innodb_file_per_table = 1

# Query cache settings
query_cache_type = 1
query_cache_size = 32M
query_cache_limit = 1M

# Connection settings
max_connections = 500
thread_cache_size = 32
table_open_cache = 1024

# Temp tables
tmp_table_size = 64M
max_heap_table_size = 64M

# Default character set
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
EOF

# Restart MySQL service
systemctl restart mysql
```

#### MySQL security hardening
```bash
# Run MySQL secure installation
mysql_secure_installation

# Create limited privilege user
mysql -u root -p -e "CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';"
mysql -u root -p -e "GRANT SELECT, INSERT, UPDATE, DELETE ON app_database.* TO 'app_user'@'localhost';"
mysql -u root -p -e "FLUSH PRIVILEGES;"

# Restrict to localhost only
mysql -u root -p -e "UPDATE mysql.user SET Host='localhost' WHERE User='root';"
mysql -u root -p -e "FLUSH PRIVILEGES;"

# Remove anonymous users
mysql -u root -p -e "DELETE FROM mysql.user WHERE User='';"
mysql -u root -p -e "FLUSH PRIVILEGES;"

# Disable remote root login
mysql -u root -p -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');"
mysql -u root -p -e "FLUSH PRIVILEGES;"
```

### PostgreSQL Administration

#### Backup and restore PostgreSQL databases
```bash
# Backup a single database
pg_dump -U postgres -d mydb > mydb.sql

# Backup all databases
pg_dumpall -U postgres > all_databases.sql

# Backup with compression
pg_dump -U postgres -d mydb | gzip > mydb.sql.gz

# Schedule daily backup
cat > /usr/local/bin/postgres_backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/postgresql"
DATE=$(date +%Y%m%d)
mkdir -p $BACKUP_DIR

# Backup all databases
pg_dumpall -U postgres | gzip > "$BACKUP_DIR/all-$DATE.sql.gz"

# Backup each database separately
for DB in $(psql -U postgres -t -c "SELECT datname FROM pg_database WHERE datname NOT IN ('template0', 'template1', 'postgres');"); do
  DB=$(echo $DB | tr -d ' ')
  pg_dump -U postgres -d $DB | gzip > "$BACKUP_DIR/$DB-$DATE.sql.gz"
done

# Delete backups older than 14 days
find $BACKUP_DIR -type f -name "*.sql.gz" -mtime +14 -delete
EOF

chmod +x /usr/local/bin/postgres_backup.sh
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/postgres_backup.sh") | crontab -

# Restore a database
psql -U postgres -d mydb < mydb.sql

# Restore from a compressed backup
gunzip < mydb.sql.gz | psql -U postgres -d mydb

# Create a database before restore
createdb -U postgres mydb
gunzip < mydb.sql.gz | psql -U postgres -d mydb
```

#### PostgreSQL performance tuning
```bash
# Show current PostgreSQL configuration
psql -U postgres -c "SHOW ALL;"

# Create a PostgreSQL configuration with performance optimizations
cat > /etc/postgresql/13/main/conf.d/performance.conf << EOF
# Memory settings
shared_buffers = 1GB
work_mem = 32MB
maintenance_work_mem = 256MB
effective_cache_size = 3GB

# Write-ahead log settings
wal_buffers = 16MB
synchronous_commit = off
checkpoint_timeout = 15min
checkpoint_completion_target = 0.9

# Planning and statistics
random_page_cost = 1.1
effective_io_concurrency = 200
default_statistics_target = 100

# Connection settings
max_connections = 100
EOF

# Restart PostgreSQL service
systemctl restart postgresql
```

## Web Server Management

### Apache Configuration

#### Set up a basic Apache virtual host
```bash
# Install Apache
apt-get install apache2

# Create directory structure
mkdir -p /var/www/example.com/public_html
chown -R www-data:www-data /var/www/example.com

# Create a basic virtual host file
cat > /etc/apache2/sites-available/example.com.conf << EOF
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    ErrorLog \${APACHE_LOG_DIR}/example.com_error.log
    CustomLog \${APACHE_LOG_DIR}/example.com_access.log combined
    
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

# Enable virtual host and rewrite module
a2ensite example.com.conf
a2enmod rewrite

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

#### Configure SSL with Apache
```bash
# Enable SSL module
a2enmod ssl

# Create SSL virtual host
cat > /etc/apache2/sites-available/example.com-ssl.conf << EOF
<VirtualHost *:443>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    ErrorLog \${APACHE_LOG_DIR}/example.com_error.log
    CustomLog \${APACHE_LOG_DIR}/example.com_access.log combined
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/example.com.key
    SSLCertificateChainFile /etc/ssl/certs/example.com-chain.crt
    
    <Directory /var/www/example.com/public_html>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    # HTTP Strict Transport Security
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</VirtualHost>
EOF

# Enable headers module and the site
a2enmod headers
a2ensite example.com-ssl.conf

# Add HTTP to HTTPS redirect
cat > /etc/apache2/sites-available/example.com.conf << EOF
<VirtualHost *:80>
    ServerAdmin webmaster@example.com
    ServerName example.com
    ServerAlias www.example.com
    DocumentRoot /var/www/example.com/public_html
    
    Redirect permanent / https://example.com/
</VirtualHost>
EOF

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

#### Configure Apache for PHP
```bash
# Install PHP and necessary modules
apt-get install php php-mysql libapache2-mod-php php-cli php-common php-gd php-xmlrpc php-mbstring php-curl

# Create PHP info file for testing
echo "<?php phpinfo(); ?>" > /var/www/html/info.php

# Create a custom PHP configuration
cat > /etc/php/7.4/apache2/conf.d/custom.ini << EOF
; Basic Settings
max_execution_time = 300
memory_limit = 128M
upload_max_filesize = 20M
post_max_size = 21M
max_input_vars = 3000

; Error Handling
display_errors = Off
display_startup_errors = Off
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
log_errors = On
error_log = /var/log/php_errors.log

; Performance Settings
opcache.enable = 1
opcache.memory_consumption = 128
opcache.interned_strings_buffer = 8
opcache.max_accelerated_files = 4000
opcache.revalidate_freq = 60
EOF

# Restart Apache
systemctl restart apache2
```

#### Apache performance tuning
```bash
# Install Apache utilities
apt-get install apache2-utils

# Create MPM configuration for better performance
cat > /etc/apache2/mods-available/mpm_event.conf << EOF
<IfModule mpm_event_module>
    StartServers             2
    MinSpareThreads         25
    MaxSpareThreads         75
    ThreadLimit             64
    ThreadsPerChild         25
    MaxRequestWorkers      150
    MaxConnectionsPerChild   0
</IfModule>
EOF

# Enable event MPM and disable prefork
a2dismod mpm_prefork
a2enmod mpm_event

# Enable HTTP/2
a2enmod http2
cat > /etc/apache2/conf-available/http2.conf << EOF
Protocols h2 http/1.1
EOF
a2enconf http2

# Enable cache modules
a2enmod expires
a2enmod cache
a2enmod cache_disk

# Create cache configuration
cat > /etc/apache2/conf-available/cache.conf << EOF
<IfModule mod_cache.c>
    CacheRoot /var/cache/apache2/mod_cache_disk
    CacheEnable disk /
    CacheDirLevels 2
    CacheDirLength 1
    CacheMaxExpire 86400
    CacheIgnoreNoLastMod On
    CacheIgnoreCacheControl On
    CacheDefaultExpire 3600
</IfModule>
EOF
mkdir -p /var/cache/apache2/mod_cache_disk
chown -R www-data:www-data /var/cache/apache2/mod_cache_disk
a2enconf cache

# Test configuration and restart
apache2ctl configtest
systemctl restart apache2
```

### Nginx Configuration

#### Set up a basic Nginx virtual host
```bash
# Install Nginx
apt-get install nginx

# Create directory structure
mkdir -p /var/www/example.com/public_html
chown -R www-data:www-data /var/www/example.com

# Create a basic virtual host file
cat > /etc/nginx/sites-available/example.com << EOF
server {
    listen 80;
    listen [::]:80;
    
    server_name example.com www.example.com;
    root /var/www/example.com/public_html;
    index index.html index.htm index.php;
    
    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }
    
    # Pass PHP scripts to PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
    
    # Logging
    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
EOF

# Enable the site
ln -s /etc/nginx/sites-available/example.com /etc/nginx/sites-enabled/

# Test configuration and restart
nginx -t
systemctl restart nginx
```

#### Configure SSL with Nginx
```bash
# Create SSL virtual host
cat > /etc/nginx/sites-available/example.com << EOF
# HTTP to HTTPS redirect
server {
    listen 80;
    listen [::]:80;
    server_name example.com www.example.com;
    return 301 https://\$host\$request_uri;
}

# HTTPS server block
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name example.com www.example.com;
    
    root /var/www/example.com/public_html;
    index index.html index.htm index.php;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/example.com.crt;
    ssl_certificate_key /etc/ssl/private/example.com.key;
    ssl_trusted_certificate /etc/ssl/certs/example.com-chain.crt;
    
    # SSL optimizations
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers on;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # OCSP Stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
    
    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }
    
    # Pass PHP scripts to PHP-FPM
    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
    }
    
    # Deny access to hidden files
    location ~ /\. {
        deny all;
    }
    
    # Logging
    access_log /var/log/nginx/example.com_access.log;
    error_log /var/log/nginx/example.com_error.log;
}
EOF

# Test configuration and restart
nginx -t
systemctl restart nginx
```

#### Nginx performance tuning
```bash
# Create optimized Nginx configuration
cat > /etc/nginx/nginx.conf << EOF
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
    multi_accept on;
    use epoll;
}

http {
    # Basic Settings
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    server_tokens off;
    
    # Buffer size for POST submissions
    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 8m;
    large_client_header_buffers 2 1k;
    
    # Timeouts
    client_body_timeout 12;
    client_header_timeout 12;
    send_timeout 10;
    
    # File type handling
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # Logging Settings
    access_log /var/log/nginx/access.log;
    error_log /var/log/nginx/error.log;
    
    # Gzip Settings
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    
    # Security headers
    add_header X-Frame-Options SAMEORIGIN;
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    
    # Virtual Host Configs
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
    
    # Open file cache
    open_file_cache max=2000 inactive=20s;
    open_file_cache_valid 60s;
    open_file_cache_min_uses 5;
    open_file_cache_errors off;
}
EOF

# Test configuration and restart
nginx -t
systemctl restart nginx
```

#### Configure Nginx as reverse proxy
```bash
# Create a reverse proxy configuration for an application
cat > /etc/nginx/sites-available/app-proxy << EOF
server {
    listen 80;
    server_name app.example.com;
    
    # Redirect to HTTPS
    return 301 https://\$host\$request_uri;
}

server {
    listen 443 ssl http2;
    server_name app.example.com;
    
    # SSL Configuration
    ssl_certificate /etc/ssl/certs/app.example.com.crt;
    ssl_certificate_key /etc/ssl/private/app.example.com.key;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    
    # Proxy settings
    location / {
        proxy_pass http://localhost:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
        proxy_cache_bypass \$http_upgrade;
    }
    
    # Logging
    access_log /var/log/nginx/app.example.com_access.log;
    error_log /var/log/nginx/app.example.com_error.log;
}
EOF

# Enable the site
ln -s /etc/nginx/sites-available/app-proxy /etc/nginx/sites-enabled/

# Test configuration and restart
nginx -t
systemctl restart nginx
```

## Automation and Scripting Scenarios

### Backup Automation

#### Create a full backup script
```bash
cat > /usr/local/bin/full_backup.sh << 'EOF'
#!/bin/bash
#
# Full Backup Script for Linux Server
#

# Configuration
BACKUP_DIR="/mnt/backup/$(hostname)/$(date +%Y-%m-%d)"
LOG_FILE="/var/log/backup/backup_$(date +%Y-%m-%d).log"
EXCLUDE_FILE="/etc/backup/exclude.list"
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

# Make sure backup and log directories exist
mkdir -p "$BACKUP_DIR"
mkdir -p "$(dirname "$LOG_FILE")"

# Start logging
exec > >(tee -a "$LOG_FILE") 2>&1
echo "==== Backup Started at $TIMESTAMP ===="

# Check for backup disk space
BACKUP_SPACE=$(df -h "$BACKUP_DIR" | awk 'NR==2 {print $4}')
echo "Available space for backup: $BACKUP_SPACE"

# Dump all databases
echo "Backing up MySQL databases..."
mkdir -p "$BACKUP_DIR/mysql"

# MySQL backup (with password in my.cnf)
if command -v mysqldump &> /dev/null; then
  if mysql --defaults-file=/root/.my.cnf -e "SHOW DATABASES;" > /dev/null 2>&1; then
    for DB in $(mysql --defaults-file=/root/.my.cnf -e "SHOW DATABASES;" | grep -v "Database\|information_schema\|performance_schema"); do
      echo "  - Backing up database: $DB"
      mysqldump --defaults-file=/root/.my.cnf --single-transaction --quick --lock-tables=false "$DB" | gzip > "$BACKUP_DIR/mysql/$DB.sql.gz"
    done
  else
    echo "ERROR: MySQL connection failed."
  fi
else
  echo "INFO: MySQL not installed, skipping database backup."
fi

# PostgreSQL backup
if command -v pg_dump &> /dev/null; then
  echo "Backing up PostgreSQL databases..."
  mkdir -p "$BACKUP_DIR/postgresql"
  
  if su - postgres -c "psql -l" > /dev/null 2>&1; then
    for DB in $(su - postgres -c "psql -t -c \"SELECT datname FROM pg_database WHERE datname NOT IN ('template0', 'template1', 'postgres');\"" | tr -d ' '); do
      echo "  - Backing up PostgreSQL database: $DB"
      su - postgres -c "pg_dump $DB" | gzip > "$BACKUP_DIR/postgresql/$DB.sql.gz"
    done
  else
    echo "ERROR: PostgreSQL connection failed."
  fi
else
  echo "INFO: PostgreSQL not installed, skipping database backup."
fi

# Configuration files backup
echo "Backing up configuration files..."
mkdir -p "$BACKUP_DIR/config"
tar -czf "$BACKUP_DIR/config/etc.tar.gz" /etc 2>/dev/null

# Home directories backup
echo "Backing up home directories..."
if [ -f "$EXCLUDE_FILE" ]; then
  tar --exclude-from="$EXCLUDE_FILE" -czf "$BACKUP_DIR/home.tar.gz" /home
else
  echo "WARNING: Exclude file not found at $EXCLUDE_FILE. Backing up all files."
  tar -czf "$BACKUP_DIR/home.tar.gz" /home
fi

# Web data backup
if [ -d "/var/www" ]; then
  echo "Backing up web data..."
  tar -czf "$BACKUP_DIR/www.tar.gz" /var/www
fi

# Application data backup
for APP_DIR in /opt/*; do
  if [ -d "$APP_DIR" ]; then
    APP_NAME=$(basename "$APP_DIR")
    echo "Backing up application data: $APP_NAME"
    tar -czf "$BACKUP_DIR/app_${APP_NAME}.tar.gz" "$APP_DIR"
  fi
done

# Create a file list for reference
echo "Creating file list..."
find /etc /home /var/www /opt -type f -exec stat -c "%y %s %n" {} \; | sort > "$BACKUP_DIR/file_list.txt"

# Calculate checksum for backup verification
echo "Calculating checksums..."
find "$BACKUP_DIR" -type f -name "*.tar.gz" -o -name "*.sql.gz" | xargs md5sum > "$BACKUP_DIR/checksums.md5"

# Set correct permissions
chmod -R 600 "$BACKUP_DIR"

# Clean old backups (keep last 7 days)
find "$(dirname "$BACKUP_DIR")" -maxdepth 1 -type d -mtime +7 -exec rm -rf {} \;

# End logging
TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")
echo "==== Backup Completed at $TIMESTAMP ===="
echo "Backup stored in: $BACKUP_DIR"
echo "Log file: $LOG_FILE"

# Check backup size
BACKUP_SIZE=$(du -sh "$BACKUP_DIR" | cut -f1)
echo "Total backup size: $BACKUP_SIZE"
EOF

chmod +x /usr/local/bin/full_backup.sh

# Create exclude file
mkdir -p /etc/backup
cat > /etc/backup/exclude.list << EOF
*.tmp
*.temp
*.cache
*/cache/*
*/tmp/*
*/.Trash/*
*/node_modules/*
*/vendor/*
*.log
EOF

# Add to crontab (run at 1am every Sunday)
(crontab -l 2>/dev/null; echo "0 1 * * 0 /usr/local/bin/full_backup.sh") | crontab -
```

### System Initialization and Bootstrap

#### Create a new system initialization script
```bash
cat > /usr/local/bin/init_system.sh << 'EOF'
#!/bin/bash
#
# System Initialization Script
#

set -e
LOG_FILE="/var/log/system_init.log"
exec > >(tee -a "$LOG_FILE") 2>&1

echo "=== Starting System Initialization: $(date) ==="

# Update system and install basic packages
echo "Updating system packages..."
apt-get update
apt-get -y upgrade

echo "Installing essential packages..."
apt-get -y install vim htop tmux git curl wget nmon iotop net-tools dnsutils \
    unzip zip tar gzip bzip2 apt-transport-https ca-certificates gnupg-agent \
    software-properties-common sudo fail2ban ufw

# Configure timezone
echo "Setting timezone to UTC..."
timedatectl set-timezone UTC

# Configure hostname
HOSTNAME="server-$(hostname -I | awk '{print $1}' | tr '.' '-')"
echo "Setting hostname to: $HOSTNAME"
hostnamectl set-hostname "$HOSTNAME"
echo "127.0.0.1 $HOSTNAME" >> /etc/hosts

# Configure firewall
echo "Configuring firewall..."
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw allow http
ufw allow https
ufw --force enable
echo "Firewall status:"
ufw status

# Configure fail2ban
echo "Configuring fail2ban..."
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sed -i 's/bantime  = 10m/bantime  = 1h/' /etc/fail2ban/jail.local
sed -i 's/maxretry = 5/maxretry = 3/' /etc/fail2ban/jail.local
systemctl enable fail2ban
systemctl restart fail2ban

# Set up automatic updates
echo "Setting up automatic security updates..."
apt-get -y install unattended-upgrades
cat > /etc/apt/apt.conf.d/20auto-upgrades << EOCFG
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
APT::Periodic::AutocleanInterval "7";
EOCFG

cat > /etc/apt/apt.conf.d/50unattended-upgrades << EOCFG
Unattended-Upgrade::Allowed-Origins {
    "\${distro_id}:\${distro_codename}";
    "\${distro_id}:\${distro_codename}-security";
    "\${distro_id}ESMApps:\${distro_codename}-apps-security";
    "\${distro_id}ESM:\${distro_codename}-infra-security";
};
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "true";
Unattended-Upgrade::Automatic-Reboot-Time "02:00";
EOCFG

# Create admin user
echo "Creating admin user..."
useradd -m -s /bin/bash -G sudo admin
mkdir -p /home/admin/.ssh
chmod 700 /home/admin/.ssh

# Add your SSH public key
echo "Adding SSH public key for admin user..."
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDZ..." > /home/admin/.ssh/authorized_keys
chmod 600 /home/admin/.ssh/authorized_keys
chown -R admin:admin /home/admin/.ssh

# Set random password for admin user
ADMIN_PASSWORD=$(openssl rand -base64 12)
echo "admin:${ADMIN_PASSWORD}" | chpasswd
echo "Admin user created with password: ${ADMIN_PASSWORD}"
echo "Please change this password immediately after login!"

# Secure SSH
echo "Securing SSH configuration..."
cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sed -i 's/#PermitRootLogin yes/PermitRootLogin no/' /etc/ssh/sshd_config
sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
systemctl restart sshd

# Set up system monitoring
echo "Setting up basic system monitoring..."
cat > /usr/local/bin/system_monitor.sh << EOSCRIPT
#!/bin/bash
LOG_FILE="/var/log/sysmon/\$(date +%Y%m%d).log"
mkdir -p "\$(dirname \$LOG_FILE)"

{
    echo "===== System Status Report: \$(date) ====="
    echo ""
    echo "=== System Load ==="
    uptime
    echo ""
    echo "=== Memory Usage ==="
    free -h
    echo ""
    echo "=== Disk Usage ==="
    df -h
    echo ""
    echo "=== Network Connections ==="
    ss -tulpn | grep LISTEN
    echo ""
    echo "=== Recent Auth Activity ==="
    grep "session opened" /var/log/auth.log | tail -10
    echo ""
} >> "\$LOG_FILE"
EOSCRIPT

chmod +x /usr/local/bin/system_monitor.sh
(crontab -l 2>/dev/null; echo "0 * * * * /usr/local/bin/system_monitor.sh") | crontab -

# Create setup complete flag
touch /var/log/system_init_complete

echo "=== System Initialization Completed: $(date) ==="
echo "Please reboot the system to apply all changes."
EOF

chmod +x /usr/local/bin/init_system.sh
```

## Container Management 

### Docker Operations 

#### Create a Docker Compose setup
```bash
# Create directory for project
mkdir -p ~/docker/webapp
cd ~/docker/webapp

# Create docker-compose.yml file
cat > docker-compose.yml << EOF
version: '3'

services:
  web:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./html:/usr/share/nginx/html
      - ./nginx/conf.d:/etc/nginx/conf.d
    depends_on:
      - app
    restart: always

  app:
    image: php:7.4-fpm
    volumes:
      - ./html:/var/www/html
    environment:
      - DB_HOST=db
      - DB_NAME=webapp
      - DB_USER=webuser
      - DB_PASS=secret
    depends_on:
      - db
    restart: always

  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=rootsecret
      - MYSQL_DATABASE=webapp
      - MYSQL_USER=webuser
      - MYSQL_PASSWORD=secret
    restart: always

volumes:
  db_data:
EOF

# Create directories for volumes
mkdir -p html nginx/conf.d

# Create a custom nginx configuration
cat > nginx/conf.d/default.conf << EOF
server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.php index.html;

    location / {
        try_files \$uri \$uri/ /index.php?\$args;
    }

    location ~ \.php$ {
        fastcgi_pass app:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        include fastcgi_params;
    }
}
EOF

# Create a test PHP file
cat > html/index.php << EOF
<?php
echo "<h1>Docker Compose Test</h1>";
echo "<p>PHP Version: " . phpversion() . "</p>";

\$host = getenv('DB_HOST');
\$dbname = getenv('DB_NAME');
\$username = getenv('DB_USER');
\$password = getenv('DB_PASS');

try {
    \$conn = new PDO("mysql:host=\$host;dbname=\$dbname", \$username, \$password);
    \$conn->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
    echo "<p>Connected to MySQL successfully!</p>";
} catch(PDOException \$e) {
    echo "<p>Connection failed: " . \$e->getMessage() . "</p>";
}
?>
EOF

# Start the Docker Compose setup
docker-compose up -d

# View logs
docker-compose logs -f

# Stop the Docker Compose setup
docker-compose down

# Stop and remove volumes
docker-compose down -v
```

#### Create a custom Docker image
```bash
# Create a directory for the Dockerfile
mkdir -p ~/docker/custom-app
cd ~/docker/custom-app

# Create a simple Python application
cat > app.py << EOF
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return '<h1>Hello from Custom Docker Container!</h1>'

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=int(os.environ.get('PORT', 5000)))
EOF

# Create requirements.txt
cat > requirements.txt << EOF
flask==2.0.1
EOF

# Create a Dockerfile
cat > Dockerfile << EOF
# Use official Python image
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app.py .

# Set environment variables
ENV PORT=5000
ENV PYTHONUNBUFFERED=1

# Expose the application port
EXPOSE 5000

# Start the application
CMD ["python", "app.py"]
EOF

# Build the Docker image
docker build -t custom-flask-app:latest .

# Run the container
docker run -d -p 5000:5000 --name flask-app custom-flask-app:latest

# Push to Docker Hub (if needed)
docker tag custom-flask-app:latest username/custom-flask-app:latest
docker push username/custom-flask-app:latest
```

### Kubernetes Operations

#### Basic kubectl commands
```bash
# View cluster information
kubectl cluster-info

# Get all resources in the cluster
kubectl get all --all-namespaces

# List nodes in the cluster
kubectl get nodes

# Describe a specific node
kubectl describe node node-name

# View all namespaces
kubectl get namespaces

# Create a new namespace
kubectl create namespace app-namespace

# Set the default namespace for kubectl
kubectl config set-context --current --namespace=app-namespace

# List all pods
kubectl get pods

# Get detailed information about a pod
kubectl describe pod pod-name

# View pod logs
kubectl logs pod-name

# View pod logs for a specific container in a pod
kubectl logs pod-name -c container-name

# Follow pod logs
kubectl logs -f pod-name

# Execute a command in a running pod
kubectl exec -it pod-name -- /bin/bash

# Get all deployments
kubectl get deployments

# Scale a deployment
kubectl scale deployment deployment-name --replicas=3

# Edit a deployment
kubectl edit deployment deployment-name

# View all services
kubectl get services

# Create a basic deployment from an image
kubectl create deployment nginx-deployment --image=nginx

# Expose a deployment as a service
kubectl expose deployment nginx-deployment --port=80 --type=LoadBalancer

# Apply a configuration from a file
kubectl apply -f deployment.yaml

# Delete a resource
kubectl delete deployment nginx-deployment
kubectl delete service nginx-deployment
```

#### Create Kubernetes resources
```bash
# Create a simple deployment YAML
cat > deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: web-app
        image: nginx:latest
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
---
apiVersion: v1
kind: Service
metadata:
  name: web-app
spec:
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
  type: LoadBalancer
EOF

# Apply the deployment
kubectl apply -f deployment.yaml

# Check the deployment
kubectl get deployments
kubectl get pods
kubectl get services

# Create a ConfigMap
cat > configmap.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_url: "mysql://db:3306/myapp"
  cache_host: "redis:6379"
  log_level: "info"
EOF

kubectl apply -f configmap.yaml

# Create a Secret
cat > secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: app-secrets
type: Opaque
data:
  db_user: $(echo -n "admin" | base64)
  db_password: $(echo -n "p@ssw0rd" | base64)
  api_key: $(echo -n "a1b2c3d4e5f6g7h8i9j0" | base64)
EOF

kubectl apply -f secret.yaml

# Use ConfigMap and Secret in a Deployment
cat > app-deployment.yaml << EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:latest
        ports:
        - containerPort: 8080
        env:
        - name: DATABASE_URL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: database_url
        - name: LOG_LEVEL
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: log_level
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_user
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: app-secrets
              key: db_password
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
      volumes:
      - name: config-volume
        configMap:
          name: app-config
EOF

kubectl apply -f app-deployment.yaml
```

## Cloud Management

### AWS CLI Operations

#### Basic AWS CLI commands
```bash
# Configure AWS CLI
aws configure

# List all S3 buckets
aws s3 ls

# Create an S3 bucket
aws s3 mb s3://my-bucket-name

# Copy a local file to S3
aws s3 cp file.txt s3://my-bucket-name/

# Copy a file from S3 to local
aws s3 cp s3://my-bucket-name/file.txt ./

# Sync a local directory with an S3 bucket
aws s3 sync ./local-dir s3://my-bucket-name/remote-dir

# List all EC2 instances
aws ec2 describe-instances

# Start an EC2 instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Stop an EC2 instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Describe security groups
aws ec2 describe-security-groups

# Create a new security group
aws ec2 create-security-group --group-name my-sg --description "My security group" --vpc-id vpc-1a2b3c4d

# Add a rule to a security group
aws ec2 authorize-security-group-ingress --group-id sg-123abc --protocol tcp --port 22 --cidr 203.0.113.0/24

# List all RDS instances
aws rds describe-db-instances

# Create an EBS snapshot
aws ec2 create-snapshot --volume-id vol-1234567890abcdef0 --description "My snapshot"

# List available EBS snapshots
aws ec2 describe-snapshots --owner-ids self
```

#### AWS S3 backup script
```bash
cat > aws-s3-backup.sh << 'EOF'
#!/bin/bash
#
# AWS S3 Backup Script
#

# Configuration
BACKUP_DIR="/tmp/backup"
S3_BUCKET="my-backup-bucket"
S3_PREFIX="backups/$(hostname)/$(date +%Y-%m-%d)"
LOG_FILE="/var/log/s3-backup.log"

# Ensure backup directory exists
mkdir -p "$BACKUP_DIR"

# Log start
echo "==== S3 Backup Started: $(date) ====" >> "$LOG_FILE"

# Backup important directories
echo "Creating archive of /etc..." >> "$LOG_FILE"
tar -czf "$BACKUP_DIR/etc.tar.gz" /etc 2>/dev/null

echo "Creating archive of /home..." >> "$LOG_FILE"
tar -czf "$BACKUP_DIR/home.tar.gz" /home 2>/dev/null

echo "Creating archive of /var/www..." >> "$LOG_FILE"
tar -czf "$BACKUP_DIR/www.tar.gz" /var/www 2>/dev/null

# MySQL backup
if command -v mysqldump &> /dev/null; then
  echo "Backing up MySQL databases..." >> "$LOG_FILE"
  mkdir -p "$BACKUP_DIR/mysql"
  
  # MySQL databases backup
  mysql -u root -p"$MYSQL_PASSWORD" -e "SHOW DATABASES;" | grep -v "Database\|information_schema\|performance_schema" | while read db; do
    echo "Backing up MySQL database: $db" >> "$LOG_FILE"
    mysqldump -u root -p"$MYSQL_PASSWORD" "$db" | gzip > "$BACKUP_DIR/mysql/$db.sql.gz"
  done
fi

# Upload to S3
echo "Uploading to S3..." >> "$LOG_FILE"
aws s3 sync "$BACKUP_DIR" "s3://$S3_BUCKET/$S3_PREFIX" --delete >> "$LOG_FILE" 2>&1

# Clean up local files
echo "Cleaning up local files..." >> "$LOG_FILE"
rm -rf "$BACKUP_DIR"

# Delete old backups from S3 (keep last 30 days)
echo "Removing old backups..." >> "$LOG_FILE"
LAST_MONTH=$(date -d "30 days ago" +%Y-%m-%d)
for prefix in $(aws s3 ls "s3://$S3_BUCKET/backups/$(hostname)/" | awk '{print $2}' | grep -E '[0-9]{4}-[0-9]{2}-[0-9]{2}' | sed 's/\///'); do
  if [[ "$prefix" < "$LAST_MONTH" ]]; then
    echo "Removing old backup: $prefix" >> "$LOG_FILE"
    aws s3 rm "s3://$S3_BUCKET/backups/$(hostname)/$prefix" --recursive >> "$LOG_FILE" 2>&1
  fi
done

# Log completion
echo "==== S3 Backup Completed: $(date) ====" >> "$LOG_FILE"
EOF

chmod +x aws-s3-backup.sh
```

## Advanced Scheduling and Automation

### Advanced Cron Jobs

#### Cron job with email notification
```bash
# Create a script with notification
cat > /usr/local/bin/backup-notify.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/var/backups/data"
LOG_FILE="/var/log/backup-$(date +%Y%m%d).log"
EMAIL="admin@example.com"

# Start logging
echo "Backup started at $(date)" > "$LOG_FILE"

# Perform backup
mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/data-$(date +%Y%m%d).tar.gz" /path/to/data >> "$LOG_FILE" 2>&1
RESULT=$?

# Check status
if [ $RESULT -eq 0 ]; then
  STATUS="SUCCESS"
  SIZE=$(du -h "$BACKUP_DIR/data-$(date +%Y%m%d).tar.gz" | cut -f1)
  MESSAGE="Backup completed successfully. Size: $SIZE"
else
  STATUS="FAILURE"
  MESSAGE="Backup failed with error code $RESULT"
fi

# Log completion
echo "Backup finished at $(date) - $STATUS" >> "$LOG_FILE"
echo "$MESSAGE" >> "$LOG_FILE"

# Send email notification
echo -e "Subject: Backup $STATUS - $(hostname) - $(date +%Y-%m-%d)\n\n$MESSAGE\n\nSee attached log for details." | mutt -a "$LOG_FILE" -s "Backup $STATUS - $(hostname)" -- "$EMAIL"

# Only keep last 30 days of backups
find "$BACKUP_DIR" -name "*.tar.gz" -type f -mtime +30 -delete
EOF

chmod +x /usr/local/bin/backup-notify.sh

# Add to crontab with specific time
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/backup-notify.sh") | crontab -
```

#### Cron job with timing options
```bash
# Schedule at specific time
(crontab -l 2>/dev/null; echo "30 4 * * * /path/to/script.sh") | crontab -  # 4:30 AM daily

# Schedule at specific time on weekdays
(crontab -l 2>/dev/null; echo "0 9 * * 1-5 /path/to/script.sh") | crontab -  # 9 AM Mon-Fri

# Schedule first day of month
(crontab -l 2>/dev/null; echo "0 0 1 * * /path/to/script.sh") | crontab -  # Midnight on first day of month

# Schedule every 15 minutes
(crontab -l 2>/dev/null; echo "*/15 * * * * /path/to/script.sh") | crontab -  # Every 15 minutes

# Schedule every hour during business hours
(crontab -l 2>/dev/null; echo "0 9-17 * * 1-5 /path/to/script.sh") | crontab -  # 9 AM to 5 PM, Mon-Fri

# Schedule twice per hour during specific hours
(crontab -l 2>/dev/null; echo "0,30 8-18 * * * /path/to/script.sh") | crontab -  # Every 30 min from 8 AM to 6 PM

# Schedule on specific months
(crontab -l 2>/dev/null; echo "0 12 1 1,4,7,10 * /path/to/script.sh") | crontab -  # Noon on 1st day of Jan/Apr/Jul/Oct

# Schedule using "at" for one-time execution
echo "/path/to/script.sh" | at 10:00 PM
```

### Systemd Timers

#### Create a systemd timer instead of cron
```bash
# Create a service unit file
cat > /etc/systemd/system/backup.service << EOF
[Unit]
Description=Daily Backup Service
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
User=root
Group=root

[Install]
WantedBy=multi-user.target
EOF

# Create a timer unit file
cat > /etc/systemd/system/backup.timer << EOF
[Unit]
Description=Run backup daily at 3 AM
Requires=backup.service

[Timer]
OnCalendar=*-*-* 03:00:00
RandomizedDelaySec=900
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Enable and start the timer
systemctl daemon-reload
systemctl enable backup.timer
systemctl start backup.timer

# Check timer status
systemctl list-timers
```

#### Create a systemd timer with more complex scheduling
```bash
# Create service unit file for database backup
cat > /etc/systemd/system/db-backup.service << EOF
[Unit]
Description=Database Backup Service
After=mysql.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/db-backup.sh
User=root
Group=root

[Install]
WantedBy=multi-user.target
EOF

# Create more complex timer
cat > /etc/systemd/system/db-backup.timer << EOF
[Unit]
Description=Run database backups on a schedule
Requires=db-backup.service

[Timer]
# Run hourly during business hours (8 AM to 6 PM) on weekdays
OnCalendar=Mon..Fri *-*-* 08..18:00:00
# Also run once at midnight
OnCalendar=*-*-* 00:00:00
# Add some randomization to not overload the system
RandomizedDelaySec=300
Persistent=true

[Install]
WantedBy=timers.target
EOF

# Enable and start the timer
systemctl daemon-reload
systemctl enable db-backup.timer
systemctl start db-backup.timer
```

## Advanced File System Operations

### LVM Management

#### Create and extend an LVM setup
```bash
# Create physical volumes from disks
pvcreate /dev/sdb /dev/sdc

# Check physical volumes
pvs
pvdisplay

# Create a volume group
vgcreate data_vg /dev/sdb /dev/sdc

# Check volume group
vgs
vgdisplay data_vg

# Create logical volumes
# Use 10GB for mysql data
lvcreate -L 10G -n mysql_lv data_vg
# Use 5GB for web data
lvcreate -L 5G -n web_lv data_vg
# Use remaining space for backups
lvcreate -l 100%FREE -n backup_lv data_vg

# Check logical volumes
lvs
lvdisplay data_vg/mysql_lv

# Create filesystems
mkfs.ext4 /dev/data_vg/mysql_lv
mkfs.ext4 /dev/data_vg/web_lv
mkfs.ext4 /dev/data_vg/backup_lv

# Create mount points
mkdir -p /var/lib/mysql /var/www /var/backups

# Add entries to /etc/fstab
echo "/dev/data_vg/mysql_lv  /var/lib/mysql  ext4  defaults  0 2" >> /etc/fstab
echo "/dev/data_vg/web_lv    /var/www        ext4  defaults  0 2" >> /etc/fstab
echo "/dev/data_vg/backup_lv /var/backups    ext4  defaults  0 2" >> /etc/fstab

# Mount all
mount -a

# Extend a logical volume
# Add a new disk
pvcreate /dev/sdd
vgextend data_vg /dev/sdd

# Extend a logical volume by 20GB
lvextend -L +20G /dev/data_vg/backup_lv
# Resize the filesystem to use the new space
resize2fs /dev/data_vg/backup_lv

# Extend a logical volume to use all remaining space
lvextend -l +100%FREE /dev/data_vg/backup_lv
resize2fs /dev/data_vg/backup_lv
```

#### Create a snapshot and restore from it
```bash
# Create a snapshot of a logical volume (1GB size for snapshot)
lvcreate -L 1G -s -n mysql_snap /dev/data_vg/mysql_lv

# Mount the snapshot read-only
mkdir -p /mnt/snapshot
mount -o ro /dev/data_vg/mysql_snap /mnt/snapshot

# Restore files from snapshot
cp -a /mnt/snapshot/important_file /var/lib/mysql/

# Unmount snapshot
umount /mnt/snapshot

# Remove the snapshot when done
lvremove /dev/data_vg/mysql_snap

# Restore entire volume from snapshot
# (Unmount original volume first)
umount /var/lib/mysql
lvconvert --merge /dev/data_vg/mysql_snap
# After merge, snapshot is gone and original LV is restored
mount /var/lib/mysql
```

### RAID Management

#### Create and manage software RAID
```bash
# Install mdadm if not already installed
apt-get install mdadm

# Create a RAID 1 (mirror) with two disks
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Check RAID status
cat /proc/mdstat
mdadm --detail /dev/md0

# Create a filesystem on the RAID
mkfs.ext4 /dev/md0

# Mount the RAID
mkdir -p /mnt/raid
mount /dev/md0 /mnt/raid

# Add entry to /etc/fstab
echo "/dev/md0  /mnt/raid  ext4  defaults  0 2" >> /etc/fstab

# Save RAID configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u

# Add a spare disk to the RAID
mdadm --add /dev/md0 /dev/sdd

# Replace a failed disk
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb
# Install new disk and add it
mdadm /dev/md0 --add /dev/sde
# RAID will rebuild automatically

# Check rebuild progress
cat /proc/mdstat

# Convert RAID 1 to RAID 5 (need to add more disks)
mdadm --grow /dev/md0 --level=5 --raid-devices=3 --add /dev/sdf

# Expand a RAID array with a new disk
mdadm --grow /dev/md0 --raid-devices=4 --add /dev/sdg
```

## Custom Command Reference Card

Here's a quick reference card of useful custom one-liners:

#### Quick system assessment
```bash
# 1. Check system load, memory, disk, and processes in one command
echo "System: $(hostname)" && uptime && free -h && df -h | grep -vE '^tmpfs|^udev|^Filesystem' && ps aux --sort=-%cpu | head -6

# 2. Check for failed services
systemctl list-units --state=failed

# 3. Check recent security events
grep -i "authentication failure\|failed password\|session opened\|session closed" /var/log/auth.log | tail -20

# 4. Quick security scan
lsof -i -P -n | grep LISTEN && cat /etc/passwd | grep "bash\|sh" | grep -v nologin && lastlog | grep -v "Never logged in"

# 5. Track who is using the most disk space
du -h --max-depth=2 /home | sort -hr | head -10
```

#### Single-command network troubleshooting
```bash
# 1. Check network interfaces, routes, and connections
ip -br addr && echo -e "\nRoutes:" && ip -br route && echo -e "\nListening sockets:" && ss -tuln

# 2. Quick network trace to Google DNS
mtr -r -c 5 8.8.8.8

# 3. Analyze HTTP response time
curl -w "\nDNS lookup: %{time_namelookup}s\nConnect: %{time_connect}s\nTLS: %{time_appconnect}s\nTotal: %{time_total}s\n" -o /dev/null -s https://example.com

# 4. Testing all ports on a remote server
for port in 22 80 443 3306 5432; do nc -zvw1 example.com $port 2>&1; done

# 5. Live monitor of all HTTP traffic
tcpdump -i any -nn -s0 port 80 or port 443
```

#### Database maintenance shortcuts
```bash
# 1. MySQL: Clean and optimize all tables
mysql -e "SELECT CONCAT('OPTIMIZE TABLE ', table_schema, '.', table_name, ';') FROM information_schema.tables WHERE table_schema NOT IN ('performance_schema', 'information_schema');" | mysql

# 2. PostgreSQL: Vacuum analyze all databases
psql -c "SELECT 'VACUUM ANALYZE ' || datname || ';' FROM pg_database WHERE datname NOT IN ('template0');" | psql

# 3. Clean MySQL binary logs
mysql -e "PURGE BINARY LOGS BEFORE DATE(NOW() - INTERVAL 7 DAY);"

# 4. PostgreSQL: List largest tables
psql -c "SELECT table_schema, table_name, pg_size_pretty(pg_total_relation_size(table_schema||'.'||table_name)) as size FROM information_schema.tables WHERE table_schema NOT IN ('pg_catalog', 'information_schema') ORDER BY pg_total_relation_size(table_schema||'.'||table_name) DESC LIMIT 10;"
```

#### Web server administration shortcuts
```bash
# 1. Nginx: Find top client IPs
awk '{print $1}' /var/log/nginx/access.log | sort | uniq -c | sort -nr | head -20

# 2. Apache: Analyze top URLs
grep -v "favicon.ico\|robots.txt" /var/log/apache2/access.log | awk '{print $7}' | sort | uniq -c | sort -nr | head -20

# 3. Apache: Find all 5xx errors
grep "HTTP/1.1\" 5" /var/log/apache2/access.log | cut -d '"' -f2,3 | sort | uniq -c | sort -rn

# 4. Generate a self-signed SSL certificate in one line
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout server.key -out server.crt -subj "/C=US/ST=State/L=City/O=Organization/CN=example.com"

# 5. Test HTTPS configuration quality
curl -sSL https://www.ssllabs.com/ssltest/analyze.html?d=example.com
```

#### Backup and recovery operations
```bash
# 1. Quick encrypted backup
tar -czf - /important/data | openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -out backup.tar.gz.enc

# 2. Decrypt backup
openssl enc -d -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -in backup.tar.gz.enc | tar -xzf -

# 3. MySQL database backup and compress
mysqldump --single-transaction --quick --lock-tables=false --all-databases | gzip > all_databases_$(date +%Y%m%d).sql.gz

# 4. Recursive rsync with resume capability
rsync -avz --partial --progress --bwlimit=2000 /source/ user@remote:/destination/
```
