Prerequisites:
* 1 VM having TrueNAS having HDD's
* 1 VM of centos as client

## On Linux Client:

To install ISCSI initiator:
    
    yum install iscsi-initiator-utils
    

Now in TrueNAS webUI

#### TARGET GLOBAL CONFIG: 

![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_53_27](https://github.com/shubnimkar/SBM/assets/46809421/d9e403f9-a516-4d1d-a9fe-7e5a09d9bbc8)

#### PORTALS

![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_53_36](https://github.com/shubnimkar/SBM/assets/46809421/f6948e93-5a95-4cc2-813e-5c939ff2cacd)

#### INITIATORS

Need to add name of initiators so that linux volume will go to linux and windows volume will go to windows

    On client
    cd /etc/iscsi/
    cat initiatorname.iscsi
    InitiatorName=iqn.1994-05.com.redhat:4a3bb8f3a60

##### LINUX
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_54_17](https://github.com/shubnimkar/SBM/assets/46809421/5daaad04-017a-48d8-b3ee-c4a2f1a60db9)

##### WINDOWS
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_54_28](https://github.com/shubnimkar/SBM/assets/46809421/b6f5b2e9-93d2-48f8-8d34-7d22897e1027)

##### FINAL SCREEN
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_54_34](https://github.com/shubnimkar/SBM/assets/46809421/b926114c-6b55-4295-8118-3d9fb68e3aeb)

#### AUTHORIZED ACCESS

![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_55_03](https://github.com/shubnimkar/SBM/assets/46809421/ea0b351b-0a35-4954-bcca-600940638c08)

#### TARGETS

##### LINUX
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_55_19](https://github.com/shubnimkar/SBM/assets/46809421/97a2cee8-1b5e-4823-8196-1a91ca3e2cf3)

##### WINDOWS
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_55_29](https://github.com/shubnimkar/SBM/assets/46809421/9e06c4e4-0046-4183-a892-36bf9d7ce2fe)

##### FINAL SCREEN
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_55_34](https://github.com/shubnimkar/SBM/assets/46809421/5abc095d-7d14-4122-9236-5eab78606d00)

#### EXTENT

##### LINUX
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_56_03](https://github.com/shubnimkar/SBM/assets/46809421/d87fc249-f98f-4d08-87f3-2ea80cbaa245)

##### WINDOWS
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_56_10](https://github.com/shubnimkar/SBM/assets/46809421/1c6a4318-f90b-4d22-ad3b-96f803991247)

##### FINAL SCREEN
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_56_36](https://github.com/shubnimkar/SBM/assets/46809421/5ce2d64d-f057-4a71-ba2d-604b9bbe2bac)

#### ASSOCIATED TARGETS

##### WINDOWS
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_57_04](https://github.com/shubnimkar/SBM/assets/46809421/0813fe9a-b040-4bd9-a9bc-1894bd923f95)

##### LINUX
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_57_14](https://github.com/shubnimkar/SBM/assets/46809421/f6390a3c-35a4-406b-8a92-4eb75c8d6186)

##### FINAL SCREEN
![TrueNAS - 192 168 100 176 - Google Chrome 14-07-2023 16_57_19](https://github.com/shubnimkar/SBM/assets/46809421/1a78658a-a780-4fe9-af9a-c78dc89f09e3)

### To create and format disk

    fdisk /dev/sdc
    
    Command (m for help): n
    Partition type:
       p   primary (0 primary, 0 extended, 4 free)
       e   extended
    Select (default p): p
    Partition number (1-4, default 1): 1
    First sector (16384-4194304, default 16384):
    Using default value 16384
    Last sector, +sectors or +size{K,M,G} (16384-4194304, default 4194304):
    Using default value 4194304
    Partition 1 of type Linux and of size 2 GiB is set
    
    Command (m for help): w
    The partition table has been altered!
    
    Calling ioctl() to re-read partition table.
    Syncing disks.

## for log in
    iscsiadm -m discovery -t st -p 192.168.100.176 --login

Now to check if volume is created

    iscsiadm -m discovery -t st -p 192.168.100.176
    
    192.168.100.176:3260,-1 iqn.2005-10.org.freenas.ctl:linuxvm

lsblk

        [root@localhost iscsi]# lsblk
        NAME            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
        sda               8:0    0   20G  0 disk
        ├─sda1            8:1    0    1G  0 part /boot
        └─sda2            8:2    0   19G  0 part
          ├─centos-root 253:0    0   17G  0 lvm  /
          └─centos-swap 253:1    0    2G  0 lvm  [SWAP]
        sdb               8:16   0    5G  0 disk
        ├─sdb1            8:17   0   16M  0 part
        └─sdb2            8:18   0    5G  0 part
        sdc               8:32   0    2G  0 disk
        sr0              11:0    1  973M  0 rom



   

## for logout
iscsiadm -m node -U all

fdisk -l
lsblk

iscsiadm -m session


      
