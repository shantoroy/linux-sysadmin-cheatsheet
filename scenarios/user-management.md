# User Management Scenarios

## User Account Management

### Create a batch of users from a file
```bash
# Create users listed in a file (one per line) with default settings
while read username; do useradd -m "$username"; done < userlist.txt

# Create users with specific groups, shells, and home directories
while read username; do useradd -m -g staff -G developers -s /bin/bash -d /home/dev/"$username" "$username"; done < userlist.txt
```

### Set password expiry for multiple users
```bash
# Set 90-day password expiration for all users in a group
getent group developers | cut -d: -f4 | tr ',' '\n' | xargs -I{} chage -M 90 {}

# Force password change at next login for inactive users
lastlog | grep "**Never logged in**" | awk '{print $1}' | xargs -I{} chage -d 0 {}
```

### Bulk modify user attributes
```bash
# Add all users to a new supplementary group
getent passwd | awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' | xargs -I{} usermod -aG newgroup {}

# Change shell for all users in a group
getent group developers | cut -d: -f4 | tr ',' '\n' | xargs -I{} usermod -s /bin/zsh {}
```

### Find and disable inactive user accounts
```bash
# Find users who haven't logged in for over 90 days
lastlog | awk -F' +' 'NR>1 && $5!="**Never" && systime()-mktime($6" "$5" "$9" "$7" "$8)>90*24*60*60 {print $1}' | xargs -I{} usermod -L {}

# Alternative using lastlog for inactive users
for user in $(awk -F: '$3 >= 1000 && $3 < 65534 {print $1}' /etc/passwd); do
  if [[ $(date -d "$(chage -l $user | grep 'Last password change' | cut -d: -f2)" +%s) -lt $(date -d "-90 days" +%s) ]]; then
    usermod -L $user
    echo "Disabled account: $user"
  fi
done
```

### Find users with sudo privileges
```bash
# List all users with sudo privileges
grep -Po '^[^#]+\s+ALL=(?:\([^)]+\))?\s+ALL' /etc/sudoers | cut -d ' ' -f1

# Alternative, including group membership checks
for user in $(getent passwd | awk -F: '{print $1}'); do
  groups $user | grep -q '\bsudo\b' && echo "$user has sudo via sudo group"
  groups $user | grep -q '\bwheel\b' && echo "$user has sudo via wheel group"
done
grep -v '^#' /etc/sudoers | grep ALL | grep -v '%' | awk '{print $1" has sudo via sudoers file"}'
```

### Reset a forgotten user password using root
```bash
# Reset password for user
passwd username

# Force user to change password at next login
passwd -e username
```

### Create a system account
```bash
# Create a system account for a service
useradd -r -s /sbin/nologin myservice

# Create a system account with custom home directory
useradd -r -s /sbin/nologin -d /opt/myapp myapp
```