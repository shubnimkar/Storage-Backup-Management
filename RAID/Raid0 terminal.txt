# RAID 0


1 base machine rocky/centos NAT 
1 boot machine centos 
2 hdd but of same size

add 2 hdd in boot machine
![Capture](https://github.com/shubnimkar/SBM/assets/46809421/906f4418-55a3-4b79-9ac4-bcd72aa64825)


login as: root
root@192.168.100.170's password:
    ┌──────────────────────────────────────────────────────────────────────┐
    │                 • MobaXterm Personal Edition v23.1 •                 │
    │               (SSH client, X server and network tools)               │
    │                                                                      │
    │ ⮞ SSH session to root@192.168.100.170                                │
    │   • Direct SSH      :  ✓                                             │
    │   • SSH compression :  ✓                                             │
    │   • SSH-browser     :  ✓                                             │
    │   • X11-forwarding  :  ✗  (disabled or not supported by server)      │
    │                                                                      │
    │ ⮞ For more info, ctrl+click on help or visit our website.            │
    └──────────────────────────────────────────────────────────────────────┘

Last login: Sat Jul  8 20:18:33 2023

[root@localhost ~]#
[root@localhost ~]# lsblk
NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda               8:0    0   20G  0 disk
├─sda1            8:1    0    1G  0 part /boot
└─sda2            8:2    0   19G  0 part
  ├─centos-root 253:0    0   17G  0 lvm  /
  └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
sdb               8:16   0    1G  0 disk
sdc               8:32   0    1G  0 disk
sr0              11:0    1  973M  0 rom
[root@localhost ~]# yum clean all
Loaded plugins: fastestmirror
Cleaning repos: base extras updates
[root@localhost ~]# mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
-bash: mdadm: command not found
[root@localhost ~]# yum install mdadm -y
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: centos.mirror.net.in
 * extras: centos.mirror.net.in
 * updates: centos.mirror.net.in
base                                                                                                      | 3.6 kB  00:00:00
extras                                                                                                    | 2.9 kB  00:00:00
updates                                                                                                   | 2.9 kB  00:00:00
(1/4): base/7/x86_64/group_gz                                                                             | 153 kB  00:00:00
(2/4): extras/7/x86_64/primary_db                                                                         | 250 kB  00:00:00
(3/4): updates/7/x86_64/primary_db                                                                        |  22 MB  00:00:00
(4/4): base/7/x86_64/primary_db                                                                           | 6.1 MB  00:00:02
Resolving Dependencies
--> Running transaction check
---> Package mdadm.x86_64 0:4.1-9.el7_9 will be installed
--> Processing Dependency: libreport-filesystem for package: mdadm-4.1-9.el7_9.x86_64
--> Running transaction check
---> Package libreport-filesystem.x86_64 0:2.1.11-53.el7.centos will be installed
--> Finished Dependency Resolution

Dependencies Resolved

=================================================================================================================================
 Package                              Arch                   Version                               Repository               Size
=================================================================================================================================
Installing:
 mdadm                                x86_64                 4.1-9.el7_9                           updates                 439 k
Installing for dependencies:
 libreport-filesystem                 x86_64                 2.1.11-53.el7.centos                  base                     41 k

Transaction Summary
=================================================================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 480 k
Installed size: 1.0 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/base/packages/libreport-filesystem-2.1.11-53.el7.centos.x86_64.rpm: Header V3 RSA/SHA256 Signature, key ID f4a80eb5: NOKEY
Public key for libreport-filesystem-2.1.11-53.el7.centos.x86_64.rpm is not installed
(1/2): libreport-filesystem-2.1.11-53.el7.centos.x86_64.rpm                                               |  41 kB  00:00:00
Public key for mdadm-4.1-9.el7_9.x86_64.rpm is not installed
(2/2): mdadm-4.1-9.el7_9.x86_64.rpm                                                                       | 439 kB  00:00:00
---------------------------------------------------------------------------------------------------------------------------------
Total                                                                                            2.8 MB/s | 480 kB  00:00:00
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Importing GPG key 0xF4A80EB5:
 Userid     : "CentOS-7 Key (CentOS 7 Official Signing Key) <security@centos.org>"
 Fingerprint: 6341 ab27 53d7 8a78 a7c2 7bb1 24c6 a8a7 f4a8 0eb5
 Package    : centos-release-7-9.2009.0.el7.centos.x86_64 (@anaconda)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libreport-filesystem-2.1.11-53.el7.centos.x86_64                                                              1/2
  Installing : mdadm-4.1-9.el7_9.x86_64                                                                                      2/2
  Verifying  : mdadm-4.1-9.el7_9.x86_64                                                                                      1/2
  Verifying  : libreport-filesystem-2.1.11-53.el7.centos.x86_64                                                              2/2

Installed:
  mdadm.x86_64 0:4.1-9.el7_9

Dependency Installed:
  libreport-filesystem.x86_64 0:2.1.11-53.el7.centos

Complete!
[root@localhost ~]# mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc
mdadm: chunk size defaults to 512K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
[root@localhost ~]# cat /proc/mdstat
Personalities : [raid0]
md0 : active raid0 sdc[1] sdb[0]
      2093056 blocks super 1.2 512k chunks

unused devices: <none>
[root@localhost ~]# mdadm  --examine /dev/sdb
/dev/sdb:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 05cbbd18:3f0d68d1:98b0fc01:6f6b7f26
           Name : localhost.localdomain:0  (local to host localhost.localdomain)
  Creation Time : Sat Jul  8 20:21:09 2023
     Raid Level : raid0
   Raid Devices : 2

 Avail Dev Size : 2093056 sectors (1022.00 MiB 1071.64 MB)
    Data Offset : 4096 sectors
   Super Offset : 8 sectors
   Unused Space : before=4016 sectors, after=0 sectors
          State : clean
    Device UUID : c383b4b4:dcf3fb33:bb517561:1a73562c

    Update Time : Sat Jul  8 20:21:09 2023
  Bad Block Log : 512 entries available at offset 8 sectors
       Checksum : e02839a - correct
         Events : 0

     Chunk Size : 512K

   Device Role : Active device 0
   Array State : AA ('A' == active, '.' == missing, 'R' == replacing)
[root@localhost ~]# mdadm  --examine /dev/sdc
/dev/sdc:
          Magic : a92b4efc
        Version : 1.2
    Feature Map : 0x0
     Array UUID : 05cbbd18:3f0d68d1:98b0fc01:6f6b7f26
           Name : localhost.localdomain:0  (local to host localhost.localdomain)
  Creation Time : Sat Jul  8 20:21:09 2023
     Raid Level : raid0
   Raid Devices : 2

 Avail Dev Size : 2093056 sectors (1022.00 MiB 1071.64 MB)
    Data Offset : 4096 sectors
   Super Offset : 8 sectors
   Unused Space : before=4016 sectors, after=0 sectors
          State : clean
    Device UUID : 215a07bb:6a0622e0:ca1f0995:12f46654

    Update Time : Sat Jul  8 20:21:09 2023
  Bad Block Log : 512 entries available at offset 8 sectors
       Checksum : 1c1fbb8f - correct
         Events : 0

     Chunk Size : 512K

   Device Role : Active device 1
   Array State : AA ('A' == active, '.' == missing, 'R' == replacing)
[root@localhost ~]# mdadm  --detail /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Sat Jul  8 20:21:09 2023
        Raid Level : raid0
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
      Raid Devices : 2
     Total Devices : 2
       Persistence : Superblock is persistent

       Update Time : Sat Jul  8 20:21:09 2023
             State : clean
    Active Devices : 2
   Working Devices : 2
    Failed Devices : 0
     Spare Devices : 0

        Chunk Size : 512K

Consistency Policy : none

              Name : localhost.localdomain:0  (local to host localhost.localdomain)
              UUID : 05cbbd18:3f0d68d1:98b0fc01:6f6b7f26
            Events : 0

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync   /dev/sdb
       1       8       32        1      active sync   /dev/sdc

[root@localhost ~]# fdisk /dev/md0
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x056af8ca.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-4186111, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4186111, default 4186111):
Using default value 4186111
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

