# Prerequisites: 

1. Add new hard disk drive (HDD): Physically connect the new hard disk drive to the servers. 

2. Configure /etc/hosts file on all servers & client

3. Stop the firewall service: Run the following command to stop the firewall service on each server & client

        systemctl stop firewalld
        systemctl disable firewalld 
        systemctl status firewalld


# On servers

1. `lsblk` to get name of disk partition on all servers
   
2. Now format and create volume on all servers
   
       fdisk /dev/sdb
       press n > p > 1 > enter > enter and w

3. Create file system on all servers
   
       mkfs.xfs /dev/sdb1
   
![1](https://github.com/shubnimkar/Storage-Backup-Management/assets/46809421/19cbc2c0-1b20-47c5-b15a-0e2ae0f16e96)

4. Create a directory for mounting the new drive on all servers

        mkdir /mnt/disk1

4. Now mount 

       mount /dev/sdb1 /mnt/disk1
  
   
6. Install the required packages: Install wget, centos-release-gluster, epel-release, and glusterfs-server packages on all servers
   
       yum install wget centos-release-gluster epel-release glusterfs-server -y
   

7. Start and enable Gluster daemon: The following commands will start the gluster daemon and set it to start automatically at boot:
   
           systemctl start glusterd
           systemctl enable glusterd
   
   
8. Add peers: Add other servers as peers to the GlusterFS cluster from server 1
   
           gluster peer probe gfs2.hpcsa.in
           gluster peer probe gfs3.hpcsa.in
   
   
9. Check the status of the GlusterFS cluster: The following commands will display the status of the peers and the list of servers in the pool:
   
           gluster peer status
           gluster pool list
   
   
10. Create a new directory for the Gluster volume on each server: 
   
           mkdir /mnt/disk1/diskvol
   
   
11. Create and start a Gluster volume: This command creates a Gluster volume named 'gdisk1' with a replication factor 
    of 3. It then starts the volume:
   
           gluster volume create gdisk1 replica 3 gfs1.hpcsa.in:/mnt/disk1/diskvol/gdisk1  gfs2.hpcsa.in:/mnt/disk1/diskvol/gdisk1 gfs3.hpcsa.in:/mnt/disk1/diskvol/gdisk1
   
           gluster volume start gdisk1
   
   
12. Display information about the Gluster volume: This command displays information about the 'gdisk1' volume:
   
           gluster volume info gdisk1
   

# On Client

1. On the client, install glusterfs-fuse package: This package allows GlusterFS volumes to be mounted via fuse:
   
           yum install glusterfs-fuse
   
   
2. Create a directory for mounting the Gluster volume:
   
           mkdir /mnt/gdrive
   
   
3. Mount the Gluster volume on the client: This command mounts the 'gdisk1' volume to the newly created directory:
   
           mount -t glusterfs gfs1:/gdisk1 /mnt/gdrive
   
   Now, the Gluster volume 'gdisk1' is available on the client at the '/mnt/gdrive' directory.

4. Now to check the volume

           df -h
![3](https://github.com/shubnimkar/Storage-Backup-Management/assets/46809421/84af6b98-bc7a-4ed7-9f59-dfd5deb30d23)

TO create file
dd if=/dev/zero of=file.data bs=1024 count=1236778675
can check using df -h
and in folders too

### Distributed

        [root@gfs1 ~]# gluster volume create gdisk11 gfs1.hpcsa.in:/mnt/disk1/diskvol/gdisk11  gfs2.hpcsa.in:/mnt/disk1/diskvol/gdisk11 gfs3.hpcsa.in:/mnt/disk1/diskvol/gdisk11
        volume create: gdisk11: success: please start the volume to access data
        [root@gfs1 ~]# df -h
        Filesystem               Size  Used Avail Use% Mounted on
        devtmpfs                 475M     0  475M   0% /dev
        tmpfs                    487M     0  487M   0% /dev/shm
        tmpfs                    487M  7.7M  479M   2% /run
        tmpfs                    487M     0  487M   0% /sys/fs/cgroup
        /dev/mapper/centos-root   17G  1.5G   16G   9% /
        /dev/sda1               1014M  138M  877M  14% /boot
        tmpfs                     98M     0   98M   0% /run/user/0
        /dev/sdb1                 30G   34M   30G   1% /mnt/disk1
        [root@gfs1 ~]# gluster volume start gdisk11
        volume start: gdisk11: success

### Dispersed
        [root@gfs1 gdisk11]# gluster volume create gdisk12 disperse gfs1.hpcsa.in:/mnt/disk1/diskvol/gdisk12  gfs2.hpcsa.in:/mnt/disk1/diskvol/gdisk12 gfs3.hpcsa.in:/mnt/disk1/diskvol/gdisk12
        volume create: gdisk12: success: please start the volume to access data
