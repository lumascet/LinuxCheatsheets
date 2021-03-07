# ZFS

## Snapshot

```
zfs snapshot datapool/storage@backup-`date +%Y-%m-%d`
```

## Backup

### rpool

#### Full Backup

```
zfs snapshot rpool/ROOT/pve-1@backup-`date +%Y-%m-%d`
zfs snapshot -r rpool/data@backup-`date +%Y-%m-%d`
zfs list -t snapshot
zfs send -v rpool/ROOT/pve-1@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/rpool_root_`date +%Y-%m-%d`.gz
zfs send -Rv rpool/data@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/rpool_data_`date +%Y-%m-%d`.gz
```

#### Incremential Backup

```
zfs send -i rpool/ROOT/pve-1@backup-LAST rpool/ROOT/pve-1@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/rpool_root_diff_`date +%Y-%m-%d`.gz
zfs send -i rpool/data@backup-LAST rpool/data@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/rpool_data_`date +%Y-%m-%d`.gz
```

### datapool

#### Full Backup

```
zfs snapshot datapool/storage@backup-`date +%Y-%m-%d`
zfs snapshot datapool/media@backup-`date +%Y-%m-%d`
zfs list -t snapshot
zfs send datapool/storage@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/datapool_storage_`date +%Y-%m-%d`.gz
zfs send datapool/media@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/datapool_media_`date +%Y-%m-%d`.gz
```

#### Incremential Backup

```
zfs send -i datapool/storage@backup-LAST datapool/storage@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/datapool_storage_diff_`date +%Y-%m-%d`.gz
zfs send -i datapool/media@backup-LAST datapool/media@backup-`date +%Y-%m-%d` | pigz -9 -p 24 > /mnt/backup/datapool_media_diff_`date +%Y-%m-%d`.gz
```

### Verify Zipped Backup Stream

```
pigz -dc -p 24 rpool_root_2021-02-20.gz | zstreamdump
```

<https://pavelanni.github.io/oracle_solaris_11_labs/zfs/zfs_backup/>\
<https://docs.oracle.com/cd/E18752_01/html/819-5461/gbchx.html>

## Restore

### Full Backup restore

possible -F force to override existing datapool

```
pigz -dc -p 4 backup.gz | zfs recv -o compression=lz4 -o mountpoint=/data backuppool
```

### Incremential Restore

Previous Snapshot has to be applied, target pool cant differ from snapshot

```
pigz -dc -p 24 diffback.gz | zfs recv testpool
```

## Replace Boot drive (proxmox)

in order to properly boot proxmox from zfs drives you have to create an efi partition on each drive.

```
sgdisk /dev/diskbyid/DriveA -R /dev/diskbyid/DriveC
sgdisk -G /dev/diskbyid/DriveC
zpool status rpool
zpool replace rpool driveB-part3 driveC-part3
zpool status -v
pve-efiboot-tool format /dev/sdX2
pve-efiboot-tool init /dev/sdX2
zpool status -v
pve-efiboot-tool refresh
nano /etc/kernel/pve-efiboot-uuids #remove the drive that was removed
```

## Remove efiboot disk entry

```
blkid /dev/sd?? | grep -f /etc/kernel/pve-efiboot-uuids
nano /etc/kernel/pve-efiboot-uuids # remove desired entries
```

# Nextcloud

## Docker root Commands

### List Commands

```
docker exec --user www-data aebeacda743f php occ list
```

### Scan Files

```
docker exec --user www-data aebeacda743f php occ files:scan --all
```

### Repair

```
docker exec --user www-data aebeacda743f php occ maintenance:repair
```

## Renamig files with timestamp

[exiftool](https://exiftool.org/)

### rename to timestamp only

```
exiftool -d %Y-%m-%d_%H-%M-%S.%%e "-filename<filemodifydate" DIR
```

### append old filename

```
exiftool -d %Y-%m-%d_%H-%M-%S_%%f.%%e "-filename<filemodifydate" DIR
``` 

E.g.: `IMG_0004.JPG` \-> `2012-12-24_20-54-48_IMG_0004.JPG`

# LINUX

## List all disks /dev/sd? mapped with /dev/disk/by-id/

```
lsblk |awk 'NR==1{print $0" DEVICE-ID(S)"}NR>1{dev=$1;printf $0" ";system("find /dev/disk/by-id -lname \"*"dev"\" -printf \" %p\"");print "";}'|grep -v -E 'part|lvm'
```