[root@localhost ~]# fdisk /dev/md0
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x056af8ca.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-4186111, default 2048):
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-4186111, default 4186111):
Using default value 4186111
Partition 1 of type Linux and of size 2 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.

[root@localhost ~]# mkfs.ext4 /dev/md0p1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=128 blocks, Stripe width=256 blocks
130816 inodes, 523008 blocks
26150 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=536870912
16 block groups
32768 blocks per group, 32768 fragments per group
8176 inodes per group
Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912

Allocating group tables: done
Writing inode tables: done
Creating journal (8192 blocks): done
Writing superblocks and filesystem accounting information: done

[root@localhost ~]# mkdir /mnt/raid0
[root@localhost ~]# mount /dev/md0p1 /mnt/raid0
[root@localhost ~]# cd /mnt/raid0/
[root@localhost raid0]# ls
lost+found
[root@localhost raid0]# touch test
[root@localhost raid0]# ls
lost+found  test

[root@localhost raid0]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 475M     0  475M   0% /dev
tmpfs                    487M     0  487M   0% /dev/shm
tmpfs                    487M  7.7M  479M   2% /run
tmpfs                    487M     0  487M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.5G   16G   9% /
/dev/sda1               1014M  138M  877M  14% /boot
tmpfs                     98M     0   98M   0% /run/user/0
/dev/md0p1               2.0G  6.0M  1.9G   1% /mnt/raid0
[root@localhost raid0]# dd if=/dev/zero of=500mb.file bs=1024 count=502400
502400+0 records in
502400+0 records out
514457600 bytes (514 MB) copied, 2.03775 s, 252 MB/s
[root@localhost raid0]# df -h
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 475M     0  475M   0% /dev
tmpfs                    487M     0  487M   0% /dev/shm
tmpfs                    487M  7.7M  479M   2% /run
tmpfs                    487M     0  487M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.5G   16G   9% /
/dev/sda1               1014M  138M  877M  14% /boot
tmpfs                     98M     0   98M   0% /run/user/0
/dev/md0p1               2.0G  497M  1.4G  27% /mnt/raid0
[root@localhost raid0]#
















[root@localhost ~]# ls -la /mnt/raid0
total 20
drwxr-xr-x. 3 root root  4096 Jul  8 20:34 .
drwxr-xr-x. 3 root root    19 Jul  8 20:34 ..
drwx------. 2 root root 16384 Jul  8 20:34 lost+found
