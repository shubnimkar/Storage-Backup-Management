# Logical Volume Manager

To perform the practical add 2 HDD in VM


1. Identify the new hard disks:

This is achieved using the `lsblk` command. `lsblk` lists information about all available or the specified block devices. It reads the sysfs filesystem and udev db to gather information.


    [root@nfs-client ~]# lsblk

![Capture](https://github.com/shubnimkar/SBM/assets/46809421/dc50a0c6-a0df-44c7-a066-630b56dc4bb9)

Here, `sdb` and `sdc` are the new hard disks that you want to add.

2. Create physical volumes:

This is done with `pvcreate`. This initializes physical volume(s) for later use by the Logical Volume Manager (LVM). Each physical volume can be a disk partition, whole disk, meta-device, or loopback file.


    pvcreate /dev/sdb /dev/sdc
   
![2](https://github.com/shubnimkar/SBM/assets/46809421/3cfc706e-b393-42a4-bec7-0a5bd3cc79c0)

pvdisplay command for view volumes

    pvdisplay

![3](https://github.com/shubnimkar/SBM/assets/46809421/9d0682cf-aaef-4637-898b-79f1eee0ae84)

3. Create a volume group:

`vgcreate` creates a new volume group named DEMO on physical volumes /dev/sdb and /dev/sdc.


    vgcreate HPCSA /dev/sdb /dev/sdc
![4](https://github.com/shubnimkar/SBM/assets/46809421/928181fe-fcc9-478a-956c-d7abd18e5361)

vgdisplay to view

    vgdisplay
![5](https://github.com/shubnimkar/SBM/assets/46809421/21abdcf3-bd05-4816-b3db-d0284ef5a410)

4. Create a logical volume:

`lvcreate` creates a logical volume in a volume group. In this case, it's creating a logical volume named `demo_lab` with a size of 1G in the volume group `DEMO`.


    lvcreate -n demo_lab --size 1G DEMO

![6](https://github.com/shubnimkar/SBM/assets/46809421/3e187407-aeca-436b-9951-069b7ae0f535)


lvdisplay to view

lvdisplay

![7](https://github.com/shubnimkar/SBM/assets/46809421/fdacf9c2-0d01-478f-bb54-bd9724aadef4)

5. Partition the new logical volume:

`fdisk` is a dialogue-driven command-line utility that creates and manipulates partition tables and partitions on a hard disk.


    fdisk /dev/mapper/DEMO-demo_lab

Here you're asked to press `n` to create a new partition.
then press p for primary
then mention size or press enter , enter
then again will ask for option press w to write and exit

    Error:
    WARNING: Re-reading the partition table failed with error 22: Invalid argument.
    The kernel still uses the old table. The new table will be used at
    the next reboot or after you run partprobe(8) or kpartx(8)
    Syncing disks.
    
    [root@localhost ~]# partprobe /dev/HPCSA/hpcsa_lab

![8](https://github.com/shubnimkar/SBM/assets/46809421/a5ea24e7-cfb4-4066-8764-b711acdf7bc1)


6. Create a filesystem:

`mkfs.ext4` is used to create an ext4 filesystem on the partition.


    mkfs.ext4 /dev/mapper/DEMO-demo_lab

![12](https://github.com/shubnimkar/SBM/assets/46809421/5dd952b7-b4bd-4884-ae86-5689487fd3fb)


7. Create a mount point and mount the logical volume:

Create a directory that will serve as the mount point, and then use the `mount` command to mount the logical volume.


    mkdir our-demo
    mount /dev/mapper/DEMO-demo_lab our-demo

![13](https://github.com/shubnimkar/SBM/assets/46809421/5077bec8-731d-422b-a54c-d0947d17bef6)


8. Extend the logical volume:

`lvextend` allows you to extend the size of a logical volume. Here, you're extending the logical volume `hpcsa_lab` by an additional 2G.


    lvextend -L +2G /dev/mapper/DEMO-demo_lab

![14](https://github.com/shubnimkar/SBM/assets/46809421/f85e74d8-6888-436f-8e26-698aae98f02d)


9. Resize the filesystem:

`resizefs` resizes the filesystem on the logical volume to use all of the available space.


    resize2fs /dev/mapper/DEMO-demo_lab

![15](https://github.com/shubnimkar/SBM/assets/46809421/3a56be1a-a19d-453a-a4c1-5b1236c43c31)

10. Create a snapshot:

`lvcreate` with `-s` creates a snapshot logical volume, which is a read-only copy of another logical volume.


    lvcreate -L 1GB -s -n demo_lab_snap /dev/mapper/DEMO-demo_lab

![16](https://github.com/shubnimkar/SBM/assets/46809421/36524f29-73f9-425b-b52c-12b49013c006)


11. Merge the snapshot:

`lvconvert` with `--merge` will merge the snapshot back into its origin volume. If both origin and snapshot volume are not open the merge will start immediately, otherwise, it will be delayed until the origin volume becomes inactive.

![17](https://github.com/shubnimkar/SBM/assets/46809421/1bfe9800-8c3e-436b-be20-6a71ed4ec444)


    lvconvert --merge /dev/mapper/DEMO-demo_lab_snap

![18](https://github.com/shubnimkar/SBM/assets/46809421/13eb2ed8-efef-4ce8-a008-daa920b8e039)

Remember to add the disks to `/etc/fstab` if you want them to be mounted automatically at system startup.
 12. Mirror
 it creates mirror of data in all physical  drives
 
    lvcreate -L 2GB -m1 -n testmirror DEMO
![19](https://github.com/shubnimkar/SBM/assets/46809421/3241fcd3-218f-4715-89a2-739eac740443)

  Logical volume "testmirror_lv" created.

13. Remove a logical volume

The command lvremove can be used to remove logical volumes. We should make sure a logical volume does not have any valuable data stored on it before we attempt to remove it. Moreover, we should make sure the volume is not mounted.

    lvremove /dev/mapper/DEMO-demo_lab

![20](https://github.com/shubnimkar/SBM/assets/46809421/e7d2db0c-8f84-4df6-a855-29f8b95226c3)

Now to remove the DEMO volume

    lvremove /dev/mapper/DEMO

![21](https://github.com/shubnimkar/SBM/assets/46809421/eb671165-1345-4bdb-abe2-0c109af1ce0e)
