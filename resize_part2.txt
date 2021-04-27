Resize partitions and file systems of Linux data disks
Last Updated: Nov 09, 2020
When you resize disks, only the storage capacity of the disks is extended. File systems of the disks are not affected. Perform the following steps in this topic to resize file systems to extend the capacity of ECS instances.

Prerequisites
A snapshot is created to back up data.
To prevent data loss caused by incorrect operations, we recommend that you create a snapshot to back up your data. For more information, see Create a normal snapshot.

A data disk is resized in the console.
If no data disks are resized, see Resize disks online for Linux instances or Resize disks offline for Linux instances to resize a data disk.

Remotely connect to an ECS instance. For more information, see Overview.
Background information
The following configurations are used in the examples of this topic:
Operating system of the ECS instance: Alibaba Cloud Linux 2.1903 LTS 64-bit public image
Data disk: an ultra disk
Device name of the data disk: /dev/vdb
Adjust the command or parameter settings based on the actual operating system and device name of the data disk.



Check the partition table format and the file system type
Run the following command to check the partition table format of the data disk:
fdisk -lu /dev/vdb
In this example, the disk has a partition named /dev/vdb1.
If the value of System is Linux, the data disk uses the MBR partition table format.
If the value of System is GPT, the data disk uses the GPT partition table format.
[root@ecshost ~]# fdisk -lu /dev/vdb
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9277b47b

Device Boot Start End Blocks Id System
/dev/vdb1 2048 41943039 20970496 83 Linux
Run the following command to check the file system type of the existing partition:
blkid /dev/vdb1
In this example, the file system type of /dev/vdb1 is ext4.
[root@ecshost ~]# blkid /dev/vdb1
/dev/vdb1: UUID="e97bf1e2-fc84-4c11-9652-73********24" TYPE="ext4"
Note No results are returned if a data disk does not have partitions or file systems, or if a data disk has partitions but not file systems.
Run the following command to check the status of the file system:
ext* file system: e2fsck -n /dev/vdb1
XFS file system: xfs_repair -n /dev/vdb1
Notice In this example, the file system is in the clean state, which indicates that the file system is normal. If the file system is not in the clean state, troubleshoot the file system.
[root@ecshost ~]# e2fsck -n /dev/vdb1
Warning! /dev/vdb1 is mounted.
Warning: skipping journal recovery because doing a read-only filesystem check.
/dev/vdb1: clean, 11/1310720 files, 126322/5242624 blocks
Select a method to resize partitions or file systems
Select a resizing method based on the partition format and file system type.

Scenario	Resizing method
The data disk has partitions and file systems.	
To resize the existing MBR partitions of the data disk, see Option 1: Resize existing MBR partitions.
To resize the disk to create more MBR partitions, see Option 2: Add and format MBR partitions.
To resize the existing GPT partitions of the data disk, see Option 3: Resize existing GPT partitions.
To resize the disk to create more GPT partitions, see Option 4: Add and format GPT partitions.
The new data disk does not have partitions or file systems.	After you resize the data disk in the console, see Partition and format a data disk or Partition and format a data disk larger than 2 TiB.
The raw data disk has a file system but no partitions.	After you resize the data disk in the console, see Option 5: Resize the file system of a raw data disk.
The data disk is not attached to any instance.	After you attach the data disk to an instance, perform the following steps in this topic to resize the data disk.
Note
If a data disk contains an MBR partition, the data disk cannot be resized to 2 TiB or larger. To prevent data loss, we recommend that you create a disk larger than 2 TiB. Format a GPT partition and copy the data in the MBR partition to the GPT partition. For more information, see Partition and format a data disk larger than 2 TiB.
If data disks fail to be resized due to the problem of the resizing or formatting tool, you can upgrade the tool version, or uninstall and reinstall the tool.
Option 1: Resize existing MBR partitions
Note To prevent data loss, we recommend that you do not resize partitions and file systems that are mounted to ECS instances. Unmount the mounted partitions (umount) first. After you resize the partitions and they can be normally used, mount the partitions (mount) again. We recommend that you use the following methods for different Linux kernel versions:
If the instance kernel version is earlier than 3.6, unmount the partition, modify the partition table, and then resize the file system.
If the instance kernel version is 3.6 or later, modify the corresponding partition table, notify the kernel to update the partition table, and then resize the file system.
Perform the following operations to resize an existing MBR partition:

