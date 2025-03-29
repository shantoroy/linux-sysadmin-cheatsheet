# Text Manipulation Scenarios

## Find and Replace Text in Files

### Replace a string in a single file
```bash
# Replace all occurrences of "old_string" with "new_string" in file.txt
sed -i 's/old_string/new_string/g' file.txt
```

### Replace a string in multiple files
```bash
# Using sed with find to replace in all .conf files
find /path/to/dir -type f -name "*.conf" -exec sed -i 's/old_string/new_string/g' {} \;
```

### Replace a string only on line 10
```bash
# Replace "old_string" with "new_string" only on line 10
sed -i '10s/old_string/new_string/g' file.txt
```

### Replace a string with special characters
```bash
# Replace a path like /var/log with /opt/log
sed -i 's/\/var\/log/\/opt\/log/g' file.txt

# Alternative with different delimiter
sed -i 's|/var/log|/opt/log|g' file.txt
```

### Replace only the first occurrence on each line
```bash
# Replace only the first occurrence of "old_string" per line
sed -i 's/old_string/new_string/' file.txt
```

### Replace with case insensitive matching
```bash
# Replace with case insensitive matching
sed -i 's/old_string/new_string/gi' file.txt
```

### Replace whole word only
```bash
# Replace only whole words matching "old"
sed -i 's/\bold\b/new/g' file.txt
```

### Replace a string across multiple lines
```bash
# Replace string across multiple lines (GNU sed)
sed -i ':a;N;$!ba;s/old_string\n/new_string\n/g' file.txt
```

### Replace IP address format
```bash
# Replace IP address 192.168.1.100 with 10.0.0.50
sed -i 's/192\.168\.1\.100/10\.0\.0\.50/g' file.txt
```

### Add a line after a specific pattern
```bash
# Add a new line after lines containing "pattern"
sed -i '/pattern/a\New line text goes here' file.txt
```

### Add a line before a specific pattern
```bash
# Add a new line before lines containing "pattern"
sed -i '/pattern/i\New line text goes here' file.txt
```

### Delete lines matching a pattern
```bash
# Delete all lines containing "pattern"
sed -i '/pattern/d' file.txt
```

### Delete empty lines
```bash
# Delete all empty lines
sed -i '/^$/d' file.txt
```

### Delete lines with only whitespace
```bash
# Delete lines with only whitespace
sed -i '/^\s*$/d' file.txt
```

### Comment out lines containing a pattern
```bash
# Comment out lines containing "pattern"
sed -i '/pattern/s/^/#/' file.txt
```

### Uncomment lines containing a pattern
```bash
# Uncomment lines (remove leading # from lines containing "pattern")
sed -i '/pattern/s/^#//' file.txt
```

## Extract Information from Files

### Extract lines between two patterns
```bash
# Extract lines between START and END patterns (inclusive)
sed -n '/START/,/END/p' file.txt
```

### Extract specific columns from CSV files
```bash
# Extract 1st and 3rd columns from CSV file
awk -F, '{print $1,$3}' file.csv

# Extract and reformat with a different delimiter
awk -F, '{print $1"|"$3}' file.csv
```

### Extract the Nth word from each line
```bash
# Extract the 3rd word from each line
awk '{print $3}' file.txt
```

### Extract lines matching multiple patterns
```bash
# Extract lines containing either "error" or "warning"
grep -E 'error|warning' file.log
```

### Extract blocks of text between empty lines
```bash
# Print paragraphs (text blocks separated by empty lines) containing "pattern"
awk -v RS="" '/pattern/' file.txt
```

### Extract text between XML tags
```bash
# Extract content between <tag> and </tag>
grep -oP '(?<=<tag>).*(?=</tag>)' file.xml
```

### Extract all IP addresses from a file
```bash
# Extract IPv4 addresses
grep -oE '[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}' file.txt

# More precise IPv4 extraction (better but more complex)
grep -oE '\b((25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\b' file.txt
```

### Extract all email addresses from a file
```bash
# Extract email addresses
grep -oE '[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}' file.txt
```

### Extract all URLs from a file
```bash
# Extract URLs
grep -oE 'https?://[^ ]+' file.txt
```

### Extract specific lines by line number
```bash
# Extract lines 10-20
sed -n '10,20p' file.txt
```
