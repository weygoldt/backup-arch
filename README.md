# Arch linux system backup
A collection of useful scripts and notes I use to backup my arch-linux system as well as other data.

## Timeshift snapshots
Timeshift snapshots of a btrfs filesystem are stored on the root device and can be used to rollback the system, e.g. when a kernel update breaks the installation. Snapshots can be used to restore software failures. 

To setup timeshift snapshots, the packages `timeshift`, `timeshift-autosnap` and `cronie` to set up scheduled backups. 

## System backup
Backups of the timeshift snapshots are stored on a secondary device to be able to restore the system in case of a failure of the primary harddrive. System backups can be used to restore hardware failures. 

### Prerequisites
- System partition formated as btrfs with two subvolumes, `@` mounted at `/` as the root partition and `@home` mounted at `/home` as the home partition.
- Regular snapshots taken to `/run/timeshift/backup` in my case using [timeshift](https://github.com/teejee2008/timeshift).
- A secondary drive (here `dev/sda`). 
- Packages installed according to the scripts in [install arch](https://github.com/weygoldt/install-arch.git), particularly `timeshift` (or any other snapshot software), `cronie` for cronjobs and [`btrbk`](https://github.com/digint/btrbk) which handles the actual backups.

### Setup
Create a btrfs partition on the secondary drive. 
```sh
# to wipe the drive
sudo umount /dev/sda2
sudo gdisk /dev/sda2

o # wipe the drive
w # to write the changes
n # for a new partition
```
Set parition size and number according to preferences and use the linux filesystem. My backup partition is `/dev/sda2`. Make a btrfs file system.
```sh
mkfs.btrfs /dev/sda2
```
Mount the file system and create the subvolumes.
```sh
sudo mkdir /mnt/backup              # make a mntpnt
sudo mount /dev/sda2                # mount
sudo btrfs subvol create @backups   # make backups subvol
sudo btrfs subvol create @data      # also added subvol for other stuff 
```
Unmount `/dev/sda2` and instead mount the subvolumes `@backups` and `@data`
```sh
sudo umount /dev/sda2               # unmount
sudo rmdir /mnt/backup              # remove mntpnt created earlier
sudo mkdir /mnt{backups,data}       # create mntpnt for subvols

# mount subvols with options (hence -o)
mount -o noatime,space_cache=v2,compress=zstd,discard=async,subvol=@backups /dev/sda2 /mnt/backups
mount -o noatime,space_cache=v2,compress=zstd,discard=async,subvol=@data /dev/sda2 /mnt/data
```

## Data backup
In addition, I backup valuable data on the secondary internal device as well as an external harddrive. 
