# WEB-SOLUTION-WITH-WORDPRESS


#Spinned up Servers by Launching two EC2 instances that will serve as "Web Server" and "database server" on AWS

![belmont0](https://user-images.githubusercontent.com/94229949/181261622-5357764e-80e5-4b59-bd2e-dc1f4d983260.png)

#Created 3 volumes in the same AZ (webserver 1,2,3 and db1,2,3) as the  Web and database Server EC2, each of 10 GiB.

![belmont2](https://user-images.githubusercontent.com/94229949/181262238-82a75426-04bf-4479-b66a-d24e652be794.png)


![belmont17](https://user-images.githubusercontent.com/94229949/181262478-e2e96a58-ec56-42da-b386-fdfd64404108.png)

#SSH into the Linux terminal through the windows terminal to  begin configuration


![belmont1](https://user-images.githubusercontent.com/94229949/181264088-fb536ae8-8129-4a49-b730-702109771783.png)


#Used 'lsblk' command to inspect what block devices are attached to the webserver.

![belmont3](https://user-images.githubusercontent.com/94229949/181264377-b0725ac5-1402-4399-86cc-641884c645e6.png)

#Use gdisk utility to create a single partition on each of the 3 disks. ' sudo gdisk /dev/xvdf ' 'sudo gdisk /dev/xvdg '  'sudo gdisk /dev/xvdh  '

![belmont4](https://user-images.githubusercontent.com/94229949/181265323-c3cc4361-2e55-4ccd-8851-79c4cc115afb.png)

#Ran the 'lsblk' utility to view the newly configured partition on each of the 3 disks.

![belmont5](https://user-images.githubusercontent.com/94229949/181266095-3b6f9a9a-3849-4934-bc31-9701e52ace20.png)

#Installed lvm2 package using sudo yum install lvm2.

![belmont](https://user-images.githubusercontent.com/94229949/181266665-273d2010-5ae6-4688-9e98-047d14f9d56c.png)

#' pvcreate /dev/xvdf1 /dev/xvdg1  /dev/xvdh1' utility to mark each of 3 disks as physical volumes  to be used by LVM

![belmont6](https://user-images.githubusercontent.com/94229949/181268809-b0817732-e4f9-4307-88a0-e32ef3c3c79d.png)

#I used vgcreate utility to add all 3 PVs to a volume group (VG). Named the VG webdata-vg using "sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1 "

#Used lvcreate utility to create 2 logical volumes. apps-lv  and logs-lv .' apps-lv ' will be used to store data for the Website while,  'logs-lv ' will  be used to store data for logs using  'sudo lvcreate -n apps-lv -L 14G webdata-vg '  ' sudo lvcreate -n logs-lv -L 14G webdata-vg '

#To verify the  Logical Volume has been created successfully i ran  'sudo lvs '

![belmont7](https://user-images.githubusercontent.com/94229949/181270729-02a4263c-80e7-4bbe-ade4-fd1eaa51a1f7.png)

#To format the logical volumes with ext4 filesystem , I ran  ' sudo mkfs -t ext4 /dev/webdata-vg/apps-lv ' and  ' sudo mkfs -t ext4 /dev/webdata-vg/logs-lv '

![belmont8](https://user-images.githubusercontent.com/94229949/181272117-4faee235-0e98-4ce3-87ea-d2ae91a00ee4.png)

#Created  ' /var/www/html ' directory to store website files using  'sudo mkdir -p /var/www/html ' and Created ' /home/recovery/logs to store backup of log data and ' sudo mkdir -p /home/recovery/logs '

![belmont9](https://user-images.githubusercontent.com/94229949/181273017-3c56d3fd-5203-40ff-9686-634b899a1c33.png)

#I went ahead to Mount /var/www/html on apps-lv logical volume using ' sudo mount /dev/webdata-vg/apps-lv /var/www/html/ '

![belmont10](https://user-images.githubusercontent.com/94229949/181275095-e42b70af-dc63-403a-8440-be8704f180f3.png)

#When mounting on ' /var/log ' the files present on the ' /var/log ' will get deleted ,to prevent this from happening i Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs using ' sudo rsync -av /var/log/. /home/recovery/logs/ ' and mount ' /var/log ' on  ' logs-lv ' logical volume using sudo  ' mount /dev/webdata-vg/logs-lv /var/log '

![belmont11](https://user-images.githubusercontent.com/94229949/181276763-d2e0237a-9df5-4c68-826d-5549331d1419.png)

#Then restored log files back into /var/log directory  ' sudo rsync -av /home/recovery/logs/. /var/log '
 
 ![belmont13](https://user-images.githubusercontent.com/94229949/181277237-3c9e5bf1-cc36-468e-8f15-87b670a680e5.png)
 
#Used  ' sudo blkid ' to get the UUID of the device that will be used to update the /etc/fstab file and ' vi  into ' /etc/fstab ' file so that the mount configuration will persist after restart of the server.

![belmont14](https://user-images.githubusercontent.com/94229949/181278498-5b7bfd75-9632-449d-858b-7d3e69f8309b.png)

#Updated  /etc/fstab in this format using the UUID

![belmont15](https://user-images.githubusercontent.com/94229949/181278801-8c22ff5e-c0fe-4df5-9d3c-580bdecbc051.png)

I Test the configuration and reload the daemon using ' sudo mount -a ' and ' sudo systemctl daemon-reload '

![belmont16](https://user-images.githubusercontent.com/94229949/181279210-a15edec5-4c57-48b2-9a13-e6d766576637.png)

##  Prepare the Database Server

#Created 3 volumes in the same AZ  database Server EC2, each of 10 GiB.

#Used 'lsblk' command to inspect what block devices are attached to the webserver.

#Use gdisk utility to create a single partition on each of the 3 disks. ' sudo gdisk /dev/xvdf ' 'sudo gdisk /dev/xvdg '  'sudo gdisk /dev/xvdh  '

#Installed lvm2 package using sudo yum install lvm2.

#' pvcreate /dev/xvdf1 /dev/xvdg1  /dev/xvdh1' utility to mark each of 3 disks as physical volumes  to be used by LVM

#Used lvcreate utility to create logical volume named ' db-lv '  using ' sudo lvcreate -n db-lv -L 14G vg-database '

![belmont23](https://user-images.githubusercontent.com/94229949/181384360-aa8ca5d2-b471-416a-ae48-abaf87933c0c.png)


#To format the logical volumes with ext4 filesystem , I ran  ' sudo mkfs -t ext4 /dev/vg-database/db-lv ' 

![belmont24](https://user-images.githubusercontent.com/94229949/181384633-acdca756-9803-4e7d-be9c-7d0a2b3a5df6.png)

## Install webserver on wordpress

#Updated the repository ' sudo yum -y update '

![belmont27](https://user-images.githubusercontent.com/94229949/181282579-7297451d-490b-447d-812d-05c60b4adc64.png)

#Installed wget, Apache and it’s dependencies using ' sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json '

![belmont28](https://user-images.githubusercontent.com/94229949/181283186-2067deab-17ab-4ab4-8b21-2f5bbd510dd9.png)

#Start Apache with command  'sudo systemctl enable httpd ' and ' sudo systemctl start httpd '

#Created  ' /var/www/html ' directory to store website files using  'sudo mkdir -p /var/www/html ' and Created ' /home/recovery/logs to store backup of log data and ' sudo mkdir -p /home/recovery/logs '


#I went ahead to Mount /var/www/html on apps-lv logical volume using ' sudo mount /dev/webdata-vg/apps-lv /var/www/html/ '


#When mounting on ' /var/log ' the files present on the ' /var/log ' will get deleted ,to prevent this from happening i Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs using ' sudo rsync -av /var/log/. /home/recovery/logs/ ' and mount ' /var/log ' on  ' logs-lv ' logical volume using sudo  ' mount /dev/webdata-vg/logs-lv /var/log '


#Then restored log files back into /var/log directory  ' sudo rsync -av /home/recovery/logs/. /var/log '
 
 
#Used  ' sudo blkid ' to get the UUID of the device that will be used to update the /etc/fstab file and ' vi  into ' /etc/fstab ' file so that the mount configuration will persist after restart of the server.


#Updated  /etc/fstab in this format using the UUID



I Test the configuration and reload the daemon using ' sudo mount -a ' and ' sudo systemctl daemon-reload '






## To install PHP and it’s dependencies ,i ran these commands:

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

![belmont29](https://user-images.githubusercontent.com/94229949/181285753-7535146b-ad4b-474a-9d50-1d4c1d21ce39.png)

sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo yum module list php

![belmont33](https://user-images.githubusercontent.com/94229949/181286384-91af5333-c4c0-4c65-8170-2b16b60f3f5e.png)

sudo yum module reset php

sudo yum module enable php:remi-8.0 -y

sudo yum install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

#Restart Apache using ' sudo systemctl restart httpd '

## Download wordpress and copy wordpress to var/www/html

  mkdir wordpress
  
  cd   wordpress
  
  sudo wget http://wordpress.org/latest.tar.gz
  
  sudo tar xzvf latest.tar.gz
  
  sudo rm -rf latest.tar.gz
  
  cp wordpress/wp-config-sample.php wordpress/wp-config.php
  
  cp -R wordpress /var/www/html/

![belmont35](https://user-images.githubusercontent.com/94229949/181289668-d8df642c-c5e6-4c38-aca7-55c4a033e9ee.png)

## Configured SELinux Policies

  sudo chown -R apache:apache /var/www/html/wordpress
  
  sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
  
  sudo setsebool -P httpd_can_network_connect=1
  
![belmont47](https://user-images.githubusercontent.com/94229949/181291264-ca958730-4ec5-42d8-adc5-705774d55fdd.png)


## Installed MySQL on WEB and DB Server EC2 by running these commands:

sudo mysql

CREATE DATABASE wdpress;

CREATE USER `myuser`@`%` IDENTIFIED BY 'mypass';

GRANT ALL ON wdpress.* TO 'myuser'@'%';
FLUSH PRIVILEGES;

SHOW DATABASES;

exit

![belmont43rd](https://user-images.githubusercontent.com/94229949/181385366-5a131f4b-ef9f-4460-8679-89d7473a054a.png)

## Configured WordPress to connect to remote database.

I#nstalled MySQL server on both web and database server

sudo vi  /etc/my.cnf to edit mysql configuration

![belmont44rd](https://user-images.githubusercontent.com/94229949/181386886-0b452233-19b6-42d2-805c-c598946a364a.png)

#Connected from the webserver to mysql database server using sudo mysql -h 172.31.0.250 -u myuser -p

![belmont46](https://user-images.githubusercontent.com/94229949/181386413-6b4b88d4-19d5-4320-bf10-68b1d194ff43.png)
