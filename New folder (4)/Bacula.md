![s](https://github.com/shubnimkar/Storage-and-Backup-Management/assets/46809421/7350eec2-3604-4cd2-b4d7-46510206dd75)


### Install bacula-director bacula-storage bacula-console bacula-client mariadb-server nano httpd
  
    yum install -y bacula-director bacula-storage bacula-console bacula-client mariadb-server nano httpd

### Start,enable & check status of Mariadb

    systemctl enable mariadb && systemctl start mariadb && systemctl status mariadb

### Create files

    /usr/libexec/bacula/grant_mysql_privileges
    /usr/libexec/bacula/create_mysql_database -u root
    /usr/libexec/bacula/make_mysql_tables -u bacula

### mysql installation

    mysql_secure_installation

### select for mysql

    alternatives --config libbaccats.so
    select mysql

### Disable SELINUX and firewalld

    setenforce 0
    systemctl stop firewalld

### make dir for restore and change permissions

    mkdir -p /bacula/backup /bacula/restore
    
    chown -R bacula:bacula /bacula
    chmod -R 700 /bacula

### Add entry in host file

    nano /etc/hosts
    192.168.100.132 bacula.hpcsa.com

nano /etc/bacula/bacula-dir.conf

    [root@localhost ~]# cat /etc/bacula/bacula-dir.conf
    #
    # Default Bacula Director Configuration file
    #
    #  The only thing that MUST be changed is to add one or more
    #   file or directory names in the Include directive of the
    #   FileSet resource.
    #
    #  For Bacula release 5.2.13 (19 February 2013) -- redhat (Core)
    #
    #  You might also want to change the default email address
    #   from root to your address.  See the "mail" and "operator"
    #   directives in the Messages resource.
    #
    
    Director {                            # define myself
      Name = bacula-dir
      DIRport = 9101                # where we listen for UA connections
      QueryFile = "/etc/bacula/query.sql"
      WorkingDirectory = "/var/spool/bacula"
      PidDirectory = "/var/run"
      Maximum Concurrent Jobs = 1
      Password = "root"         # Console password
      Messages = Daemon
      DirAddress= 192.168.100.132
    }
    
    JobDefs {
      Name = "DefaultJob"
      Type = Backup
      Level = Incremental
      Client = bacula-fd
      FileSet = "Full Set"
      Schedule = "WeeklyCycle"
      Storage = File
      Messages = Standard
      Pool = File
      Priority = 10
      Write Bootstrap = "/var/spool/bacula/%c.bsr"
    }
    
    
    #
    # Define the main nightly save backup job
    #   By default, this job will back up to disk in /tmp
    Job {
      Name = "BackupClient1"
      JobDefs = "DefaultJob"
    }
    
    #Job {
    #  Name = "BackupClient2"
    #  Client = bacula2-fd
    #  JobDefs = "DefaultJob"
    #}
    
    # Backup the catalog database (after the nightly save)
    Job {
      Name = "BackupCatalog"
      JobDefs = "DefaultJob"
      Level = Full
      FileSet="Catalog"
      Schedule = "WeeklyCycleAfterBackup"
      # This creates an ASCII copy of the catalog
      # Arguments to make_catalog_backup.pl are:
      #  make_catalog_backup.pl <catalog-name>
      RunBeforeJob = "/usr/libexec/bacula/make_catalog_backup.pl MyCatalog"
      # This deletes the copy of the catalog
      RunAfterJob  = "/usr/libexec/bacula/delete_catalog_backup"
      Write Bootstrap = "/var/spool/bacula/%n.bsr"
      Priority = 11                   # run after main backup
    }
    
    #
    # Standard Restore template, to be changed by Console program
    #  Only one such job is needed for all Jobs/Clients/Storage ...
    #
    Job {
      Name = "RestoreFiles"
      Type = Restore
      Client=bacula-fd
      FileSet="Full Set"
      Storage = File
      Pool = Default
      Messages = Standard
      Where = /tmp/bacula-restores
    }
    
    
    # List of files to be backed up
    FileSet {
      Name = "Full Set"
      Include {
        Options {
          signature = MD5
          compression = GZIP
        }
    #
    #  Put your list of files here, preceded by 'File =', one per line
    #    or include an external list with:
    #
    #    File = <file-name
    #
    #  Note: / backs up everything on the root partition.
    #    if you have other partitions such as /usr or /home
    #    you will probably want to add them too.
    #
    #  By default this is defined to point to the Bacula binary
    #    directory to give a reasonable FileSet to backup to
    #    disk storage during initial testing.
    #
        File = /var/www/html #folder needed to backup
      }
    
    #
    # If you backup the root directory, the following two excluded
    #   files can be useful
    #
      Exclude {
        File = /var/lib/bacula
        File = /proc
        File = /tmp
        File = /.journal
        File = /.fsck
        File = /bacula
      }
    }
    
    #
    # When to do the backups, full backup on first sunday of the month,
    #  differential (i.e. incremental since full) every other sunday,
    #  and incremental backups other days
    Schedule {
      Name = "WeeklyCycle"
      Run = Full 1st sun at 23:05
      Run = Differential 2nd-5th sun at 23:05
      Run = Incremental mon-sat at 23:05
    }
    
    # This schedule does the catalog. It starts after the WeeklyCycle
    Schedule {
      Name = "WeeklyCycleAfterBackup"
      Run = Full sun-sat at 23:10
    }
    
    # This is the backup of the catalog
    FileSet {
      Name = "Catalog"
      Include {
        Options {
          signature = MD5
        }
        File = "/var/spool/bacula/bacula.sql"
      }
    }
    
    # Client (File Services) to backup
    Client {
      Name = bacula-fd
      Address = localhost
      FDPort = 9102
      Catalog = MyCatalog
      Password = "root"          # password for FileDaemon
      File Retention = 30 days            # 30 days
      Job Retention = 6 months            # six months
      AutoPrune = yes                     # Prune expired Jobs/Files
    }
    
    #
    # Second Client (File Services) to backup
    #  You should change Name, Address, and Password before using
    #
    #Client {
    #  Name = bacula2-fd
    #  Address = localhost2
    #  FDPort = 9102
    #  Catalog = MyCatalog
    #  Password = "@@FD_PASSWORD@@2"         # password for FileDaemon 2
    #  File Retention = 30 days            # 30 days
    #  Job Retention = 6 months            # six months
    #  AutoPrune = yes                     # Prune expired Jobs/Files
    #}
    
    
    # Definition of file storage device
    Storage {
      Name = File
    # Do not use "localhost" here
      Address = master                # N.B. Use a fully qualified name here
      SDPort = 9103
      Password = "root"
      Device = FileStorage
      Media Type = File
    }
    
    
    
    # Definition of DDS tape storage device
    #Storage {
    #  Name = DDS-4
    #  Do not use "localhost" here
    #  Address = localhost                # N.B. Use a fully qualified name here
    #  SDPort = 9103
    #  Password = "@@SD_PASSWORD@@"          # password for Storage daemon
    #  Device = DDS-4                      # must be same as Device in Storage daemon
    #  Media Type = DDS-4                  # must be same as MediaType in Storage daemon
    #  Autochanger = yes                   # enable for autochanger device
    #}
    
    # Definition of 8mm tape storage device
    #Storage {
    #  Name = "8mmDrive"
    #  Do not use "localhost" here
    #  Address = bacula.hpcsa.com                # N.B. Use a fully qualified name here
    #  SDPort = 9103
    #  Password = "@@SD_PASSWORD@@"
    #  Device = "Exabyte 8mm"
    #  MediaType = "8mm"
    #}
    
    # Definition of DVD storage device
    #Storage {
    #  Name = "DVD"
    #  Do not use "localhost" here
    #  Address = localhost                # N.B. Use a fully qualified name here
    #  SDPort = 9103
    #  Password = "@@SD_PASSWORD@@"
    #  Device = "DVD Writer"
    #  MediaType = "DVD"
    #}
    
    
    # Generic catalog service
    Catalog {
      Name = MyCatalog
    # Uncomment the following line if you want the dbi driver
    # dbdriver = "dbi:postgresql"; dbaddress = 127.0.0.1; dbport =
      dbname = "bacula"; dbuser = "root"; dbpassword = "root"
    }
    
    # Reasonable message delivery -- send most everything to email address
    #  and to the console
    Messages {
      Name = Standard
    #
    # NOTE! If you send to two email or more email addresses, you will need
    #  to replace the %r in the from field (-f part) with a single valid
    #  email address in both the mailcommand and the operatorcommand.
    #  What this does is, it sets the email address that emails would display
    #  in the FROM field, which is by default the same email as they're being
    #  sent to.  However, if you send email to more than one address, then
    #  you'll have to set the FROM address manually, to a single address.
    #  for example, a 'no-reply@mydomain.com', is better since that tends to
    #  tell (most) people that its coming from an automated source.
    
    #
      mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: %t %e of %c %l\" %r"
      operatorcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula: Intervention needed for %j\" %r"
      mail = root@localhost = all, !skipped
      operator = root@localhost = mount
      console = all, !skipped, !saved
    #
    # WARNING! the following will create a file that you must cycle from
    #          time to time as it will grow indefinitely. However, it will
    #          also keep all your messages if they scroll off the console.
    #
      append = "/var/log/bacula/bacula.log" = all, !skipped
      catalog = all
    }
    
    
    #
    # Message delivery for daemon messages (no job).
    Messages {
      Name = Daemon
      mailcommand = "/usr/sbin/bsmtp -h localhost -f \"\(Bacula\) \<%r\>\" -s \"Bacula daemon message\" %r"
      mail = root@localhost = all, !skipped
      console = all, !skipped, !saved
      append = "/var/log/bacula/bacula.log" = all, !skipped
    }
    
    # Default pool definition
    Pool {
      Name = Default
      Pool Type = Backup
      Recycle = yes                       # Bacula can automatically recycle Volumes
      AutoPrune = yes                     # Prune expired volumes
      Volume Retention = 365 days         # one year
    }
    
    # File Pool definition
    Pool {
      Name = File
      Pool Type = Backup
      Recycle = yes                       # Bacula can automatically recycle Volumes
      AutoPrune = yes                     # Prune expired volumes
      Volume Retention = 365 days         # one year
      Maximum Volume Bytes = 50G          # Limit Volume size to something reasonable
      Maximum Volumes = 100               # Limit number of Volumes in Pool
    }
    
    
    # Scratch pool definition
    Pool {
      Name = Scratch
      Pool Type = Backup
    }
    
    #
    # Restricted console used by tray-monitor to get the status of the director
    #
    Console {
      Name = bacula-mon
      Password = "root"
      CommandACL = status, .status
    }
    

































Director {                           
  Name = bacula-dir
  DIRport = 9101                
  QueryFile = "/etc/bacula/query.sql"
  WorkingDirectory = "/var/spool/bacula"
  PidDirectory = "/var/run"
  Maximum Concurrent Jobs = 1
  Password = "manish"         
  Messages = Daemon
  DirAddress = 127.0.0.1 #DIR address
}

Job {
  Name = "BackupLocalFiles"
  JobDefs = "DefaultJob"
}


Job {
  Name = "RestoreLocalFiles"
  Type = Restore
  Client=bacula-fd
  FileSet="Full Set"
  Storage = File
  Pool = Default
  Messages = Standard
  Where = /bacula/restore # restore point
}


FileSet {
  Name = "Full Set"
  Include {
    Options {
      signature = MD5
      compression = GZIP
    }    
File = /var/www/html/ #folder needed to backup
}
  Exclude {
    File = /var/lib/bacula
    File = /proc
    File = /tmp
    File = /.journal
    File = /.fsck
    File = /bacula
  }
}

Storage {
  Name = File
# Do not use "localhost" here
  Address = backup.hpcsa.com               # N.B. Use a fully qualified name here
  SDPort = 9103
  Password = "@@SD_PASSWORD@@"
  Device = FileStorage
  Media Type = File
}


# Generic catalog service
Catalog {
  Name = MyCatalog
# Uncomment the following line if you want the dbi driver
# dbdriver = "dbi:postgresql"; dbaddress = 127.0.0.1; dbport =
  dbname = "bacula"; dbuser = "root"; dbpassword = "manish"
}


# File Pool definition
Pool {
  Name = File
  Pool Type = Backup
  Label Format = Local-
  Recycle = yes                       # Bacula can automatically recycle Volumes
  AutoPrune = yes                     # Prune expired volumes
  Volume Retention = 30 days         # one year
  Maximum Volume Bytes = 10G          # Limit Volume size to something reasonable
  Maximum Volumes = 100               # Limit number of Volumes in Pool
}

bacula-dir -tc /etc/bacula/bacula-dir.conf


nano /etc/bacula/bacula-sd.conf


    [root@localhost ~]# cat /etc/bacula/bacula-sd.conf
    #
    # Default Bacula Storage Daemon Configuration file
    #
    #  For Bacula release 5.2.13 (19 February 2013) -- redhat (Core)
    #
    # You may need to change the name of your tape drive
    #   on the "Archive Device" directive in the Device
    #   resource.  If you change the Name and/or the
    #   "Media Type" in the Device resource, please ensure
    #   that dird.conf has corresponding changes.
    #
    
    Storage {                             # definition of myself
      Name = BackupServer-sd
      SDPort = 9103                  # Director's port
      WorkingDirectory = "/var/spool/bacula"
      Pid Directory = "/var/run/"
      Maximum Concurrent Jobs = 20
      SDAddress = bacula.hpcsa.com
    }
    
    #
    # List Directors who are permitted to contact Storage daemon
    #
    Director {
      Name = bacula-dir
      Password = "@@SD_PASSWORD@@"
    }
    
    #
    # Restricted Director, used by tray-monitor to get the
    #   status of the storage daemon
    #
    Director {
      Name = bacula-mon
      Password = "@@MON_SD_PASSWORD@@"
      Monitor = yes
    }
    
    #
    # Note, for a list of additional Device templates please
    #  see the directory <bacula-source>/examples/devices
    # Or follow the following link:
    #  http://bacula.svn.sourceforge.net/viewvc/bacula/trunk/bacula/examples/devices/
    #
    
    #
    # Devices supported by this Storage daemon
    # To connect, the Director's bacula-dir.conf must have the
    #  same Name and MediaType.
    #
    
    Device {
      Name = FileStorage
      Media Type = File
      Archive Device = /bacula/backup
      LabelMedia = yes;                   # lets Bacula label unlabeled media
      Random Access = Yes;
      AutomaticMount = yes;               # when device opened, read it
      RemovableMedia = no;
      AlwaysOpen = no;
    }
    
    #
    # An autochanger device with two drives
    #
    #Autochanger {
    #  Name = Autochanger
    #  Device = Drive-1
    #  Device = Drive-2
    #  Changer Command = "/usr/libexec/bacula/mtx-changer %c %o %S %a %d"
    #  Changer Device = /dev/sg0
    #}
    
    #Device {
    #  Name = Drive-1                      #
    #  Drive Index = 0
    #  Media Type = DLT-8000
    #  Archive Device = /dev/nst0
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes;
    #  RemovableMedia = yes;
    #  RandomAccess = no;
    #  AutoChanger = yes
    #  #
    #  # Enable the Alert command only if you have the mtx package loaded
    #  # Note, apparently on some systems, tapeinfo resets the SCSI controller
    #  #  thus if you turn this on, make sure it does not reset your SCSI
    #  #  controller.  I have never had any problems, and smartctl does
    #  #  not seem to cause such problems.
    #  #
    #  Alert Command = "sh -c 'tapeinfo -f %c |grep TapeAlert|cat'"
    #  If you have smartctl, enable this, it has more info than tapeinfo
    #  Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    #Device {
    #  Name = Drive-2                      #
    #  Drive Index = 1
    #  Media Type = DLT-8000
    #  Archive Device = /dev/nst1
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes;
    #  RemovableMedia = yes;
    #  RandomAccess = no;
    #  AutoChanger = yes
    #  # Enable the Alert command only if you have the mtx package loaded
    #  Alert Command = "sh -c 'tapeinfo -f %c |grep TapeAlert|cat'"
    #  If you have smartctl, enable this, it has more info than tapeinfo
    #  Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    #
    # A Linux or Solaris LTO-2 tape drive
    #
    #Device {
    #  Name = LTO-2
    #  Media Type = LTO-2
    #  Archive Device = /dev/nst0
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes;
    #  RemovableMedia = yes;
    #  RandomAccess = no;
    #  Maximum File Size = 3GB
    ## Changer Command = "/usr/libexec/bacula/mtx-changer %c %o %S %a %d"
    ## Changer Device = /dev/sg0
    ## AutoChanger = yes
    #  # Enable the Alert command only if you have the mtx package loaded
    ## Alert Command = "sh -c 'tapeinfo -f %c |grep TapeAlert|cat'"
    ## If you have smartctl, enable this, it has more info than tapeinfo
    ## Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    #
    # A Linux or Solaris LTO-3 tape drive
    #
    #Device {
    #  Name = LTO-3
    #  Media Type = LTO-3
    #  Archive Device = /dev/nst0
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes;
    #  RemovableMedia = yes;
    #  RandomAccess = no;
    #  Maximum File Size = 4GB
    ## Changer Command = "/usr/libexec/bacula/mtx-changer %c %o %S %a %d"
    ## Changer Device = /dev/sg0
    ## AutoChanger = yes
    #  # Enable the Alert command only if you have the mtx package loaded
    ## Alert Command = "sh -c 'tapeinfo -f %c |grep TapeAlert|cat'"
    ## If you have smartctl, enable this, it has more info than tapeinfo
    ## Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    #
    # A Linux or Solaris LTO-4 tape drive
    #
    #Device {
    #  Name = LTO-4
    #  Media Type = LTO-4
    #  Archive Device = /dev/nst0
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes;
    #  RemovableMedia = yes;
    #  RandomAccess = no;
    #  Maximum File Size = 5GB
    ## Changer Command = "/usr/libexec/bacula/mtx-changer %c %o %S %a %d"
    ## Changer Device = /dev/sg0
    ## AutoChanger = yes
    #  # Enable the Alert command only if you have the mtx package loaded
    ## Alert Command = "sh -c 'tapeinfo -f %c |grep TapeAlert|cat'"
    ## If you have smartctl, enable this, it has more info than tapeinfo
    ## Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    
    
    
    #
    # A FreeBSD tape drive
    #
    #Device {
    #  Name = DDS-4
    #  Description = "DDS-4 for FreeBSD"
    #  Media Type = DDS-4
    #  Archive Device = /dev/nsa1
    #  AutomaticMount = yes;               # when device opened, read it
    #  AlwaysOpen = yes
    #  Offline On Unmount = no
    #  Hardware End of Medium = no
    #  BSF at EOM = yes
    #  Backward Space Record = no
    #  Fast Forward Space File = no
    #  TWO EOF = yes
    #  If you have smartctl, enable this, it has more info than tapeinfo
    #  Alert Command = "sh -c 'smartctl -H -l error %c'"
    #}
    
    #
    # Send all messages to the Director,
    # mount messages also are sent to the email address
    #
    Messages {
      Name = Standard
      director = bacula-dir = all
    }






































Storage {                             # definition of myself
  Name = BackupServer-sd
  SDPort = 9103                  # Director's port
  WorkingDirectory = "/var/lib/bacula"
  Pid Directory = "/var/run/bacula"
  Maximum Concurrent Jobs = 20
  SDAddress = backup.hpcsa.com
}


Device {
  Name = FileStorage
  Media Type = File
  Archive Device = /bacula/backup 
  LabelMedia = yes;                   # lets Bacula label unlabeled media
  Random Access = Yes;
  AutomaticMount = yes;               # when device opened, read it
  RemovableMedia = no;
  AlwaysOpen = no;
}

bacula-sd -tc /etc/bacula/bacula-sd.conf

vi /etc/bacula/bconsole.conf

    [root@localhost ~]# cat /etc/bacula/bconsole.conf
    #
    # Bacula User Agent (or Console) Configuration File
    #
    
    Director {
      Name = bacula-dir
      DIRport = 9101
      address = 192.168.100.132
      Password = "root"
    }


systemctl enable bacula-dir &&  systemctl start bacula-dir &&  systemctl status bacula-dir  
systemctl enable bacula-sd &&  systemctl start bacula-sd &&  systemctl status bacula-sd 
systemctl enable bacula-fd &&  systemctl start bacula-fd &&  systemctl status bacula-fd 

bconsole
label
run

status director

restore all





























