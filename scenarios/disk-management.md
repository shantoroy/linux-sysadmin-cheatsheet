# Disk Management Scenarios

## Space Management and Analysis

### Find directories consuming the most space
```bash
# Show top 10 directories by size
du -hx --max-depth=1 /path | sort -hr | head -10

# Alternative using ncdu for interactive exploration
ncdu /path
```

### Clean up disk space
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

### Find largest files
```bash
# Find top 20 largest files in a path
find /path -type f -exec du -h {} \; | sort -hr | head -20

# Find large files (>100MB) only
find /path -type f -size +100M -exec ls -lh {} \; | sort -k5,5hr
```

### Clean temporary files
```bash
# Clean /tmp files older than 10 days
find /tmp -type f -atime +10 -delete

# Clean specific temporary file patterns
find /tmp -name "*.tmp" -o -name "*.temp" -o -name "*.bak" -delete
```

### Check disk usage by file types
```bash
# Check disk usage by file extension in current directory
find . -type f | grep -v "^\./\." | sed 's/.*\.//' | sort | uniq -c | sort -nr
```

## Filesystem Management

### Create a new filesystem
```bash
# Create and mount an ext4 filesystem
mkfs.ext4 /dev/sdb1
mkdir -p /mnt/newdisk
mount /dev/sdb1 /mnt/newdisk

# Add to fstab for persistent mounting
echo "/dev/sdb1 /mnt/newdisk ext4 defaults 0 2" >> /etc/fstab
```

### Extend a logical volume and filesystem
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

### Set up disk quotas
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

### Set up disk monitoring alerts
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
