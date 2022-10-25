`lsblk`
`sudo gdisk /dev/xvdf`
`sudo gdisk /dev/xvdg`
`sudo gdisk /dev/xvdh`
`lsblk`
`sudo yum install lvm2 -y`
`sudo lvmdiskscan`
`lsblk`
`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`
`sudo pvs`
`sudo vgcreate webdate-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`
`sudo vgs`
`sudo lvcreate -n lv-apps -L 9G webdata-vg`
`sudo lvcreate -n lv-logs -L 9G webdata-vg`
`sudo lvcreate -n lv-opt -L 9G webdata-vg`
`sudo lvs`
`lsblk`
`sudo vgdisplay -v #view complete setup - VG, PV, and LV`
`sudo mkfs -t xfs /dev/webdata-vg/lv-apps`
`sudo mkfs -t xfs /dev/webdata-vg/lv-logs`
`sudo mkfs -t xfs /dev/webdata-vg/lv-opt`
`sudo mkdir /mnt/apps`
`sudo mkdir /mnt/logs`
`sudo mkdir /mnt/opt`
`sudo mount /dev/webdata-vg/lv-apps /mnt/apps`
`sudo mount /dev/webdata-vg/lv-logs /mnt/logs`
`sudo mount /dev/webdata-vg/lv-opt /mnt/opt`
`sudo yum -y update`

`sudo apt install mysql-server -y`
`sudo apt update`
`sudo mysql`
create database tooling;
create user 'webaccess'@'<address>' identified by 'password';
grant all privileges on tooling.* to 'webaccess'@'<address>';
flush privleges;
show databases;
exit;
`sudo systemctl status mysql`
`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`
change bind address to 0.0.0.0
`sudo systemctl restart mysql`
`sudo systemctl status mysql`
sudo mysql
show databases;
use tooling;
show tabes;
select * from users;

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
`sudo vi /etc/exports`
/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
esc
:wq!
`sudo exportfs -arv`
`rpcinfo -p | grep nfs`
`sudo yum install nfs-utils nfs4-acl-tools -y`
`sudo mkdir /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`
`sudo vi /etc/fstab`
`df -h`
`sudo vi /etc/fstab`
<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0
`sudo yum install httpd -y`
`sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
`sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`
`sudo dnf module reset php`
`sudo dnf module enable php:remi-7.4`
`sudo dnf install php php-opcache php-gd php-curl php-mysqlnd`
`sudo systemctl start php-fpm`
`sudo systemctl enable php-fpm`
`sudo setsebool -P httpd_execmem 1`
`ls /var/www`
`sudo touch /var/www/test.md`
`ls /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`
`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd`
`sudo vi /etc/fstab`
172.31.39.235:/mnt/apps /var/www nfs defaults 0 0
`sudo yum install git -y`
`git init`
`git clone https://github.com/darey-io/tooling.git`
`ls`

`cd tooling/`
`ls`
`ls /var/www`
`sudo cp -R html/. /var/www/html`
`ls /var/www/html`
`ls html`
`cd ..`
`sudo setenforce 0`
`sudo vi /etc/sysconfig/selinux`
SELINUX=disabled
`sudo systemctl start httpd`
`sudo systemctl status httpd`
`sudo vi /var/www/html/functions.php`
mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
cd tooling/
sudo yum install mysql -y

mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql
sudo vi tooling-db.sql
