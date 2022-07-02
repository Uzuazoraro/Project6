# PROJECT 6: WEB SOLUTION WITH WORDPRESS

## STEP 1: Creating Volumes

We are going to configure storage infrastructure for web and database servers. We will also install wordpress and connect it to a remote MYSQL database server.

Our first step is to open AWS EC2. We'll be launching two instances in our AWS EC2

![AWS EC2 Instances](./Two EC2 Instances Launhed.png)
One Instance will serve as Webserver and the other Instance will serve as Database.

![volume creation for server and database](./Create Volume.png)

![Three volumes created](./VolumesCreated.png)
Three Server volumes created for Webserver.

![database volumes](./DatabaseVolumes.png)
Three Database volumes created.

## STEP 2: Configuration

`git init`
Getting into the git repo.

![git initialized](./Git Initialized.png)

`lsblk`
This command is used to inspect what block devices are attached to the server. Apply to both server and database server.

![Block devices](./BlockDevices.png)

`ls /dev/`
This command is used to inspect devices.

![Devices inspection](./Devices inspection.png)

`df -h`
This command is used to see the mounts and free space on the server. Apply to both server and database server.

![See mounts and free space on server](./Mounts&ServerFreeSpace.png)

`sudo gdisk /dev/xvdf`
We shall be using this gdisk utility to create partitions on each of the three disks. We shall apply it to both the server and the database server.

![PartitionTable](./PartitionTableScan.png)

`n`
This command will enable you to add a new partition. Apply it to both sides.

`8e00`
Enter this code to changed type of partition to 'Linux LVM'

`p`
This command will tell you more about the partitions and the free space available.

![Partitions&free space](./Partitions&FreeSpace.png)

`w`
We use this command to write the GPT data. This will overwrite existing partitions.

![Write new partition table](./WrittenNewPartitionTable.png)
Successful completion of /dev/xdvf partition. Writing new GPT partition to /dev/xdvf.

Now we'are going to follow same procedure for the other two disks.

GDISK XDVG Configuration begins:

`sudo gdisk /dev/xdvg`
This will create the partition table scan.

![Partition table scan](./PartitionTable.png)

`n`
Use to add new partition

`8e00'
Change type of partition to Linux LVM

`W`
Write table to disk and exit.

`y`
Confirmed writimg to new GPT partition.

![Write new GUID Partition](./WriteNewGPT to dev xvdg.png)
Successful completion of /dev/xdvg partition.

Now, let us configure XDVH.
It's going to be same procedure.

![dev xdvh successful](./dev xvdh completed.png)
Successful completion of /dev/xdvh partition in the server.

`lsblk`
Used this utility to view the newly configured partition on each of the 3 disks.

![show View of the created partitions](./PartitionView.png)

NOTE:
Same commands were used on the Database Server and disk partitioning was successfully completed.

`sudo yum install lvm2`
Command used to install the lvm2 package. Use on both the Server and the Database server.

![Installation of lvm2 package](./LVM2 Package Installed.png)

`sudo lvmdiskscan`
This command was used to check for available partitions.
Used for both Server and Database Server.

![Chack for available partitions](./Check for partitions.png)

Pvcreate utility used to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

![Create & verify creation of physical volume](./CreatePhysicalVolume.png)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg
`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

`sudo vgs`
Used to verify the addition of all 3 PVS to a volume group.

![VolumeGroup created](./VolumeGroup.png)
Volume was successfully created to both Server and Database server.

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

`sudo lvs` Verify creation and running of logical volumes.

![Creation of logical volume](./LogicalVolume.png)

VERIFY THE ENTIRE SETUP WITH THESE COMMANDS
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

![All volume setup](./Entire Volume setup.png)

`sudo lsblk`

![Verification of entire volume setup](./VerifyEntireSetup.png)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`
Logical volumes formatted.

![Formatted logical volume](./LogicalVolume.png)

Create /var/www/html directory to store website files
`sudo mkdir -p /var/www/html`

Create /home/recovery/logs to store backup of log data
`sudo mkdir -p /home/recovery/logs`

Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`

![MkdirMount and recover files](./MkdirMountRecoverFiles.png)

Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files back into /var/log directory
`sudo rsync -av /home/recovery/logs/. /var/log`

![Mounting and restoration of log files](./Mount&RestoreLogFiles.png)

Update /etc/fstab file so that the mount configuration will persist after restart of the server. Use this command to update the server.

`sudo vi /etc/fstab`

![Update fstab file](./vi etc fstab file.png)

`sudo mount -a`
Use this command to test the configuration.

`sudo systemctl daemon-reload`
Use this command to reload the daemon.

Verify your setup by running `df -h`, output must look like this:

![Output](./df -h output.png)

Prepare the logical volume for database server.
`sudo lvcreate -n db-lv -L 14G webdata-vg`