Modify the partition table.
Run the following command to view the partition information and record the start and end sectors of the existing partition:
fdisk -lu /dev/vdb
In this example, the start sector number of the /dev/vdb1 partition is 2048 and the end sector number is 41943039.

[root@ecshost ~]# fdisk -lu /dev/vdb
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9277b47b

Device Boot Start End Blocks Id System
/dev/vdb1 2048 41943039 20970496 83 Linux
View the mount path of the data disk. Unmount the partition based on the returned file path and wait until the partition is unmounted.
[root@ecshost ~]# mount | grep "/dev/vdb"
/dev/vdb1 on /mnt type ext4 (rw,relatime,data=ordered)
[root@ecshost ~]# umount /dev/vdb1
[root@ecshost ~]# mount | grep "/dev/vdb"
Run the fdisk command to delete the existing partition.
Warning If errors occur when you delete a partition, data stored on the partition may be deleted. To avoid data loss, back up important data such as user data in a database before you delete a partition.
Run the fdisk -u /dev/vdb command to partition the data disk.
Enter p to show the partition table.
Enter d to delete the partition.
Enter p to confirm that the partition has been deleted.
Enter w to save changes and exit.
[root@ecshost ~]# fdisk -u /dev/vdb
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): p
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9277b47b
Device Boot Start End Blocks Id System
/dev/vdb1 2048 41943039 20970496 83 Linux
Command (m for help): d
Selected partition 1
Partition 1 is deleted
Command (m for help): p
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9277b47b
Device Boot Start End Blocks Id System
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
Run the fdisk command to create a partition.
Run the fdisk -u /dev/vdb command to partition the data disk.
Enter p to show the partition table.
Enter n to create a partition.
Enter p to select the primary partition type.
Enter <partition number> to select the partition number. In this example, 1 is selected.
Set the start and end sector numbers for the new partition.
Warning The start sector number of the new partition must be the same as that of the existing partition. The end sector number must be greater than that of the existing partition. Otherwise, the resizing operation will fail.
Enter w to save changes and exit.
In this example, the /dev/vdb1 partition is resized from 20 GiB to 40 GiB.

[root@ecshost ~]# fdisk -u /dev/vdb
Welcome to fdisk (util-linux 2.23.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
Command (m for help): p
Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9277b47b
Device Boot Start End Blocks Id System
Command (m for help): n
Partition type:
p primary (0 primary, 0 extended, 4 free)
e extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-83886079, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-83886079, default 83886079):
Partition 1 of type Linux and of size 40 GiB is set
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
Notify the kernel that the partition table of the data disk has been updated.
Run the partprobe /dev/vdb or partx -u /dev/vdb1 command to notify the kernel that the partition table of the data disk has been modified and the kernel must be synchronized.
Run the lsblk /dev/vdb command to verify that the partition table has been resized.
Run the e2fsck -f /dev/vdb1 command to check the file system again and verify that the file system is in the clean state after the partition is resized.
Note If the file system is not in the clean state after you run the preceding command, you can run the e2fsck -n /dev/vdb1 command to check the file system again.
Resize the file system.
For an ext* file system such as ext3 or ext4, run the following commands in sequence to resize the ext* file system and remount the partition:
[root@ecshost ~]# resize2fs /dev/vdb1
resize2fs 1.43.5 (04-Aug-2017)
Resizing the filesystem on /dev/vdb1 to 7864320 (4k) blocks.
The filesystem on /dev/vdb1 is now 7864320 blocks long.
[root@ecshost ~]# mount /dev/vdb1 /mnt
For an XFS file system, run the following commands in sequence to remount the partition and then resize the XFS file system:
Note The new version xfs_growfs identifies the device to be resized based on the mount point. Example: xfs_growfs /mnt. You can run the xfs_growfs --help command to check how to use xfs_growfs of different versions.
[root@ecshost ~]# mount /dev/vdb1 /mnt/
[root@ecshost ~]# xfs_growfs /mnt
meta-data=/dev/vdb1              isize=512    agcount=4, agsize=1310720 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=5242880, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 5242880 to 7864320
Option 2: Add and format MBR partitions
If you want to resize the disk to create more MBR partitions, perform the following operations to resize the file system of the instance:

