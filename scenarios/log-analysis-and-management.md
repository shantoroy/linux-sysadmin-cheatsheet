# Log Analysis and Management

## Analyzing Log Files

### Find errors in log files
```bash
# Search for ERROR, error, or Error in log files
grep -i "error" /var/log/*.log

# Show 2 lines before and after each error
grep -i -B2 -A2 "error" /var/log/*.log

# Count errors by type
grep -i "error" /var/log/*.log | awk '{print $NF}' | sort | uniq -c | sort -nr
```

### Find failed login attempts
```bash
# Check for failed logins in auth log
grep "Failed password" /var/log/auth.log

# Count failed logins by username
grep "Failed password" /var/log/auth.log | awk '{print $(NF-5)}' | sort | uniq -c | sort -nr

# Count failed logins by IP address
grep "Failed password" /var/log/auth.log | grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" | sort | uniq -c | sort -nr
```

### Extract and summarize web server access logs
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

### Extract time-specific log entries
```bash
# Get log entries between specific times
awk '$4 >= "[01/Mar/2023:10:00:00" && $4 <= "[01/Mar/2023:11:00:00"' /var/log/apache2/access.log

# Get today's log entries
grep "`date +"%b %d"`" /var/log/syslog
```

### Calculate average response time from logs
```bash
# For logs with response time in the last field
awk '{sum+=$NF; count++} END {print "Average:",sum/count,"ms"}' /var/log/response_times.log
```

### Analyze sudo usage
```bash
# List all sudo commands executed
grep "sudo:" /var/log/auth.log | grep "COMMAND"

# List sudo commands by user
grep "sudo:" /var/log/auth.log | grep "COMMAND" | awk '{print $6,$NF}' | sort | uniq -c
```

## Log Rotation and Maintenance

### Manually rotate a log file
```bash
# Create a timestamped backup and reset the log file
cp access.log "access.log.$(date +%Y%m%d)"
cat /dev/null > access.log
```

### Create a custom logrotate configuration
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

### Force log rotation
```bash
# Force immediate rotation for all configurations
logrotate -f /etc/logrotate.conf

# Force rotation for a specific configuration
logrotate -f /etc/logrotate.d/myapp
```

### Set up log file size limits
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