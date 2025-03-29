# Advanced File System Operations

## LVM Management

### Create and extend an LVM setup
```bash
# Create physical volumes from disks
pvcreate /dev/sdb /dev/sdc

# Check physical volumes
pvs
pvdisplay

# Create a volume group
vgcreate data_vg /dev/sdb /dev/sdc

# Check volume group
vgs
vgdisplay data_vg

# Create logical volumes
# Use 10GB for mysql data
lvcreate -L 10G -n mysql_lv data_vg
# Use 5GB for web data
lvcreate -L 5G -n web_lv data_vg
# Use remaining space for backups
lvcreate -l 100%FREE -n backup_lv data_vg

# Check logical volumes
lvs
lvdisplay data_vg/mysql_lv

# Create filesystems
mkfs.ext4 /dev/data_vg/mysql_lv
mkfs.ext4 /dev/data_vg/web_lv
mkfs.ext4 /dev/data_vg/backup_lv

# Create mount points
mkdir -p /var/lib/mysql /var/www /var/backups

# Add entries to /etc/fstab
echo "/dev/data_vg/mysql_lv  /var/lib/mysql  ext4  defaults  0 2" >> /etc/fstab
echo "/dev/data_vg/web_lv    /var/www        ext4  defaults  0 2" >> /etc/fstab
echo "/dev/data_vg/backup_lv /var/backups    ext4  defaults  0 2" >> /etc/fstab

# Mount all
mount -a

# Extend a logical volume
# Add a new disk
pvcreate /dev/sdd
vgextend data_vg /dev/sdd

# Extend a logical volume by 20GB
lvextend -L +20G /dev/data_vg/backup_lv
# Resize the filesystem to use the new space
resize2fs /dev/data_vg/backup_lv

# Extend a logical volume to use all remaining space
lvextend -l +100%FREE /dev/data_vg/backup_lv
resize2fs /dev/data_vg/backup_lv
```

### Create a snapshot and restore from it
```bash
# Create a snapshot of a logical volume (1GB size for snapshot)
lvcreate -L 1G -s -n mysql_snap /dev/data_vg/mysql_lv

# Mount the snapshot read-only
mkdir -p /mnt/snapshot
mount -o ro /dev/data_vg/mysql_snap /mnt/snapshot

# Restore files from snapshot
cp -a /mnt/snapshot/important_file /var/lib/mysql/

# Unmount snapshot
umount /mnt/snapshot

# Remove the snapshot when done
lvremove /dev/data_vg/mysql_snap

# Restore entire volume from snapshot
# (Unmount original volume first)
umount /var/lib/mysql
lvconvert --merge /dev/data_vg/mysql_snap
# After merge, snapshot is gone and original LV is restored
mount /var/lib/mysql
```

## RAID Management

### Create and manage software RAID
```bash
# Install mdadm if not already installed
apt-get install mdadm

# Create a RAID 1 (mirror) with two disks
mdadm --create /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc

# Check RAID status
cat /proc/mdstat
mdadm --detail /dev/md0

# Create a filesystem on the RAID
mkfs.ext4 /dev/md0

# Mount the RAID
mkdir -p /mnt/raid
mount /dev/md0 /mnt/raid

# Add entry to /etc/fstab
echo "/dev/md0  /mnt/raid  ext4  defaults  0 2" >> /etc/fstab

# Save RAID configuration
mdadm --detail --scan >> /etc/mdadm/mdadm.conf
update-initramfs -u

# Add a spare disk to the RAID
mdadm --add /dev/md0 /dev/sdd

# Replace a failed disk
mdadm /dev/md0 --fail /dev/sdb
mdadm /dev/md0 --remove /dev/sdb
# Install new disk and add it
mdadm /dev/md0 --add /dev/sde
# RAID will rebuild automatically

# Check rebuild progress
cat /proc/mdstat

# Convert RAID 1 to RAID 5 (need to add more disks)
mdadm --grow /dev/md0 --level=5 --raid-devices=3 --add /dev/sdf

# Expand a RAID array with a new disk
mdadm --grow /dev/md0 --raid-devices=4 --add /dev/sdg
```