Run the fdisk -u /dev/vdb command to create a partition.
In this example, the /dev/vdb2 partition is created for the newly added 20 GiB disk space.

[root@ecshost ~]# fdisk -u /dev/vdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write commad.

Command (m for help): p

Disk /dev/vdb: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x2b31a2a3

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048    41943039    20970496   83  Linux

Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): p
Partition number (2-4, default 2): 2
First sector (41943040-83886079, default 41943040):
Using default value 41943040
Last sector, +sectors or +size{K,M,G} (41943040-83886079, default 83886079):
Using default value 83886079
Partition 2 of type Linux and of size 20 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
Run the lsblk /dev/vdb command to view the partition.
[root@ecshost ~]# lsblk /dev/vdb
NAME   MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
vdb    253:16   0  40G  0 disk
├─vdb1 253:17   0  20G  0 part
└─vdb2 253:18   0  20G  0 part
Format the new partition.
To create an ext4 file system, run the mkfs.ext4 /dev/vdb2 command.
[root@ecshost ~]# mkfs.ext4 /dev/vdb2
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
1310720 inodes, 5242880 blocks
262144 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=2153775104
160 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000

Allocating group tables: done
Writing inode tables: done
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done
[root@ecshost ~]# blkid /dev/vdb2
/dev/vdb2: UUID="e3f336dc-d534-4fdd-****-b6ff1a55bdbb" TYPE="ext4"
To create an ext3 file system, run the mkfs.ext3 /dev/vdb2 command.
To create an XFS file system, run the mkfs.xfs -f /dev/vdb2 command.
To create a btrfs file system, run the mkfs.btrfs /dev/vdb2 command.
Run the mount /dev/vdb2 /mnt command to mount the partition.
Run the df -h command to check the current capacity and usage of the data disk.
If information about the new file system is displayed, the partition is mounted.
[root@ecshost ~]# df -h
Filesystem Size Used Avail Use% Mounted on
/dev/vda1 40G 1.6G 36G 5% /
devtmpfs 3.9G 0 3.9G 0% /dev
tmpfs 3.9G 0 3.9G 0% /dev/shm
tmpfs 3.9G 460K 3.9G 1% /run
tmpfs 3.9G 0 3.9G 0% /sys/fs/cgroup
/dev/vdb2 9.8G 37M 9.2G 1% /mnt
tmpfs 783M 0 783M 0% /run/user/0
Option 3: Resize existing GPT partitions
Perform the following operations to resize an existing GPT partition:



