# YUM Package Management

## Repository Management

### List All Configured Repositories
```bash
# List all enabled and disabled repositories
yum repolist all

# List only enabled repositories
yum repolist enabled

# List only disabled repositories
yum repolist disabled
```

### Get Repository Information
```bash
# Get detailed information about a specific repository
yum repoinfo base

# Get verbose information about all repositories
yum repoinfo -v
```

### Add a New Repository
```bash
# Add a repository manually by creating a .repo file
cat > /etc/yum.repos.d/example.repo << EOF
[example]
name=Example Repository
baseurl=http://example.com/repo
enabled=1
gpgcheck=1
gpgkey=http://example.com/key.asc
EOF

# Add a repository using rpm
rpm -Uvh https://example.com/example-repo.rpm
```

### Enable/Disable Repositories
```bash
# Disable a repository
yum-config-manager --disable example

# Enable a repository
yum-config-manager --enable example

# Install yum-utils if yum-config-manager is not available
yum install yum-utils
```

### Clean Repository Cache
```bash
# Clean all cached packages and metadata
yum clean all

# Clean just the cached packages
yum clean packages

# Clean just the metadata
yum clean metadata

# Rebuild the yum cache
yum makecache
```

## Package Queries and Information

### Search for Packages
```bash
# Search for packages by name
yum search nginx

# Search package names and summaries for a keyword
yum search all "web server"

# List packages matching a specific name pattern
yum list "php*"

# List available packages that can be installed
yum list available

# List installed packages
yum list installed

# List packages that have updates available
yum list updates

# Check if a specific package is installed
yum list installed httpd
```

### Get Package Information
```bash
# Get detailed information about a package
yum info nginx

# Get information about a specific installed package
yum info installed httpd

# List all versions of a package available
yum --showduplicates list httpd
```

### Find Which Package Provides a File
```bash
# Find which package provides a specific file
yum provides /usr/bin/htpasswd

# Alternative syntax
yum whatprovides */htpasswd
```

### List Package Dependencies
```bash
# Show dependencies for a package
yum deplist httpd

# Show packages that require a specific package
yum repoquery --requires httpd
```

### List Files in a Package
```bash
# List all files included in an installed package
rpm -ql httpd

# List all files in a package (not installed)
repoquery -l httpd
```

### Show Package Change History
```bash
# Show transaction history
yum history

# Get details about a specific transaction
yum history info 25

# Show history for a specific package
yum history list httpd
```

## Package Installation and Management

### Install Packages
```bash
# Install a single package
yum install httpd

# Install multiple packages
yum install httpd php php-mysql

# Install a specific version of a package
yum install httpd-2.4.6-93.el7

# Install a local RPM package with dependency resolution
yum localinstall ./package.rpm
```

### Update Packages
```bash
# Check for updates
yum check-update

# Update a specific package
yum update httpd

# Update all packages
yum update

# Apply only security updates
yum update --security
```

### Remove Packages
```bash
# Remove a package
yum remove httpd

# Remove a package without removing dependencies
yum remove --noautoremove httpd
```

### Install Package Groups
```bash
# List available package groups
yum group list

# Get information about a package group
yum group info "Development Tools"

# Install a package group
yum group install "Development Tools"

# Update a package group
yum group update "Development Tools"

# Remove a package group
yum group remove "Development Tools"
```

### Downgrade Packages
```bash
# Downgrade a package to a previous version
yum downgrade httpd
```

### Reinstall Packages
```bash
# Reinstall a package
yum reinstall httpd
```

## Transaction Management

### Perform Dry Run
```bash
# See what would happen for any yum command without actually doing it
yum install nginx --assumeno
```

### Download Only
```bash
# Download a package without installing it
yum install --downloadonly --downloaddir=/tmp/packages httpd
```

### Rollback Transactions
```bash
# Undo a transaction by ID
yum history undo 42

# Rollback to a specific transaction ID
yum history rollback 36
```

### Work with Multiple Packages
```bash
# Install all packages needed for a specific task
yum install @lamp-server

# See what package group provides a functionality
yum group list | grep -i database
```

## Advanced YUM Operations

### Configure YUM Settings
```bash
# List all YUM configuration settings
yum-config-manager --dump

# Set a specific configuration option
yum-config-manager --save --setopt=timeout=30
```

### Work with Yum Plugins
```bash
# List installed YUM plugins
yum list installed "yum-plugin-*"

# Install a YUM plugin
yum install yum-plugin-fastestmirror
```

### Lock Package Versions
```bash
# Prevent a package from being updated (requires versionlock plugin)
yum install yum-plugin-versionlock
yum versionlock add httpd

# Display version-locked packages
yum versionlock list

# Remove a version lock
yum versionlock delete httpd
```

### Work with Security Updates
```bash
# List available security updates
yum updateinfo list security

# Apply security updates rated as critical
yum update --security --sec-severity=Critical
```

### Package Verification
```bash
# Verify integrity of an installed package
rpm -V httpd

# Verify all installed packages
rpm -Va
```

### Troubleshooting Package Issues
```bash
# Fix package dependencies issues
yum clean all
yum distro-sync

# Check for conflicting files
rpm -Va | grep -i missing
```

## Switching to DNF (YUM's Successor)

### Basic DNF Usage (Similar to YUM)
```bash
# Install DNF
yum install dnf

# Basic operations with DNF (syntax is very similar to YUM)
dnf install httpd
dnf update
dnf remove httpd
```

### DNF-Specific Features
```bash
# Automatic removal of no longer needed dependencies
dnf autoremove

# List available modules
dnf module list

# Enable a specific module stream
dnf module enable php:7.4

# Install a specific module
dnf module install php:7.4
```

## Practical Scenarios

### Find All Installed Security-Related Packages
```bash
yum list installed | grep -E 'security|firewall|ssl|crypto'
```

### Find Large Installed Packages
```bash
rpm -qa --queryformat '%{size} %{name}-%{version}\n' | sort -rn | head -20
```

### Clean Up Unused Dependencies
```bash
package-cleanup --leaves
package-cleanup --orphans
```

### Find What Package a Command Belongs To
```bash
# Find what package installed the 'top' command
rpm -qf $(which top)
```

### Upgrade to Latest Available Kernel
```bash
yum update kernel
```

### Find Configuration Files for a Package
```bash
rpm -qc httpd
```

### Trace Package Installation Process
```bash
yum install -v httpd
```

### Export List of Installed Packages for Reinstallation
```bash
rpm -qa > installed_packages.txt

# Later, to reinstall packages from this list
yum install $(cat installed_packages.txt)
```

These commands cover most package management tasks that a system administrator would need to perform on Red Hat-based systems, allowing for efficient management of software installations, updates, and repositories.