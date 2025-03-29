# Custom Command Reference Card

Here's a quick reference card of useful custom one-liners:

### Quick system assessment
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

### Single-command network troubleshooting
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

### Database maintenance shortcuts
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

### Web server administration shortcuts
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

### Backup and recovery operations
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
