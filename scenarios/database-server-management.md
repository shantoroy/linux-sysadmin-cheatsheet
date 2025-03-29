# Database Server Management 

## MySQL/MariaDB Administration 

### MySQL performance tuning
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

### MySQL security hardening
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

## PostgreSQL Administration

### Backup and restore PostgreSQL databases
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

### PostgreSQL performance tuning
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