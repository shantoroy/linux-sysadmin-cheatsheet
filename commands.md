# Linux Commands Cheatsheet for System Administrators

A comprehensive reference guide for Linux system administrators featuring essential commands organized by category, with examples and usage notes. Perfect for daily administration tasks and RHCSA exam preparation.

## Table of Contents

- [System Information](#system-information)
- [User and Group Management](#user-and-group-management)
- [File and Directory Operations](#file-and-directory-operations)
- [File Permissions and Ownership](#file-permissions-and-ownership)
- [Process Management](#process-management)
- [Package Management](#package-management)
- [Service Management (systemd)](#service-management-systemd)
- [Networking](#networking)
- [Disk and File System Management](#disk-and-file-system-management)
- [Text Processing and Editing](#text-processing-and-editing)
- [Archiving and Compression](#archiving-and-compression)
- [System Monitoring](#system-monitoring)
- [Logging and Journaling](#logging-and-journaling)
- [Scheduling Tasks](#scheduling-tasks)
- [Security and Access Control](#security-and-access-control)
- [SSH and Remote Management](#ssh-and-remote-management)
- [Kernel Management](#kernel-management)
- [Boot Management](#boot-management)
- [Containerization Basics](#containerization-basics)
- [Shell Scripting Essentials](#shell-scripting-essentials)
- [SELinux Management](#selinux-management)
- [Performance Tuning](#performance-tuning)
- [Backup and Recovery](#backup-and-recovery)
- [Virtualization](#virtualization)

## System Information

| Command | Description | Example |
|---------|-------------|---------|
| `uname` | Display system information | `uname -a` (all info) |
| `hostname` | Show or set system hostname | `hostname` |
| `uptime` | Show how long the system has been running | `uptime` |
| `date` | Display or set date and time | `date` |
| `timedatectl` | Control system time and date | `timedatectl status` |
| `lsb_release` | Display Linux Standard Base info | `lsb_release -a` |
| `cat /etc/os-release` | Show OS version information | `cat /etc/os-release` |
| `dmidecode` | Display hardware info from BIOS | `dmidecode -t system` |
| `lshw` | List hardware | `lshw -short` |
| `lscpu` | Display CPU info | `lscpu` |
| `lspci` | List PCI devices | `lspci` |
| `lsusb` | List USB devices | `lsusb` |
| `lsblk` | List block devices | `lsblk` |
| `free` | Show memory usage | `free -h` (human-readable) |
| `vmstat` | Report virtual memory statistics | `vmstat 1` (update every second) |

## User and Group Management

| Command | Description | Example |
|---------|-------------|---------|
| `useradd` | Create a new user | `useradd -m -s /bin/bash username` |
| `usermod` | Modify user account | `usermod -aG wheel username` (add to wheel group) |
| `userdel` | Delete a user | `userdel -r username` (remove home dir too) |
| `passwd` | Change user password | `passwd username` |
| `chage` | Change user password expiry info | `chage -l username` (list aging info) |
| `groupadd` | Create a new group | `groupadd groupname` |
| `groupmod` | Modify a group | `groupmod -n newname oldname` |
| `groupdel` | Delete a group | `groupdel groupname` |
| `id` | Display user and group IDs | `id username` |
| `whoami` | Display current username | `whoami` |
| `w` | Show who is logged in and what they're doing | `w` |
| `who` | Show who is logged in | `who` |
| `last` | Show last logins | `last` |
| `lastlog` | Show the last login of all users | `lastlog` |
| `su` | Switch user | `su - username` (with environment) |
| `sudo` | Execute command as another user | `sudo command` |
| `visudo` | Edit sudoers file safely | `visudo` |
| `getent` | Get entries from admin databases | `getent passwd username` |

## File and Directory Operations

| Command | Description | Example |
|---------|-------------|---------|
| `ls` | List directory contents | `ls -la` (long format, including hidden) |
| `pwd` | Print working directory | `pwd` |
| `cd` | Change directory | `cd /path/to/directory` |
| `mkdir` | Create directories | `mkdir -p parent/child` (create parents too) |
| `rmdir` | Remove empty directories | `rmdir directory` |
| `touch` | Create empty file or update timestamp | `touch file.txt` |
| `cp` | Copy files and directories | `cp -r source/ destination/` (recursive) |
| `mv` | Move/rename files and directories | `mv oldname newname` |
| `rm` | Remove files or directories | `rm -rf directory/` (recursive, force) |
| `ln` | Create links | `ln -s target linkname` (symbolic) |
| `find` | Search for files | `find /path -name "pattern"` |
| `locate` | Find files by name | `locate filename` |
| `updatedb` | Update locate database | `updatedb` |
| `stat` | Display file or file system status | `stat file` |
| `file` | Determine file type | `file filename` |
| `readlink` | Print resolved symbolic links | `readlink -f symlink` |
| `realpath` | Print resolved path | `realpath path` |
| `tree` | List contents in tree format | `tree -L 2` (depth 2) |
| `dirname` | Strip last component from path | `dirname /path/to/file` |
| `basename` | Strip directory from filename | `basename /path/to/file` |

## File Permissions and Ownership

| Command | Description | Example |
|---------|-------------|---------|
| `chmod` | Change file permissions | `chmod 755 file` or `chmod u+x file` |
| `chown` | Change file owner and group | `chown user:group file` |
| `chgrp` | Change group ownership | `chgrp group file` |
| `umask` | Set default permissions | `umask 022` |
| `getfacl` | Get file ACL | `getfacl file` |
| `setfacl` | Set file ACL | `setfacl -m u:user:rwx file` |
| `lsattr` | List file attributes on Linux file systems | `lsattr file` |
| `chattr` | Change attributes on Linux file systems | `chattr +i file` (make immutable) |

## Process Management

| Command | Description | Example |
|---------|-------------|---------|
| `ps` | Report process status | `ps aux` (all processes) |
| `pgrep` | Look up processes by name | `pgrep process_name` |
| `top` | Display Linux processes | `top` |
| `htop` | Interactive process viewer | `htop` |
| `kill` | Send signal to process | `kill -9 PID` (SIGKILL) |
| `pkill` | Kill processes by name | `pkill process_name` |
| `killall` | Kill processes by name | `killall process_name` |
| `nice` | Run with modified scheduling priority | `nice -n 10 command` |
| `renice` | Alter priority of running process | `renice -n 10 -p PID` |
| `nohup` | Run command immune to hangups | `nohup command &` |
| `fg` | Bring job to foreground | `fg %job_number` |
| `bg` | Send job to background | `bg %job_number` |
| `jobs` | List active jobs | `jobs` |
| `pstree` | Display tree of processes | `pstree` |
| `pidof` | Find process ID of a program | `pidof program_name` |
| `time` | Time command execution | `time command` |
| `timeout` | Run command with time limit | `timeout 10s command` |
| `watch` | Execute command periodically | `watch -n 1 'command'` |
| `at` | Schedule commands for later execution | `at now + 1 hour` |
| `fuser` | Show which processes use files/sockets | `fuser -m /mount/point` |
| `lsof` | List open files | `lsof -p PID` |

## Package Management

### RPM-based Systems (RHEL, CentOS, Fedora)

| Command | Description | Example |
|---------|-------------|---------|
| `rpm` | Low-level package manager | `rpm -qa` (list all packages) |
| `rpm` | Verify package | `rpm -V package` |
| `rpm` | Install package | `rpm -ivh package.rpm` |
| `dnf` | Package manager | `dnf install package` |
| `dnf` | Update packages | `dnf update` |
| `dnf` | Remove packages | `dnf remove package` |
| `dnf` | Search for package | `dnf search keyword` |
| `dnf` | List repositories | `dnf repolist` |
| `dnf` | List package groups | `dnf group list` |
| `dnf` | Install package group | `dnf group install "group name"` |
| `yum` | Legacy package manager | `yum install package` |
| `createrepo` | Create RPM repository | `createrepo /path/to/repo` |

### DEB-based Systems (Debian, Ubuntu)

| Command | Description | Example |
|---------|-------------|---------|
| `dpkg` | Low-level package manager | `dpkg -l` (list packages) |
| `dpkg` | Install package | `dpkg -i package.deb` |
| `apt` | Package manager | `apt install package` |
| `apt` | Update package lists | `apt update` |
| `apt` | Upgrade packages | `apt upgrade` |
| `apt` | Remove packages | `apt remove package` |
| `apt` | Autoremove unused dependencies | `apt autoremove` |
| `apt` | Search for package | `apt search keyword` |
| `apt-cache` | Query package cache | `apt-cache show package` |
| `apt-get` | Legacy package handling | `apt-get install package` |
| `aptitude` | Alternative package manager | `aptitude install package` |

## Service Management (systemd)

| Command | Description | Example |
|---------|-------------|---------|
| `systemctl` | Control system and service manager | `systemctl status service` |
| `systemctl` | Start service | `systemctl start service` |
| `systemctl` | Stop service | `systemctl stop service` |
| `systemctl` | Restart service | `systemctl restart service` |
| `systemctl` | Enable service at boot | `systemctl enable service` |
| `systemctl` | Disable service at boot | `systemctl disable service` |
| `systemctl` | Check if service is enabled | `systemctl is-enabled service` |
| `systemctl` | Check if service is active | `systemctl is-active service` |
| `systemctl` | Reload systemd | `systemctl daemon-reload` |
| `systemctl` | List all units | `systemctl list-units` |
| `systemctl` | List unit files | `systemctl list-unit-files` |
| `systemd-analyze` | Analyze boot time | `systemd-analyze` |
| `journalctl` | Query systemd journal | `journalctl -u service` |
| `journalctl` | Follow logs | `journalctl -f` |
| `journalctl` | Logs since boot | `journalctl -b` |

## Networking

| Command | Description | Example |
|---------|-------------|---------|
| `ip` | Show/manipulate routing, devices, etc. | `ip addr show` |
| `ip` | Show network interfaces | `ip link show` |
| `ip` | Show routing table | `ip route show` |
| `ifconfig` | Configure network interface (legacy) | `ifconfig eth0` |
| `route` | Show/manipulate IP routing table (legacy) | `route -n` |
| `ss` | Socket statistics | `ss -tuln` (TCP/UDP listening) |
| `netstat` | Network statistics (legacy) | `netstat -tuln` |
| `ping` | Send ICMP ECHO_REQUEST | `ping -c 4 host` |
| `traceroute` | Print route packets trace to host | `traceroute host` |
| `tracepath` | Trace path to a network host | `tracepath host` |
| `dig` | DNS lookup utility | `dig domain` |
| `host` | DNS lookup utility | `host domain` |
| `nslookup` | Query DNS servers | `nslookup domain` |
| `whois` | Client for whois directory service | `whois domain` |
| `hostname` | Show or set system hostname | `hostname` |
| `hostnamectl` | Control system hostname | `hostnamectl set-hostname name` |
| `nmcli` | NetworkManager command-line tool | `nmcli con show` |
| `nmtui` | NetworkManager text user interface | `nmtui` |
| `ethtool` | Query/control network drivers | `ethtool eth0` |
| `iwconfig` | Configure wireless interface | `iwconfig wlan0` |
| `iftop` | Display bandwidth usage | `iftop` |
| `tcpdump` | Dump traffic on a network | `tcpdump -i eth0` |
| `nmap` | Network exploration and security auditing | `nmap host` |
| `curl` | Transfer data from/to a server | `curl -O http://url/file` |
| `wget` | Retrieve files from the web | `wget http://url/file` |
| `ssh` | OpenSSH SSH client | `ssh user@host` |
| `scp` | Secure copy | `scp file user@host:/path` |
| `rsync` | Remote file sync | `rsync -avz source/ dest/` |
| `firewall-cmd` | Firewalld command-line client | `firewall-cmd --list-all` |
| `iptables` | Administration tool for IPv4 packet filtering | `iptables -L` |
| `nc` | Netcat, networking utility | `nc -zv host port` |

## Disk and File System Management

| Command | Description | Example |
|---------|-------------|---------|
| `df` | Report file system disk space usage | `df -h` |
| `du` | Estimate file space usage | `du -sh /path` |
| `fdisk` | Manipulate disk partition table | `fdisk -l` |
| `gdisk` | GPT fdisk interactive editor | `gdisk /dev/sda` |
| `parted` | Partition manipulation program | `parted -l` |
| `blkid` | Locate/print block device attributes | `blkid` |
| `lsblk` | List block devices | `lsblk` |
| `mkfs` | Build a Linux file system | `mkfs.ext4 /dev/sda1` |
| `mkswap` | Set up a swap area | `mkswap /dev/sda2` |
| `swapon` | Enable swap | `swapon /dev/sda2` |
| `swapoff` | Disable swap | `swapoff /dev/sda2` |
| `mount` | Mount a file system | `mount /dev/sda1 /mnt` |
| `umount` | Unmount a file system | `umount /mnt` |
| `fstab` | Static file system information | `nano /etc/fstab` |
| `findmnt` | Find a file system | `findmnt /mnt` |
| `tune2fs` | Adjust tunable file system parameters | `tune2fs -l /dev/sda1` |
| `fsck` | Check and repair a Linux file system | `fsck /dev/sda1` |
| `xfs_repair` | Repair an XFS file system | `xfs_repair /dev/sda1` |
| `smartctl` | Control and monitor SMART disks | `smartctl -a /dev/sda` |
| `badblocks` | Search for bad blocks | `badblocks -v /dev/sda` |
| `e2label` | Change the label on an ext2/3/4 file system | `e2label /dev/sda1 label` |
| `resize2fs` | Resize ext2/3/4 file system | `resize2fs /dev/sda1` |
| `dd` | Convert and copy a file | `dd if=/dev/zero of=/file bs=1M count=100` |
| `lvmdiskscan` | Scan for LVM physical volumes | `lvmdiskscan` |
| `pvcreate` | Initialize physical volume for LVM | `pvcreate /dev/sda1` |
| `pvdisplay` | Display LVM physical volumes | `pvdisplay` |
| `vgcreate` | Create a volume group | `vgcreate vg0 /dev/sda1` |
| `vgdisplay` | Display volume groups | `vgdisplay` |
| `lvcreate` | Create a logical volume | `lvcreate -L 10G -n lv0 vg0` |
| `lvdisplay` | Display logical volumes | `lvdisplay` |
| `lvextend` | Extend logical volume | `lvextend -L +5G /dev/vg0/lv0` |
| `vgextend` | Add physical volumes to a volume group | `vgextend vg0 /dev/sdb1` |

## Text Processing and Editing

| Command | Description | Example |
|---------|-------------|---------|
| `cat` | Concatenate and print files | `cat file` |
| `tac` | Concatenate and print files in reverse | `tac file` |
| `less` | View file contents (forward) | `less file` |
| `more` | View file contents (forward only) | `more file` |
| `head` | Output the first part of files | `head -n 10 file` |
| `tail` | Output the last part of files | `tail -n 10 file` |
| `grep` | Search text using patterns | `grep 'pattern' file` |
| `egrep` | Extended grep (regex) | `egrep 'pattern1|pattern2' file` |
| `sed` | Stream editor | `sed 's/old/new/g' file` |
| `awk` | Pattern scanning and processing | `awk '{print $1}' file` |
| `cut` | Remove sections from lines | `cut -d: -f1 /etc/passwd` |
| `sort` | Sort lines of text files | `sort file` |
| `uniq` | Report or omit repeated lines | `uniq file` |
| `wc` | Print newline, word, byte counts | `wc -l file` (line count) |
| `diff` | Compare files line by line | `diff file1 file2` |
| `vimdiff` | Edit two or more files with vim and show diff | `vimdiff file1 file2` |
| `comm` | Compare two sorted files line by line | `comm file1 file2` |
| `tr` | Translate or delete characters | `tr 'a-z' 'A-Z' < file` |
| `tee` | Read from stdin and write to stdout/files | `command | tee file` |
| `vi/vim` | Text editor | `vim file` |
| `nano` | Simple text editor | `nano file` |
| `emacs` | Text editor | `emacs file` |
| `join` | Join lines of two files on a common field | `join file1 file2` |
| `paste` | Merge lines of files | `paste file1 file2` |
| `column` | Columnate lists | `command | column -t` |
| `strings` | Print printable characters in files | `strings binary_file` |
| `hexdump` | ASCII, decimal, hexadecimal dump | `hexdump -C file` |

## Archiving and Compression

| Command | Description | Example |
|---------|-------------|---------|
| `tar` | Archive files | `tar -cvf archive.tar files/` |
| `tar` | Extract archive | `tar -xvf archive.tar` |
| `tar` | Create compressed archive (gzip) | `tar -czvf archive.tar.gz files/` |
| `tar` | Extract compressed archive (gzip) | `tar -xzvf archive.tar.gz` |
| `gzip` | Compress files | `gzip file` |
| `gunzip` | Expand files | `gunzip file.gz` |
| `bzip2` | Compress files | `bzip2 file` |
| `bunzip2` | Expand files | `bunzip2 file.bz2` |
| `xz` | Compress files | `xz file` |
| `unxz` | Expand files | `unxz file.xz` |
| `zip` | Package and compress files | `zip -r archive.zip directory/` |
| `unzip` | Extract compressed files | `unzip archive.zip` |
| `7z` | 7-Zip file archiver | `7z a archive.7z files/` |
| `cpio` | Copy files to/from archives | `ls | cpio -ov > archive.cpio` |

## System Monitoring

| Command | Description | Example |
|---------|-------------|---------|
| `top` | Display Linux processes | `top` |
| `htop` | Interactive process viewer | `htop` |
| `atop` | Advanced system & process monitor | `atop` |
| `free` | Display amount of free and used memory | `free -h` |
| `vmstat` | Report virtual memory statistics | `vmstat 1` |
| `iostat` | Report CPU and I/O statistics | `iostat 1` |
| `mpstat` | Report processors related statistics | `mpstat -P ALL` |
| `sar` | Collect, report system activity | `sar -u` (CPU) |
| `dstat` | Versatile resource statistics tool | `dstat` |
| `netstat` | Network statistics | `netstat -tuln` |
| `ss` | Socket statistics | `ss -tuln` |
| `iftop` | Display bandwidth usage | `iftop` |
| `nethogs` | Net top tool grouping by process | `nethogs eth0` |
| `ps` | Report process status | `ps aux` |
| `pstree` | Display process tree | `pstree` |
| `lsof` | List open files | `lsof -u username` |
| `fuser` | Identify processes using files | `fuser -m /mnt` |
| `uptime` | Show how long system has been running | `uptime` |
| `w` | Show who is logged on | `w` |
| `watch` | Execute a program periodically | `watch -n 1 'command'` |
| `strace` | Trace system calls and signals | `strace command` |
| `ltrace` | A library call tracer | `ltrace command` |
| `nmon` | Performance monitoring | `nmon` |

## Logging and Journaling

| Command | Description | Example |
|---------|-------------|---------|
| `journalctl` | Query systemd journal | `journalctl` |
| `journalctl` | Follow new messages | `journalctl -f` |
| `journalctl` | Show messages for specific service | `journalctl -u service` |
| `journalctl` | Show messages since boot | `journalctl -b` |
| `journalctl` | Show messages from previous boot | `journalctl -b -1` |
| `journalctl` | Show messages from specific time | `journalctl --since "2021-01-01"` |
| `tail` | Output the last part of files | `tail -f /var/log/syslog` |
| `less` | View file contents | `less /var/log/messages` |
| `dmesg` | Print or control kernel ring buffer | `dmesg` |
| `logrotate` | Rotate log files | `logrotate -f /etc/logrotate.conf` |
| `logger` | Enter messages in system log | `logger "message"` |
| `lastlog` | Reports recent login info for all users | `lastlog` |
| `ausearch` | Search audit logs | `ausearch -m login` |
| `aureport` | Report audit statistics | `aureport` |

## Scheduling Tasks

| Command | Description | Example |
|---------|-------------|---------|
| `crontab` | Schedule periodic jobs | `crontab -e` (edit) |
| `crontab` | List cron jobs | `crontab -l` |
| `at` | Execute commands at specified time | `at now + 1 hour` |
| `atq` | List pending jobs | `atq` |
| `atrm` | Delete jobs | `atrm job_number` |
| `batch` | Execute commands when system load permits | `batch` |
| `anacron` | Run commands periodically | `anacron -n` (run now) |
| `systemd-run` | Run command as systemd transient service | `systemd-run --on-calendar="*-*-* 12:00:00" command` |

## Security and Access Control

| Command | Description | Example |
|---------|-------------|---------|
| `chage` | Change user password expiry info | `chage -l username` |
| `passwd` | Change password | `passwd username` |
| `su` | Switch user | `su - username` |
| `sudo` | Execute command as another user | `sudo command` |
| `visudo` | Edit sudoers file | `visudo` |
| `chmod` | Change file permissions | `chmod 755 file` |
| `chown` | Change file owner and group | `chown user:group file` |
| `chgrp` | Change group ownership | `chgrp group file` |
| `umask` | Set default permissions | `umask 022` |
| `setfacl` | Set file access control lists | `setfacl -m u:user:rwx file` |
| `getfacl` | Get file access control lists | `getfacl file` |
| `chcon` | Change SELinux context | `chcon -t httpd_sys_content_t file` |
| `restorecon` | Restore file default SELinux contexts | `restorecon -v file` |
| `semanage` | SELinux policy management tool | `semanage fcontext -a -t httpd_sys_content_t '/var/www(/.*)?'` |
| `getenforce` | Get SELinux enforcement status | `getenforce` |
| `setenforce` | Set SELinux enforcement status | `setenforce 1` (enforcing) |
| `setsebool` | Set SELinux boolean value | `setsebool -P httpd_can_network_connect on` |
| `getsebool` | Get SELinux boolean value | `getsebool -a` (all booleans) |
| `sestatus` | SELinux status tool | `sestatus` |
| `audit2allow` | Generate SELinux policy allow rules | `audit2allow -a` |
| `firewall-cmd` | Firewalld command line client | `firewall-cmd --list-all` |
| `iptables` | Administer IPv4 packet filter rules | `iptables -L` |
| `fail2ban-client` | Fail2ban command line interface | `fail2ban-client status` |
| `openssl` | OpenSSL command-line tool | `openssl req -new -x509 -nodes -out server.crt -keyout server.key` |

## SSH and Remote Management

| Command | Description | Example |
|---------|-------------|---------|
| `ssh` | OpenSSH SSH client | `ssh user@host` |
| `ssh-keygen` | Generate SSH key pair | `ssh-keygen -t rsa` |
| `ssh-copy-id` | Install SSH key on a server | `ssh-copy-id user@host` |
| `scp` | Secure copy | `scp file user@host:/path` |
| `sftp` | Secure FTP | `sftp user@host` |
| `rsync` | Remote file synchronization | `rsync -avz source/ user@host:/dest/` |
| `ssh-agent` | SSH authentication agent | `eval $(ssh-agent)` |
| `ssh-add` | Add private key to agent | `ssh-add ~/.ssh/id_rsa` |
| `sshfs` | Mount remote filesystem | `sshfs user@host:/path /mnt/point` |
| `nc` | Netcat utility | `nc -l 8080` (listen) |
| `screen` | Terminal multiplexer | `screen` |
| `tmux` | Terminal multiplexer | `tmux` |
| `rdesktop` | Remote Desktop Protocol client | `rdesktop host` |
| `xrdp` | Remote Desktop Protocol server | `systemctl start xrdp` |
| `vnc` | Virtual Network Computing | Various implementations |

## Kernel Management

| Command | Description | Example |
|---------|-------------|---------|
| `uname` | Print system information | `uname -r` (kernel version) |
| `modprobe` | Add/remove modules from kernel | `modprobe module_name` |
| `lsmod` | Show loaded modules | `lsmod` |
| `modinfo` | Show info about a kernel module | `modinfo module_name` |
| `rmmod` | Remove a module from the kernel | `rmmod module_name` |
| `insmod` | Insert a module into the kernel | `insmod /path/to/module.ko` |
| `depmod` | Generate modules.dep | `depmod` |
| `sysctl` | Configure kernel parameters | `sysctl -a` (list all) |
| `sysctl` | Set kernel parameter | `sysctl -w kernel.hostname=value` |
| `dmesg` | Print kernel messages | `dmesg` |
| `kexec` | Load and boot into another kernel | `kexec -l /boot/vmlinuz --initrd=/boot/initrd --reuse-cmdline` |

## Boot Management

| Command | Description | Example |
|---------|-------------|---------|
| `grub2-mkconfig` | Generate GRUB configuration | `grub2-mkconfig -o /boot/grub2/grub.cfg` |
| `grub2-install` | Install GRUB bootloader | `grub2-install /dev/sda` |
| `grubby` | Command line tool for GRUB config | `grubby --default-kernel` |
| `dracut` | Create initramfs | `dracut -f` (force) |
| `systemd-analyze` | Analyze boot time | `systemd-analyze` |
| `systemd-analyze` | Analyze critical chain | `systemd-analyze critical-chain` |
| `bootctl` | Control EFI boot manager | `bootctl status` |
| `efibootmgr` | Manipulate EFI boot manager | `efibootmgr -v` |

## Containerization Basics

| Command | Description | Example |
|---------|-------------|---------|
| `podman` | Manage pods/containers/images | `podman ps` (list containers) |
| `podman` | Run container | `podman run -it image command` |
| `podman` | Build image | `podman build -t name .` |
| `podman` | Pull image | `podman pull image:tag` |
| `podman` | List images | `podman images` |
| `podman` | List containers | `podman ps -a` |
| `podman` | Stop container | `podman stop container_id` |
| `podman` | Remove container | `podman rm container_id` |
| `podman` | Remove image | `podman rmi image_id` |
| `podman` | Container logs | `podman logs container_id` |
| `podman` | Execute command in container | `podman exec -it container_id command` |
| `docker` | Docker container management | `docker ps` (list containers) |
| `docker` | Run container | `docker run -it image command` |
| `docker-compose` | Define multi-container applications | `docker-compose up` |
| `buildah` | Build OCI container images | `buildah bud -t name .` |
| `skopeo` | Work with container images/registries | `skopeo copy docker://source docker://destination` |
| `cri-o` | Container Runtime Interface | Service management via systemd |


## Shell Scripting Essentials

| Command | Description | Example |
|---------|-------------|---------|
| `bash` | Bourne Again SHell | `bash script.sh` |
| `sh` | POSIX-compliant shell | `sh script.sh` |
| `echo` | Display a line of text | `echo "Hello World"` |
| `printf` | Format and print data | `printf "Name: %s\n" "$name"` |
| `read` | Read from standard input | `read -p "Enter name: " name` |
| `if` | Conditional statement | `if [ condition ]; then commands; fi` |
| `for` | Loop through items | `for i in {1..5}; do commands; done` |
| `while` | Loop while condition is true | `while [ condition ]; do commands; done` |
| `until` | Loop until condition is true | `until [ condition ]; do commands; done` |
| `case` | Conditional branching | `case $var in pattern) commands;; esac` |
| `test` | Evaluate expression | `test -f file` or `[ -f file ]` |
| `set` | Set/unset shell options and positional parameters | `set -e` (exit on error) |
| `export` | Set environment variable | `export VAR=value` |
| `source` | Execute commands from a file | `source script.sh` or `. script.sh` |
| `alias` | Create command alias | `alias ll='ls -la'` |
| `function` | Define function | `function name() { commands; }` |
| `exit` | Exit the shell | `exit 0` (success) or `exit 1` (failure) |
| `expr` | Evaluate expressions | `expr 5 + 5` |
| `let` | Evaluate arithmetic expressions | `let "a = 5 + 5"` |
| `$()` | Command substitution | `echo "Today is $(date)"` |
| `${}` | Parameter expansion | `echo "${var:-default}"` |
| `$?` | Exit status of last command | `echo $?` |
| `$#` | Number of positional parameters | `echo $#` |
| `$*` | All positional parameters | `echo $*` |
| `$@` | All positional parameters (preserves quoting) | `echo $@` |
| `$$` | Process ID of the shell | `echo $$` |
| `$!` | Process ID of last background command | `echo $!` |
| `xargs` | Build and execute command lines from standard input | `find . -name "*.txt" | xargs grep "pattern"` |

## SELinux Management

| Command | Description | Example |
|---------|-------------|---------|
| `getenforce` | Get SELinux enforcement status | `getenforce` |
| `setenforce` | Set SELinux enforcement status | `setenforce 1` (enforcing) |
| `sestatus` | SELinux status tool | `sestatus` |
| `seinfo` | Query SELinux policy | `seinfo -t` (list all types) |
| `sesearch` | Search SELinux policy | `sesearch --allow --source httpd_t` |
| `semanage` | SELinux policy management | `semanage port -l` (list ports) |
| `semanage` | Add custom port | `semanage port -a -t http_port_t -p tcp 8080` |
| `semanage` | List file contexts | `semanage fcontext -l` |
| `semanage` | Add file context | `semanage fcontext -a -t httpd_sys_content_t "/web(/.*)?"` |
| `setsebool` | Set SELinux boolean | `setsebool -P httpd_can_network_connect on` |
| `getsebool` | Get SELinux boolean | `getsebool -a` (list all) |
| `chcon` | Change file SELinux context | `chcon -t httpd_sys_content_t file` |
| `restorecon` | Restore default file context | `restorecon -Rv /path` |
| `audit2why` | Translate SELinux denials | `audit2why -a` (analyze all denials) |
| `audit2allow` | Generate policy from denials | `audit2allow -a -M mymodule` |
| `load_policy` | Load new SELinux policy | `load_policy` |
| `checkpolicy` | SELinux policy compiler | `checkpolicy -M -o policy.kern policy.conf` |
| `semodule` | Manage SELinux policy modules | `semodule -l` (list modules) |
| `semodule` | Install module | `semodule -i mymodule.pp` |

## Performance Tuning

| Command | Description | Example |
|---------|-------------|---------|
| `nice` | Run command with modified scheduling priority | `nice -n 10 command` |
| `renice` | Alter priority of running process | `renice -n 10 -p PID` |
| `ionice` | Set I/O scheduling class and priority | `ionice -c 2 -n 0 command` |
| `chrt` | Manipulate real-time attributes of a process | `chrt -f 99 command` |
| `tuned-adm` | Switch between tuning profiles | `tuned-adm list` (show profiles) |
| `tuned-adm` | Apply profile | `tuned-adm profile throughput-performance` |
| `sysctl` | Configure kernel parameters at runtime | `sysctl vm.swappiness=10` |
| `ulimit` | Control resource limits | `ulimit -n 4096` (file descriptors) |
| `cpupower` | CPU power management | `cpupower frequency-info` |
| `cpupower` | Set CPU governor | `cpupower frequency-set -g performance` |
| `hdparm` | Get/set SATA/IDE device parameters | `hdparm -tT /dev/sda` (test speed) |
| `blockdev` | Call block device ioctls | `blockdev --getra /dev/sda` (get readahead) |
| `blkid` | Locate/print block device attributes | `blkid` |
| `irqbalance` | Distribute IRQs across CPUs | `systemctl status irqbalance` |
| `numad` | NUMA affinity management daemon | `systemctl start numad` |
| `perf` | Performance analysis tools | `perf stat command` |
| `perf` | Record system profile | `perf record -g command` |
| `perf` | Report system profile | `perf report` |
| `tuna` | Application tuning GUI/CLI | `tuna --show_threads` |
| `schedtool` | Query/set CPU affinity and scheduling policies | `schedtool -a 0-3 -e command` |

## Backup and Recovery

| Command | Description | Example |
|---------|-------------|---------|
| `tar` | Create archive | `tar -cvzf backup.tar.gz /path/to/directory` |
| `tar` | Extract archive | `tar -xvzf backup.tar.gz` |
| `rsync` | Remote file synchronization | `rsync -avz --delete source/ destination/` |
| `dd` | Convert and copy a file | `dd if=/dev/sda of=/path/backup.img bs=4M` |
| `dump` | Ext filesystem backup | `dump -0uf /path/backup.dump /dev/sda1` |
| `restore` | Restore from ext backup | `restore -if /path/backup.dump` |
| `xfsdump` | XFS filesystem backup | `xfsdump -f /path/backup.xfsdump /dev/sda1` |
| `xfsrestore` | Restore from XFS backup | `xfsrestore -f /path/backup.xfsdump /mnt` |
| `cpio` | Copy files to/from archives | `find /path -print | cpio -o > archive.cpio` |
| `amanda` | Advanced backup system | Various commands |
| `bacula` | Network backup solution | Various commands |
| `duplicity` | Encrypted bandwidth-efficient backup | `duplicity /path scp://user@host/path` |
| `rdiff-backup` | Remote incremental backup | `rdiff-backup /source user@host::/destination` |
| `clonezilla` | Disk cloning/imaging | `clonezilla` |
| `partclone` | Partition cloning utility | `partclone.ext4 -c -s /dev/sda1 -o backup.img` |
| `fsarchiver` | Filesystem archiver | `fsarchiver savefs backup.fsa /dev/sda1` |
| `restic` | Fast, secure, efficient backup | `restic -r /path/to/repo backup /data` |
| `borg` | Deduplicating archiver with compression | `borg create ::archive /path` |

## Virtualization

| Command | Description | Example |
|---------|-------------|---------|
| `virsh` | Manage virtual machines | `virsh list --all` |
| `virsh` | Start VM | `virsh start vm_name` |
| `virsh` | Shutdown VM | `virsh shutdown vm_name` |
| `virsh` | Console access | `virsh console vm_name` |
| `virsh` | Edit VM configuration | `virsh edit vm_name` |
| `virsh` | Get VM info | `virsh dominfo vm_name` |
| `virsh` | Show VM resources | `virsh dommemstat vm_name` |
| `virt-install` | Create new VM | `virt-install --name=vm_name ...` |
| `virt-clone` | Clone existing VM | `virt-clone --original=source_vm --name=new_vm ...` |
| `virt-viewer` | Graphical console | `virt-viewer vm_name` |
| `virt-manager` | GUI for VM management | `virt-manager` |
| `qemu-img` | QEMU disk image utility | `qemu-img create -f qcow2 disk.qcow2 10G` |
| `qemu-img` | Convert image format | `qemu-img convert -f raw -O qcow2 disk.img disk.qcow2` |
| `qemu-system-x86_64` | QEMU emulator | `qemu-system-x86_64 -hda disk.qcow2 ...` |
| `vmware` | VMware commands | Various commands |
| `VBoxManage` | VirtualBox management | `VBoxManage list vms` |

## Advanced Networking

| Command | Description | Example |
|---------|-------------|---------|
| `ip route` | Show/manipulate routing table | `ip route show` |
| `ip route` | Add route | `ip route add 192.168.2.0/24 via 192.168.1.1` |
| `ip route` | Delete route | `ip route del 192.168.2.0/24` |
| `ip neighbor` | Show/manipulate neighbor objects | `ip neighbor show` |
| `ip link` | Network device configuration | `ip link set dev eth0 up` |
| `bridge` | Show/manipulate bridge addresses and ports | `bridge link show` |
| `vconfig` | VLAN configuration | `vconfig add eth0 10` |
| `brctl` | Ethernet bridge administration | `brctl addbr br0` |
| `iw` | Wireless configuration | `iw dev wlan0 scan` |
| `iwconfig` | Configure wireless interface | `iwconfig wlan0 essid "network"` |
| `nmcli connection` | Show connections | `nmcli connection show` |
| `nmcli device` | Show devices | `nmcli device status` |
| `nmcli radio` | Show radio switches | `nmcli radio wifi on` |
| `firewall-cmd` | Add service | `firewall-cmd --permanent --add-service=http` |
| `firewall-cmd` | Add port | `firewall-cmd --permanent --add-port=8080/tcp` |
| `firewall-cmd` | List services | `firewall-cmd --list-services` |
| `iptables` | Add rule | `iptables -A INPUT -p tcp --dport 80 -j ACCEPT` |
| `iptables` | Delete rule | `iptables -D INPUT 1` |
| `iptables-save` | Save rules | `iptables-save > /etc/iptables/rules.v4` |
| `tc` | Traffic control | `tc qdisc add dev eth0 root netem delay 100ms` |
| `nftables` | Packet filtering framework | `nft add table inet filter` |
| `openvpn` | VPN daemon | `openvpn --config client.ovpn` |

## Troubleshooting and Diagnostics

| Command | Description | Example |
|---------|-------------|---------|
| `strace` | Trace system calls and signals | `strace command` |
| `ltrace` | Library call tracer | `ltrace command` |
| `ldd` | Print shared library dependencies | `ldd /usr/bin/bash` |
| `lsof` | List open files | `lsof -p PID` |
| `fuser` | Show which processes use files | `fuser -m /mnt` |
| `pstree` | Display process tree | `pstree -p` |
| `pmap` | Display process memory map | `pmap PID` |
| `vmstat` | Report virtual memory statistics | `vmstat 1` |
| `iostat` | Report CPU and I/O statistics | `iostat -xz 1` |
| `mpstat` | Report CPU statistics | `mpstat -P ALL 1` |
| `sar` | Collect and report system activity | `sar -u 1 10` |
| `sysstat` | System performance tools | Various commands |
| `dstat` | Versatile tool for generating system resource statistics | `dstat -cdnpmgs --top-bio 5` |
| `iotop` | I/O monitoring | `iotop` |
| `ioping` | Monitor I/O latency | `ioping -c 10 /dev/sda` |
| `blktrace` | Block layer I/O tracing mechanism | `blktrace -d /dev/sda -o trace` |
| `slabtop` | Display kernel slab cache information | `slabtop` |
| `crash` | Kernel crash utility | `crash /path/to/vmcore /path/to/vmlinux` |
| `sosreport` | Collect system logs and configuration | `sosreport` |
| `kdump` | Kernel crash dumping mechanism | `systemctl enable kdump` |
| `kexec` | Load and boot into another kernel | `kexec -l /boot/vmlinuz --initrd=/boot/initramfs.img` |