![alt text](https://raw.githubusercontent.com/reinaldoca/linux/main/p70886.png)

 

View the mount path of the data disk. Unmount the partition based on the returned file path and wait until the partition is unmounted.
[root@ecshost ~]# mount | grep "/dev/vdb"
/dev/vdb1 on /mnt type ext4 (rw,relatime,data=ordered)
[root@ecshost ~]# umount /dev/vdb1
[root@ecshost ~]# mount | grep "/dev/vdb"
Use the Parted tool to allocate capacity for the existing GPT partition.
Run the parted /dev/vdb command to start the parted tool.
To view the Parted tool instructions, run the help command.

Run the print command to view the partition information and record the partition number and start sector value of the existing partition.
If Fix/Ignore/Cancel? or Fix/Ignore? is displayed, enter Fix.

In this example, the size of the existing partition is 1 TiB. The partition number (the value of Number) is 1. The start sector number (the value of Start) is 1049kB.

resize-gpt-start
Run the rm <Partition number> command to unmount the existing partition.
In this example, the partition number of the existing partition is 1. Therefore, the following command is used:

rm 1
Run the mkpart primary <the start sector number of the existing partition> <the percentage of allocated capacity> command to recreate the primary partition.
In this example, the start sector number of the existing partition is 1049kB and the 3 TiB of total capacity is allocated to the partition. Therefore, the following command is used:

mkpart primary 1049kB 100%
Run the print command to check whether the new partition is created.
The following figure shows that the new GPT partition number is still 1, but the capacity of the partition has been increased to 3 TiB.

GPT partitioning result
Run the quit command to exit the Parted tool.
The following section shows the complete sample code:
[root@ecshost ~]# parted /dev/vdb
GNU Parted 3.1
Using /dev/vdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Error: The backup GPT table is not at the end of the disk, as it should be.
This might mean that another operating system believes the disk is smaller.
Fix, by moving the backup to the end (and removing the old backup)?
Fix/Ignore/Cancel? Fix
Warning: Not all of the space available to /dev/vdb appears to be used, you can
fix the GPT to use all of the space (an extra 4294967296 blocks) or continue
with the current setting?
Fix/Ignore? Fix
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 3299GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  1100GB  1100GB  ext4         primary

(parted) rm 1
(parted) mkpart primary 1049kB 100%
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 3299GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name     Flags
 1      1049kB  3299GB  3299GB  ext4         primary

(parted) quit
Information: You may need to update /etc/fstab.
Run the fsck -f /dev/vdb1 command to check the file system for consistency.
[root@ecshost ~]# fsck -f /dev/vdb1
fsck from util-linux 2.23.2
e2fsck 1.43.5 (04-Aug-2017)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
/dev/vdb1: 11/67108864 files (0.0% non-contiguous), 4265369/268434944 blocks
Resize the file system corresponding to the partition and remount the partition.
For an ext* file system such as ext3 or ext4, run the following commands in sequence to resize the ext* file system of the new partition and remount the partition:
[root@ecshost ~]# resize2fs /dev/vdb1
resize2fs 1.43.5 (04-Aug-2017)
Resizing the filesystem on /dev/vdb1 to 805305856 (4k) blocks.
The filesystem on /dev/vdb1 is now 805305856 blocks long.
[root@ecshost ~]# mount /dev/vdb1 /mnt
For an XFS file system, run the following commands in sequence to remount the partition and then resize the XFS file system.
Note The new version xfs_growfs identifies the device to be resized based on the mount point. Example: xfs_growfs /mnt. You can run the xfs_growfs --help command to check how to use xfs_growfs of different versions.
[root@ecshost ~]# mount /dev/vdb1 /mnt/
[root@ecshost ~]# xfs_growfs /mnt
Option 4: Add and format GPT partitions
If new disk space is added to create more GPT partitions, perform the following operations to resize the file system of the instance: In this example, a data disk of 32 TiB is used. The existing partition /dev/vdb1 has a 4.8 TiB capacity. The /dev/vdb2 partition is to be created.

Run the fdisk command to view information about the existing partition.
[root@ecshost ~]# fdisk -l
Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b1b45

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83875364    41936658+  83  Linux
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/vdb: 35184.4 GB, 35184372088832 bytes, 68719476736 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: BCE92401-F427-45CC-8B0D-B30EDF279C2F

#         Start          End    Size  Type            Name
 1         2048  10307921919    4.8T  Microsoft basic mnt                    
Use the parted tool to create a new partition and allocate capacity for it.
Run the parted /dev/vdb command to start the Parted tool.
Run the print free command to view the disk capacity to be allocated. Record the start and end sectors and capacity of the existing partition.
In this example, the start sector number of /dev/vdb1 is 1,049 KB, the end sector number is 5,278 GB, and the capacity is 5,278 GiB.
(parted) print free
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 35.2TB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
        17.4kB  1049kB  1031kB  Free Space
 1      1049kB  5278GB  5278GB  ext4         mnt
        5278GB  35.2TB  29.9TB  Free Space                    
Run the mkpart <the partition name> <the start sector number> <the percentage of allocated capacity> command.
In this example, the /dev/vdb2 partition named test is created. The start sector number of the new partition is the end sector number of the existing partition. The new capacity is allocated to the new partition.

Run the print command to check whether the capacity (Size) of the partition is changed.
(parted) mkpart test 5278GB 100%
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 35.2TB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
 1      1049kB  5278GB  5278GB  ext4         mnt
 2      5278GB  35.2TB  29.9TB               test                    
Run the quit command to exit the parted tool.
Create a file system for the new partition.
To create an ext4 file system, run the mkfs.ext4 /dev/vdb2 command.
To create an ext3 file system, run the mkfs.ext3 /dev/vdb2 command.
To create an XFS file system, run the mkfs.xfs -f /dev/vdb2 command.
To create a btrfs file system, run the mkfs.btrfs /dev/vdb2 command.
In this example, an XFS file system is created.
[root@ecshost ~]# mkfs -t xfs /dev/vdb2
meta-data=/dev/vdb2              isize=512    agcount=28, agsize=268435455 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=7301444096, imaxpct=5
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=521728, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0               
Run the fdisk -l command to view the change of partition capacity.
[root@ecshost ~]# fdisk -l
Disk /dev/vda: 42.9 GB, 42949672960 bytes, 83886080 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x000b1b45

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    83875364    41936658+  83  Linux
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/vdb: 35184.4 GB, 35184372088832 bytes, 68719476736 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: gpt
Disk identifier: BCE92401-F427-45CC-8B0D-B30EDF279C2F

#         Start          End    Size  Type            Name
 1         2048  10307921919    4.8T  Microsoft basic mnt
 2  10307921920  68719474687   27.2T  Microsoft basic test  
Run the blkid command to view the file system types.
[root@ecshost ~]# blkid
/dev/vda1: UUID="ed95c595-4813-480e-****-85b1347842e8" TYPE="ext4"
/dev/vdb1: UUID="21e91bbc-7bca-4c08-****-88d5b3a2303d" TYPE="ext4" PARTLABEL="mnt" PARTUUID="576235e0-5e04-4b76-****-741cbc7e98cb"
/dev/vdb2: UUID="a7dcde59-8f0f-4193-****-362a27192fb1" TYPE="xfs" PARTLABEL="test" PARTUUID="464a9fa9-3933-4365-****-c42de62d2864"  
Mount the new partition.
[root@ecshost ~]# mount /dev/vdb2 /mnt
Option 5: Resize the file system of a raw data disk
If a raw data disk contains a file system but no partitions, perform the following operations to directly resize the file system:

View the mount path of the data disk and unmount the partition based on the returned file path.
[root@ecshost ~]# mount | grep "/dev/vdb"
/dev/vdb on /mnt type ext4 (rw,relatime,data=ordered)
[root@ecshost ~]# umount /dev/vdb
[root@ecshost ~]# mount | grep "/dev/vdb"
Run different commands based on the file system type.
For an ext* file system, run the resize2fs command as the root user to resize the file system. Example:
resize2fs /dev/vdb
For an XFS file system, run the xfs_growfs command as the root user to resize the file system.
Note The new version xfs_growfs identifies the device to be resized based on the mount point. Example: xfs_growfs /mnt. You can run the xfs_growfs --help command to check how to use xfs_growfs of different versions.
xfs_growfs of the new version
xfs_growfs /mnt
xfs_growfs that is not updated
xfs_growfs /dev/vdb
Mount the disk to the mount point.
mount /dev/vdb /mnt
Run the df -h command to view the result of data disk resizing.
The file system has a larger capacity, which indicates that the file system is resized.
[root@ecshost ~]# df -h
Filesystem Size Used Avail Use% Mounted on
/dev/vda1 40G 1.6G 36G 5% /
devtmpfs 3.9G 0 3.9G 0% /dev
tmpfs 3.9G 0 3.9G 0% /dev/shm
tmpfs 3.9G 460K 3.9G 1% /run
tmpfs 3.9G 0 3.9G 0% /sys/fs/cgroup
/dev/vdb 98G 37G 61G 37% /mnt
tmpfs 783M 0 783M 0% /run/user/0
