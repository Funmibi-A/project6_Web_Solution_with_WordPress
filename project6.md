# Project 6 -  Web Solution With WordPress

## Step 1 - Launch an instance with the red hat AMI

![launching an instance](./images/launch_instance.jpg)


## Edit the security group to allow ssh connections

![Allow_ssh](./images/add_ssh.jpg)

## Step 2 - Create and attach three volumes to the new instance.

![add_volumes](./images/add_volumes.jpg)

## Step 3 - Log into the instance and begin configuration

 > Update the repository

      sudo yum update

 > Upgrade the repository

      sudo yum upgrade

> Run the `lsblk` command used to inspect what block devices are attached to the server, In this case, you should see /dev/xvdf, /dev/xvdh, xvdg

    lsblk

![list attached volume](./images/lsblk.jpg)


> Step 4 - Run the `df -h` command used to see free space and mounted partition on the server.
 
    df -h

![free_space and mounts](./images/free_space.jpg)


## Step 5 - Create new partition 

> Create a single partition on each of the 3 attached volume

    using the fdisk utility run the command

    sudo fdisk /dev/xvdf  - 1st disk
    sudo fdisk /dev/xvdg  - 2nd disk
    sudo fdisk /dev/xvdh  - 3rd disk

![creating partition](./images/partition.jpg)


## Step 6 - View newly configured partiions on the attached drives.

> Running the `lsblk` command command to view the block devices attached to the server after been partitions using the fdisk utility

![volumes after been partiioned using fdisk](./images/partitioned.jpg)

## Step 7 - Install lvm2 package

    sudo yum install lvm2

![install lvm2](./images/lvm2.jpg)
  
> Run `lvdiskscan` command to check for available partitions

    sudo lvmdiskscan
  
![lvmdiskscan](./images/lvmdiskcscan.jpg)

## Step 8 - pvcreate

> Use the `pvcreate` utility to make each of the 3 attached devices as physical volums to be used by LVM

    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

![creating physical volume](./images/physical_volume.jpg)

> Verity the physical volume has been created successfully by running 

    sudo pvs

![sudo_pvs](./images/sudo_pvs.jpg)

## Step 9 - vgcreate

> Use the `vgcreate` utility to all 3 PVs to a volume group, name the VG 'webdata-vg

    sudo vgcreate webdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

![create a volumen group](./images/vgcreate.jpg)

> Verify the VG has been created successfully by running the `sudo vgs` command.

    sudo vgs

![verify volume group](./images/verify_vg.jpg)

## Step 10 - lvcreate

> Create 2 logical volumes using the `lvcreate` utility, name them apps-lv and logs-lv

    Note: For apps-lv (use half of the PV size)
              logs-lv (use the remaining space of the PV size)

      apps-lv: will be used to store data for the website
      logs-lv: used to store data for logs.

  
    `sudo lvcreate -n apps-lv -L 14G webdata-vg`
    `sudo lvcreate -n logs-lv -L 14G webdata-vg`

![create_logical_volume](./images/create_logical_volumes.jpg)

> Verify logical volume was created successfully

    sudo lvs

![verify_logical_volume](./images/verify_logical_volume.jpg)


## Step 11 - Verify the entire set up

> command 1
 
    sudo vgdisplay -v #view complete setup - VG, PV, and LV

> volume group 

![volume_group](./images/vg.jpg)

> logical volume

![logical_group](./images/lv.jpg)

> physical volume

![physical_group](./images/pv.jpg)

> command 2

    sudo lsblk
![sudo_lsblk](./images/lsblk.jpg)

## Step 12 - Format logical volumes

> Using the mkfs.ext4 command 

    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

![Format logical volume](./images/format_logical_volume.jpg)


## Step 13 - Create /var/www/html directory to store website files

    sudo mkdir -p /var/www/html

![create directory for website files](./images/html_directory.jpg)

## Step 14 - Create /home/recovery/logs to store backup of log data

    sudo mkdir -p /home/recovery/logs

![create a directory for log files](./images/logs_directory.jpg)

## Step 15 - Mount /var/www/html on apps-lv logical volume

    sudo mount /dev/webdata-vg/apps-lv /var/www/html

## Step 16 - Use the rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

    sudo rsync -av /var/log/. /home/recovery/logs/

## Step 17 - Mount /var/log on logs-lv logical volume
   
    Note: all the existing data on /var/log will be deleted. That is why step 16 above is very important.

    sudo mount /dev/webdata-vg/logs-lv /var/log

## Step 18 - Restore log files back into /var/log directory

    sudo rsync -av /home/recovery/logs/. /var/log

## Step 19 - Update the `/etc/fstab` file so that the mount configuration will persist 

> The UUID of the device will be used to update the /etc/fstab file

> run the command `sudo blkid` to get the uuid of the logical volumes

    sudo blkid

![blkid](./images/blkid.jpg)

> Edit the /etc/fstab file

    suod vi /etc/fstab

![uuid for logical drives](./images/uuid.jpg)

