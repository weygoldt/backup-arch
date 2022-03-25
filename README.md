# Arch linux system backup
A collection of useful scripts and notes I use to backup my arch-linux system as well as other data.

## System snapshots and backups
This manual describes how to schedule hourly snapshots on the live system on the system partition and automatically backup the snapshots incrementally onto a secondary drive.

### Prerequisites
- System partition formated as btrfs with two subvolumes, `@` mounted at `/` as the root partition and `@home` mounted at `/home` as the home partition.
- A secondary drive (here `dev/sda`). 
- Packages installed according to the scripts in [install arch](https://github.com/weygoldt/install-arch.git) e.g. `cronie` for cronjobs and [`btrbk`](https://github.com/digint/btrbk) which handles the actual snapshots and backups.

### Setup
#### Partitioning
##### 1. Mount the root of the primary btrfs device. 

In my case, only the two subdirectories `@` and `@home` were mounted at startup. Btrbk needs access to the root directory to create a directory e.g. `btrbk-snapshots` where the snapshots are stored next to the two subvolumes of the system. The root of a btrfs filesystem always has the `subvolid=5`, which we can use to mount it.
```sh
sudo mkdir /mnt/archlinux # named after system partition label
sudo mount -o defaults,subvolid=5 /dev/nvme0n1p2 /mnt/archlinux
``` 
Add the mountpoint to `fstab` and check if successful by `sudo mount -a`. The directory `/mnt/archlinux` should now include the two subvolumes `@` and `@home`.
##### 2. Create a btrfs partition on the secondary drive. 
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

# mount subvols
sudo mount -o defaults,subvol=@backups /dev/sda2 /mnt/backups
sudo mount -o defaults,subvol=@data /dev/sda2 /mnt/data
lsblk
```
Add the two subvolumes to the `fstab` using the `UUID` of the parent partition. Test if the new `fstab` is working by unmounting the volumes and then automically mounting all volumes in the `fstab` by running `sudo mount -a`. For the following, only the `@backups` subvolume is used.

#### btrbk
Btrbk is a cli-based software that allows the creation of read-only snapshots (i.e. they cannot be booted but only used to restore the system) on the root directory and simultaneously, backup the snapshots to another harddrive.

##### 1. Create directories for snapshots and backups
```sh
sudo mkdir /mnt/archlinux/btrbk-snapshots
sudo mkdir /mnt/backups/btrbk-backups
```

##### 2. Create a btrbk [configuration file](btrbk.conf)
I used the provided example file (stored in `/etc/btrbk/btrbk.conf.example`)and store it in in `/home/weygoldt/Data/projects/backup-arch/btrbk.conf`.

##### 3. Test if the config file works
Run btrbk with 
```sh
sudo btrbk -c /home/weygoldt/Data/projects/backup-arch/btrbk.conf -v -n run
```
 for a dry-run. The `-c` flag adds the path to the config file, the `-v` flag adds verbose output and the `-n` flag activates dry run. If no errors occur, the first real snapshots can be taken with

```sh
# to make a snapshot
sudo btrbk -c /home/weygoldt/Data/projects/backup-arch/btrbk.conf -v snapshot
```
The output should indicate that a snapshot was taken but no backup was taken. The snapshot should be stored at `/mnt/archlinux`. With the snapshots in place, the first backup can be created now with
```sh
# to make a backup
sudo btrbk -c /home/weygoldt/Data/projects/backup-arch/btrbk.conf -v -n resume
```
Note that the first backup will **not** be incremental, because logically, only the following backups can be incremental. If both snapshots and backups can be generated manullay, this can be implemented in a cronjob.

#### 4. Automate snapshots and backups


## Data backups
In addition, I backup valuable data on the secondary internal device as well as an external harddrive. 
