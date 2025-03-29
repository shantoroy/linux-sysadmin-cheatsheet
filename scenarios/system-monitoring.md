# System Monitoring Scenarios

## Performance Monitoring

### Monitor system load and resources
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

### Create a system resource monitoring script
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

### Setup system load alerts
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

## System Maintenance 

### Automate system updates
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

### Schedule system maintenance tasks
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