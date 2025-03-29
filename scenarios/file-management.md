# File Management Scenarios

## Finding and Deleting Files

### Find and delete files older than X days
```bash
# Find and delete files older than 15 days in /var/log
find /var/log -type f -mtime +15 -exec rm {} \;

# Safer version with confirmation
find /var/log -type f -mtime +15 -exec rm -i {} \;

# List files that would be deleted
find /var/log -type f -mtime +15 -ls
```

### Find and delete empty files
```bash
# Find and delete empty files
find /path/to/dir -type f -empty -delete
```

### Find and delete empty directories
```bash
# Find and delete empty directories
find /path/to/dir -type d -empty -delete
```

### Find and delete files by size
```bash
# Find and delete files larger than 100MB
find /path/to/dir -type f -size +100M -exec rm {} \;

# Find and delete files smaller than 10KB
find /path/to/dir -type f -size -10k -exec rm {} \;
```

### Find and delete files with specific extensions
```bash
# Find and delete .tmp and .bak files
find /path/to/dir -type f \( -name "*.tmp" -o -name "*.bak" \) -delete
```

### Clean up specific log files over X size
```bash
# Find log files over 100MB and truncate them
find /var/log -name "*.log" -size +100M -exec sh -c 'echo "" > "{}"' \;

# Alternative using truncate
find /var/log -name "*.log" -size +100M -exec truncate -s 0 {} \;
```

### Remove duplicate files
```bash
# Find and list duplicate files based on content (requires fdupes)
fdupes -r /path/to/dir

# Find and delete duplicate files (keeping one copy)
fdupes -r -d -N /path/to/dir
```

## Advanced File Operations

### Find and compress old log files
```bash
# Find log files older than 30 days and compress them
find /var/log -name "*.log" -type f -mtime +30 -exec gzip {} \;
```

### Batch rename files
```bash
# Rename all .txt files to .bak
find /path/to/dir -name "*.txt" -exec sh -c 'mv "$1" "${1%.txt}.bak"' sh {} \;

# Using rename (perl version) to replace spaces with underscores
rename 's/ /_/g' *.txt

# Rename files with sequential numbers
i=1; for f in *.jpg; do mv "$f" "image_$(printf '%03d' $i).jpg"; ((i++)); done
```

### Recursively change file permissions
```bash
# Change all directories to 755 and files to 644
find /path/to/dir -type d -exec chmod 755 {} \; 
find /path/to/dir -type f -exec chmod 644 {} \;

# Change file ownership recursively
find /path/to/dir -exec chown user:group {} \;
```

### Find files modified in the last X minutes
```bash
# Find files modified in the last 60 minutes
find /path/to/dir -type f -mmin -60
```

### Synchronize two directories
```bash
# Mirror source directory to destination
rsync -avz --delete /source/dir/ /destination/dir/

# Backup with exclusions
rsync -avz --exclude='*.tmp' --exclude='cache/' /source/dir/ /destination/dir/
```

### Find and archive old files
```bash
# Archive files older than 90 days, then remove them
find /path/to/dir -type f -mtime +90 -exec tar -rvf old_files.tar {} \; -exec rm {} \;
```

### Handle files with unusual names (spaces, special chars)
```bash
# Find and fix files with problematic names
find /path/to/dir -name "* *" -exec rename 's/ /_/g' {} \;

# Handle files with newlines or special characters
find /path/to/dir -type f -print0 | xargs -0 some_command
```

### Set creation time to match modification time
```bash
# Make access/creation time match modification time
find /path/to/dir -type f -exec touch -am {} \;
```