> Test the configuration and reload the daemon

    sudo mount -a
    sudo systemctl daemon-reload

> Run the command `df -h` to verify the setup

![df -h command](./images/df-h.jpg)


## Prepare the Database Server

> Lunch a second RedHat EC2 instance for the database server.

> Update repository

    sudo yum -y update 

> Upgrade repository

    sudo yum -y upgrade

> Run the `lsblk` command used to inspect what block devices are attached to the server, In this case, you should see /dev/xvdf, /dev/xvdh, xvdg

    lsblk

![lsblk on the db server](./images/lsblk_db.jpg)

> Run the `df -h` command used to see free space and mounted partition on the server.
 
    df -h

![free_space and mounts on db server](./images/free_space_db.jpg)


> Create a single partition on each of the 3 attached volume

    using the fdisk utility run the command

    sudo fdisk /dev/xvdf  - 1st disk
    sudo fdisk /dev/xvdg  - 2nd disk
    sudo fdisk /dev/xvdh  - 3rd disk

![creating partition](./images/partition_db.jpg)

![creating partition](./images/partition2_db.jpg)

![creating partition](./images/partition3_db.jpg)

> View newly configured partiions on the attached drives.

> Running the `lsblk` command command to view the block devices attached to the server after been partitions using the fdisk utility

    lsblk 

![volumes after been partioned using fdisk](./images/partitioned_db.jpg)

> Install yum install lvm2

    sudo yum install lvm2

![install lvm_db](./images/lvm2_db.jpg)


> Run `lvmdiskscan` command to check for available partitions

    sudo lvmdiskscan
  
![lvmdiskscan](./images/lvmdiskcscan_db.jpg)


## pvcreate

> Use the `pvcreate` utility to make each of the 3 attached devices as physical volums to be used by LVM

    sudo pvcreate /dev/xvdf1
    sudo pvcreate /dev/xvdg1
    sudo pvcreate /dev/xvdh1

![pvcreate for the db server](./images/pvcreate_db.jpg)

> Verity the physical volume has been created successfully by running 

    sudo pvs

![sudo_pvs](./images/pvs_db.jpg)


## vgcreate

> Use the `vgcreate` utility to all 3 PVs to a volume group, name the VG 'webdata-vg

    sudo vgcreate dbdata-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1

![create a volume group](./images/vgcreate_db.jpg)

> Verify the VG has been created successfully by running the `sudo vgs` command.

    sudo vgs

![verify volume group](./images/verify_vg_db.jpg)

## lvcreate

> Create 2 logical volumes using the `lvcreate` utility, name them apps-lv and logs-lv

    Note: For db-lv (use half of the PV size)
              logs-lv (use the remaining space of the PV size)

      apps-lv: will be used to store data for the website
      logs-lv: used to store data for logs.

  
    `sudo lvcreate -n apps-lv -L 14G dbdata-vg`
    `sudo lvcreate -n logs-lv -L 14G dbdata-vg`

![create_logical_volume](./images/create_logical_volumes_db.jpg)

> Verify logical volume was created successfully

    sudo lvs

![verify_logical_volume](./images/verify_logical_volume_db.jpg)


## Verify the entire set up

> command 1
 
    sudo vgdisplay -v #view complete setup - VG, PV, and LV

> volume group 

![volume_group](./images/vg_db.jpg)

> logical volume

![logical_group](./images/lv_db.jpg)

> physical volume

![physical_group](./images/pv_db.jpg)

> command 2

    sudo lsblk
![sudo_lsblk](./images/sudo_lsblk_db.jpg)


> Install wget, Apache and it's dependencies
    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

## Format logical volumes

> Using the mkfs.ext4 command 

    sudo mkfs -t ext4 /dev/dbdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/dbdata-vg/logs-lv

![Format logical volume](./images/format_logical_volume_db.jpg)


## Create /db directory to store website files

    sudo mkdir -p /db

![create directory for website files](./images/db_directory.jpg)

## Create /home/recovery/logs to store backup of log data

    sudo mkdir -p /home/recovery/logs

![create a directory for log files](./images/log_directory_db.jpg)

## Mount /var/www/html on apps-lv logical volume

    sudo mount /dev/dbdata-vg/db-lv /db

![mount_point](./images/mount_point_db.jpg)

## Use the rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

    sudo rsync -av /var/log/. /home/recovery/logs/

![rsync_ reovery](./images/rsync.jpg)


## Mount /var/log on logs-lv logical volume
   
    Note: all the existing data on /var/log will be deleted. That is why step 16 above is very important.

    sudo mount /dev/dbdata-vg/logs-lv /var/log

## Restore log files back into /var/log directory

    sudo rsync -av /home/recovery/logs/. /var/log

## Update the `/etc/fstab` file so that the mount configuration will persist 

> The UUID of the device will be used to update the /etc/fstab file

> run the command `sudo blkid` to get the uuid of the logical volumes

    sudo blkid

![blkid_db](./images/blkid_db.jpg)

> Edit the /etc/fstab file

    sudo vi /etc/fstab

