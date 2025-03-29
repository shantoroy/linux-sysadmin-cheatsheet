# Advanced Scheduling and Automation

## Advanced Cron Jobs

### Cron job with email notification
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

### Cron job with timing options
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

## Systemd Timers

### Create a systemd timer instead of cron
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

### Create a systemd timer with more complex scheduling
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

