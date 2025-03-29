# Cloud Management

## AWS CLI Operations

### Basic AWS CLI commands
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

### AWS S3 backup script
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

