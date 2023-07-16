* same disk size and same speed no required unlike raid san,nas
* software defined storage
* unified distributed storage
* used in cloud devops
* CEPHFS - file system
* CEPFX - provide both authentication and authorization
* Ceph MON - provides opinion about data that what is right
* AT 85% CEPH cluster gives warning , and at 90% cluster only become read only
* coz we require space for metadata
* eraser coding
* default chunk/block size = 4MB
### CEPH Architecture

![Screenshot 2023-07-16 102755](https://github.com/shubnimkar/Storage-Backup-Management/assets/46809421/9dc67446-b3b4-4ab3-8f2f-f047fa4b8882)



#### RADOS software 
* RADOS (Reliable Autonomic Distributed Object Store)
* OSD - Object storage daemon (where main data is stored)
* MON- Monitor
* Meta- Keeps record of where and what is written on storage(METADATA)
* CRUSH -  CRUSH algorithm computes storage locations in order to determine how to store and retrieve data
* LIBRADOS - librados provides low-level access to the RADOS service.