![uuid for logical drives](./images/uuid.jpg)


> Test the configuration and reload the daemon

    sudo mount -a
    sudo systemctl daemon-reload

> Run the command `df -h` to verify the setup

![df -h command](./images/df-h.jpg)

## Installing wordPress on your Web Server 

> Launch the webserver and log into it

> Update the repository

    sudo yum -y update

![yum_update](./images/yum_update.jpg)

> Install wget, Apache and it's dependencies

    sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json

![apache_dependencies](./images/apache_dependencies.jpg)

> Start Apache

    sudo systemctl start httpd
    sudo systemctl enable httpd

![apache server status](./images/apache_server_status.jpg)
> Installing php and it's dependencies

    sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

![php dependencies](./images/php_dependency1.jpg)

    sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-9.rpm

    Note: The commands above has to match the current version of your RedHat O.S
    In my case right, I'm running a version 9.1,  that's why I have the "...release-9.rpm"

![php dependencies](./images/php_dependency2.jpg)

    sudo yum module list php

![php dependencies](./images/php_dependency3.jpg)

    sudo yum module reset php

![php dependencies](./images/php_dependency4.jpg)

    sudo yum module enable php:remi-7.4

![php dependencies](./images/php_dependency5.jpg)

    sudo yum install php php-opcache php-gd php-curl php-mysqlnd

![php dependencies](./images/php_dependency6.jpg)

    sudo systemctl start php-fpm

    sudo systemctl enable php-fpm

![php dependencies](./images/php_dependency7.jpg)

    sudo setsebool -P httpd_execmem 1

> Restart the Apache server

    sudo systemctl restart httpd

> Download wordpress and copy to the var/www/html directory

> Create a folder named 'wordpress'

    mkdir wordpress

> Navigate into the 'folder'

    cd wordpress

> Download wordpress

    sudo wget http://wordpress.org/latest.tar.gz

![download wordpress](./images/download_wordpress.jpg)   

> Extract the `latest.tar.gz` file

    sudo tar xzvf latest.tar.gz

![extract wordpress](./images/extract_wordpress.jpg)

> Delete the compressed wordpress file

    sudo rm -rf latest.tar.gz

> Copy the 'wordpress-config-sample.php' file into the 'wordpress' directory

    sudo cp wordpress-config-sample.php wordpress/wp-config.php

> Edit the 'wordpress-config.php' file to and include the database settings

![editing the config.php file](./images/editing_wp-config_file.jpg)

> Copy the 'wordpress' directory into the html directory

    sudo cp -R wordpress /var/www/html/

> Configure SELinux Policies

    sudo chown -R apache:apache /var/www/html/wordpress

    sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R

    sudo setsebool -P httpd_can_network_connect=1 


## Log back into the database server

> Update the repository           

    sudo yum update

> Install mysql-server

    sudo yum install mysql-server

![installing mysql server](./images/installing_mysql_server.jpg)

> Restart mysqld service

    sudo systemctl restart mysqld

> Enable mysqld service

    sudo systemctl enable mysqld

![enable_mysqld](./images/enable_mysqld.jpg)

> Verify status

    sudo systemctl status mysqld

![verify_mysqld_status](./images/verify_mysqld_status.jpg)


## Configure database to work with wordpress

```SQL
 sudo mysql -u root
 CREATE DATABASE wordpress;
 CREATE USER 'Funmibi'@'172.31.60.196' IDENTIFIED BY 'us3rp@ss';
 GRANT ALL ON wordpress.* TO 'Funmibi'@'172.31.60.196';
 FLUSH PRIVILEGES;
 SHOW DATABASES;
 exit
```

![create db and user](./images/create_db_and%20_user.jpg)

## Configure WordPress to connect to remote database
Hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server’s IP address, so in the Inbound Rule configuration specify source as /32

![editing inbould rule for mysql port 3306](./images/security_rule_for_mysql_access.jpg)

## Edit the `my.cnf` file 

![editing my.cnf file](./images/editing_my.cnf_file.jpg)

## Log back into the web-server

> Install MySQL client and test that you can connect from your web server to you DB server using `mysql-client`

    sudo yum install mysql

![installing my sql client](./images/installing_mysql_client.jpg)

> Login frm mysql server from the mysql client

    sudo mysql -u admin -p -h 172.31.51.148

> Verify that you can successfully execute the `SHOW DATABASES;` command and see a list of existing databases.

    SHOW DATABASES;

![remote login from the webserver to the mysql server](./images/remote_login.jpg)

> Change permissions and configuration so Apache could use WordPress

    sudo chown -R apache:apache /var/www/html/wordpress

> Enable TCP port 80 inbound rules configuration for your web server EC2 (enable from everywhere 0.0.0.0/0 or from your workstations's IP)


    
> Access from a web browser the link to your WordPress

    http://52.91.117.194/wordpress/

![wordpress](./images/wordpress1.jpg)

![wordpress](./images/wordpress2.jpg)


![wordpress](./images/wordpress3.jpg)

   
    




