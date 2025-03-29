# Security Scenarios

## Firewall Configuration

### Configure firewalld (RHEL/CentOS/Fedora)
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

### Configure UFW (Ubuntu/Debian)
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

### Configure iptables directly
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

## Securing SSH

### Secure SSH configuration
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

### Set up SSH key authentication
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

### Set up SSH with 2FA
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

## Intrusion Detection and Prevention

### Configure Fail2Ban
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

### Set up rootkit scanning with rkhunter
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

### Audit system with Lynis
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