`sudo lvcreate -n logs-lv -L 14G webdata-vg`

![DB Logical volume created](./DB LogicalVolume.png)

Verify the entire setup with this command:
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

`sudo lsblk`

![Verify entire setup](./VerifyDBSetup.png)

Make your DB file system
`sudo mkfs -t ext4 /dev/webdata-vg/db-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![Formatted Logical Volume](./DB LogicalVolFormatted.png)

Create /DB/www/html directory to store website files
`sudo mkdir -p /db`

`sudo mkdir -p /home/recovery/logs`
This is use to back up data.

Mount /var/www/html on apps-lv logical volume
`sudo mount /dev/webdata-vg/db-lv /db`

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system)
`sudo rsync -av /var/log/. /home/recovery/logs/`

![Log/home recovery logs](./Log files.png)

`sudo vi /etc/fstab`

![DBFstab](./DBfstab.png)
DBFstab updated.

Test the configuration and reload the daemon

`sudo mount -a`
`sudo systemctl daemon-reload`

Verify your setup by running `df -h`, output must look like this:

![Verify the setup](./SetupVerification.png)

 Install WordPress on your Web Server EC2
`sudo yum -y update`

Install wget, Apache and it’s dependencies
`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

START APACHE

`sudo systemctl enable httpd`
`sudo systemctl start httpd`

![Apache installed and running](./Apache Installed.png)

We are going to install PHP and it’s dependencies. We will use the following commands to get it done:

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`

![Installing php dependencies](./Php Installation1.png)

`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`

![Installing php dependencies](./Php dependency.png)

`sudo yum module list php`

![Installing php dependencies](./Php dependency1.png)

`sudo yum module reset php`

![Installing php dependencies](./Php dependency2.png)

`sudo yum module enable php:remi-7.4`

![Installing php dependencies](./Php dependency3.png)

`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`

![Installing php dependencies](./Php dependency4.png)

`sudo systemctl start php-fpm`

`sudo setsebool -P httpd_execmem 1`

![Installing php dependencies](./Php other dependencies.png)

Restart Apache

`sudo systemctl restart httpd`

Go to your AWS EC2 Werserver Security group. Add a rule, HTTP port 80. Then copy the public IPV4 and paste it on your brwoser. Press enter and see Apache test page.

![See apache test page on a browser](./Apachetestpage.png)

Download wordpress and copy wordpress to var/www/html

`mkdir wordpress`

`cd wordpress`

`sudo wget http://wordpress.org/latest.tar.gz`

![uploading wordpress files](./wordpressfiles.png)

`sudo tar xzvf latest.tar.gz`

![uploading wordpress files](./wordpressfiles1.png)

`sudo rm -rf latest.tar.gz`

`sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php`

`sudo cp -R wordpress /var/www/html/`

Configure SELinux Policies

These are policies that borders around security and access.
We'll be making use of the following three commands:

`sudo chown -R apache:apache /var/www/html/wordpress`
This is change of wonership.

`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`

`sudo setsebool -P httpd_can_network_connect=1`

![Access and policies](./policies configuration.png)

Install MySQL on your DB Server EC2
`sudo yum update`
`sudo yum install mysql-server`

![Installation of mysql](./Mysql Installed.png)

`sudo vi /etc/my-cnf`

![file creation updated](./vi etc files.png)

`sudo systemctl restart mysqld`

`sudo systemctl enable mysqld`

`sudo systemctl status mysqld`
Mysql server confirmed. It is active and running.

![confirm the status of mysql server](./mysql server status)

## Configure DB to work with WordPress

`sudo mysql`
This will take you to mysql.

`CREATE DATABASE wordpress;`

`CREATE USER`admin`@`%`IDENTIFIED BY 'PassWord.1';`

`GRANT ALL ON wordpress.* TO 'admin'@'%';`

`flush privileges;`

`SHOW DATABASES;`

![configuration commands](./DB configuration to work with wordpress)

`exit'
This will take you out of mysql.

## Configure WordPress to connect to remote database

Update your security group with mysql/aurora rules.

## Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client

Go back to your server and install mysql with this command:
`sudo yum install mysql`

![Mysql installed successfully](./Mysql installed on server)

`sudo mysql -u admin -p -h 172.31.95.82`

![Successful connection of servers](./Connecting Server to DBServer)

`SHOW DATABASES;`

![Successfully shown](./show databases)

## Change permissions and configuration so Apache could use WordPress

`cd /var`

`cd www`

`cd html`

`cd wordpress`

![Got into wordpress directory](./wordpress directory)

Edit wordpress configuration to show wordpress page.
`sudo vi wp-config.php`

![wordpress configuration](./wordpress configuration completed.png)

![wordpress welcome page](./wordpress page.png)

![wordpress welcome page](./WP welcome page.png)

![wordpress completed(.WP success page.png)
