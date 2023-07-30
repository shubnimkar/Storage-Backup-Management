<h1 align="center">RAID 0</h1>

### To implement this follow config below

* 1 BOOT PARTITION
* ADD 2 HDD OF ANY SIZE

<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/a20e5923-23d9-432d-88ff-710ff4b153b8>
<p>


### To Check if our harddisk are mounted or not 

    lsblk
    
<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/e5b4be0e-fa13-4e98-9bdc-af2f28cf81be>
</p>


### Install mdadm to create raid

    yum install mdadm -y

    
### Create Raid-0 with command below

    mdadm  --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdb /dev/sdc

<p laign="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/f3ba37f5-843f-4ecd-b68b-f36f673451a0>
<p>

### To check if it's created or not

    cat /proc/mdstat
<p align="center">    
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/d7ae585a-ebad-4598-807a-89885d7d7e8c>
<p>

### You can examine drives as well

    mdadm --examine /dev/sdb
<p align="center">    
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/ad283859-7ed9-4dab-b4a8-0ec9b6cfea6a>
<p>
### List Blocks

    lsblk
<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/35a97fbb-20bf-4a26-af5a-444352272cab>
<p>
### Create new partition
    
    fdisk /dev/md0
<p align="center">    
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/55f855ba-e4d0-42ad-a57f-81f81f113185>
<p>

### Make filesystem

    mkfs.ext4 /dev/md0p1
<p align="center">    
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/db26b6fe-1ceb-4c3d-a4bd-5bd8f0d4ab0a>
<p>
### Mounting the created partition 

    mkdir /mnt/raid0
    mount /dev/md0p1 /mnt/raid0

### Check for partition folder if it's mounted or not

    cd /mnt/raid0
    ll
![13](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/f90d8d30-9db0-478a-9281-54cff7cbb48f)


### Check for partitions and used space by command below

    df- h
<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/8c974fba-0c0e-42f9-8c5d-02636e5f6303>
<p>

### Creating a file so that we can see the space usage

    dd if=/dev/zero of=500mb.file bs=1024 count=502400
 <p align="center">   
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/e20cf315-0568-4eb1-9ad7-8a1b33fd004f>
<p>

### And now we can see changes

    df -h
<p align="center">    
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/1dfc0a02-5cdf-4080-adc2-95563fd65cac>
<p>

### To unmount the raid partition

    umount /mnt/raid0
    
### check for partitions ,it'll be missing from list 

    df -h 

<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/05b175d3-6413-4a13-b7bb-bdb1db2c67d9>
<p>

### Stop the raid 0 partition

    mdadm --stop /dev/md0

<p align="center">
<img src=https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/1b9348d6-69a2-4189-bf31-88d4f8a955e6>
<p>
