##Tooling

Prepare NFS server
#inspect block devices attached to server
`lsblk`

#create partition on disks
`sudo gdisk /dev/xvdf`
`sudo gdisk /dev/xvdg`
`sudo gdisk /dev/xvdh`
#view partitions
`lsblk`

#install lvm2
`sudo yum install lvm2 -y`
`sudo lvmdiskscan`
`lsblk`

#create physical volumes
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`
`sudo pvs`
#add physical volumes to volume group
`sudo vgcreate webdate-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
`sudo vgs`

#create logical volumes
`sudo lvcreate -n lv-apps -L 9G webdata-vg`
`sudo lvcreate -n lv-logs -L 9G webdata-vg`
`sudo lvcreate -n lv-opt -L 9G webdata-vg`
`sudo lvs`
`lsblk`

#verify setup
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`

#format logical volumes
`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`

#create directories to store website files and store backup of log data
`sudo mkdir /mnt/apps`
`sudo mkdir /mnt/logs`
`sudo mkdir /mnt/opt`

#mount logical volumes in created directories
`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`

#install and configure NFS server
`sudo yum -y update`
`sudo yum install nfs-utils -y`
`sudo systemctl start nfs-server.service`
`sudo systemctl enable nfs-server.service`
`sudo systemctl status nfs-server.service`

`sudo chown -R nobody: /mnt/apps`
`sudo chown -R nobody: /mnt/logs`
`sudo chown -R nobody: /mnt/opt`
`sudo chmod -R 777 /mnt/apps`
`sudo chmod -R 777 /mnt/logs`
`sudo chmod -R 777 /mnt/opt`
`sudo systemctl restart nfs-server.service`
`sudo systemctl status nfs-server.service`
![alt text](./images/nfs status.PNG)

#open to configure NFS access and insert 
`sudo vi /etc/exports`
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
esc
:wq!
  
`sudo exportfs -arv`

#check NFS port
`rpcinfo -p | grep nfs`
![alt text](./images/ports.PNG)
  
Configure Database
#install mysql, create database and grant permission
`sudo apt install mysql-server -y`
`sudo apt update`
`sudo mysql`
create database tooling;
create user 'webaccess'@'<address>' identified by 'password';
grant all privileges on tooling.* to 'webaccess'@'<address>';
flush privleges;
show databases;
exit;

#open and change bind address
`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
change bind address to 0.0.0.0

#restart and confirm status 
`sudo systemctl restart mysql`
`sudo systemctl status mysql`
`sudo mysql`
show databases;
use tooling;
show tabes;
select * from users;

Prepare webservers
#install nfs client, create directory and mount to directory
`sudo yum install nfs-utils nfs4-acl-tools -y`
`sudo mkdir /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

#verify NFS is mounted successfully
`df -h`
#open and add code line
`sudo vi /etc/fstab`
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0

#install Remi's repository, Apache and PHP  
`sudo yum install httpd -y`
`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`
`sudo dnf module reset php`
`sudo dnf module enable php:remi-7.4`
`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`
`sudo systemctl start php-fpm`
`sudo systemctl enable php-fpm`
`sudo setsebool -P httpd_execmem 1`

#verify files in /var/wwww
`ls /var/www`
`sudo touch /var/www/test.md`
`ls /var/www`
  
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd`

`sudo vi /etc/fstab`
172.31.39.235:/mnt/apps /var/www nfs defaults 0 0

#fork tooling source code
`sudo yum install git -y`
`git init`
`git clone https://github.com/darey-io/tooling.git`
`ls`

`cd tooling/`
`ls`
`ls /var/www`

#
`sudo cp -R html/. /var/www/html`
`ls /var/www/html`
`ls html`
`cd ..`
 
#disable selinux
`sudo setenforce 0`

#open config file and edit
`sudo vi /etc/sysconfig/selinux`
SELINUX=disabled
  
#start and check status of Apache
`sudo systemctl start httpd`
`sudo systemctl status httpd`
#open and apply script to database
`sudo vi /var/www/html/functions.php`
mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql

cd tooling/
`sudo yum install mysql -y`
mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql

![alt text](./images/login.PNG)                                                               
![alt text](./images/home login.PNG)
                                                                               
