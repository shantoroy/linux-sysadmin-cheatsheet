# Automation and Scripting Scenarios

## Backup Automation

### Create a full backup script
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

## System Initialization and Bootstrap

### Create a new system initialization script
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

