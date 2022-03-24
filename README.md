# Arch linux system backup
A collection of useful scripts and notes I use to backup my arch-linux system as well as other data.

## Timeshift snapshots
Timeshift snapshots of a btrfs filesystem are stored on the root device and can be used to rollback the system, e.g. when a kernel update breaks the installation. Snapshots can be used to restore software failures. 

To setup timeshift snapshots, the packages `timeshift`, `timeshift-autosnap` and `cronie` to set up scheduled backups. 

## System backup
Backups of the timeshift snapshots are stored on a secondary device to be able to restore the system in case of a failure of the primary harddrive. System backups can be used to restore hardware failures. 

## Data backup
In addition, I backup valuable data on the secondary internal device as well as an external harddrive. 
