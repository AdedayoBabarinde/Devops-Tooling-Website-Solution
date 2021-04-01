**Devops Tooling Website Solution**


In this Project the set up of NFS shared folders to 3 Apache Web Servers will be demonstrated.

This is caried out to provide a Devops Tooling Website Solution.

The NFS shared folders will store var/www/ and  httpd logs contents to the NFS server and the project is executed in Virtual Box.

The aim of this work is to understand the use of  NFS servers to store logs and shared folders in certain location.

**Network Information**

- MySQL Server 192.168.1.132
- Web Server1 192.168.1.223
- Web Server2 192.168.1.122
- Web Server3 192.168.1.249
- NFS Server 192.168.1.142




**SETTING UP THE NFS SERVER**

- I added 3 disks to the NFS server and i used `gdisk` to create LVM partition on the disks /dev/sdb, /dev/sdc, /dev/sdd

```
sudo gdisk /dev/sdb
sudo gdisk /dev/sdc
sudo gdisk /dev/sdd	
```



I created physical volume,logical volumes and volume group as follows:
- ```Pvcreate /dev/sdb1 /dev/sdc1 /dev/sdd1``` --- creating physical volumes
 

- ```Vgcreate vg-apps /dev/sdb1``` – to create volume group for apps on the ```/dev/sdb```
 
- ```Vgcreate vg-logs /dev/sdc1``` – to create volume group for logs on the ```/dev/sdc```
 
- ``` Vgcreate vg-opt /dev/sdd1``` – to create volume group for opt on the ```/dev/sdd```

- ```lvcreate -l 100%FREE -n lv-apps vg-apps``` – to create logical volumes for app group using all the available space

- ```lvcreate -l 100%FREE -n lv-logs vg-logs``` – to create logical volumes for logs using all the available space
 
- ```lvcreate -l 100%FREE -n lv-opt vg-opt``` – to create logical volumes for opt on the volume group opt using all the available space

I formatted the disks with the xfs file system as show below
```mkfs.xfs /dev/vg-apps/lv-apps``` – first make it an xfs filesystem
```mkfs.xfs /dev/vg-logs/lv-logs``` – first make it an xfs filesystem
```mkfs.xfs /dev/vg-opt/lv-opt``` – first make it an xfs filesystem


I created mount points for logs,html and opt on 
```/mnt``` directory for the logical volumes as follows

I created mount points directory for the logical volumes

- ```mkdir /mnt/html``` – making directory for html
- ```mkdir /mnt/logs``` – making directory for logs
- ```mkdir /mnt/opt``` – making directory for logs


I mounted them as shown below

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/mount.jpg)



![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/xfs.png)


To ensure that all the mounts  persists after reboot i edited the /etc/fstab config as follows
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/uuid.jpg)


Then i Checked the mount.
```df -h```

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/output.png)



I changed the permision on the following folders as follows:
```chmod 777 /mnt/html/```
```chmod 777 /mnt/logs/```
```chmod 777 /mnt/opt/```

 

**Installing NFS Server**

```sudo apt install nfs-kernel-server```

Enabling nfs service

 ```sudo systemctl restart nfs-kernel-server```


I edited the /etc/exports to allow the NFS for remote servers as shown below

```sudo nano /etc/exports```


![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/export.png)

Export NFS shares as shown below
```exportfs -ra```
```exportfs -v```

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/exportfs.png)


I used the following command to open port 2049 on the webserver

```sudo ufw allow from webserver_ip to any port nfs```




**Step 2 — Configure the database server**

Install Mysql

```sudo apt install mysql-server```

-I performed mysql secure installation

```sudo mysql_secure_installation```


 I Logged in as root , created a database named "tooling" and created a user account named "webaccess" as shown in the figure below:

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/mysqllogin.png)


-
-I Granted permission to webaccess user on tooling database to do anything only from the webservers subnet (192.168.1.0/24)

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/privi.png)



- I saved changes

```FLUSH PRIVILEGES;```

Exit mysql.

-I edited the bind config  file on the mysql server to allow remote connections

 ```sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf```

I changed the bind address  as follows
 ```bind-address            = 0.0.0.0```


 -I restarted the MySQL service to effect the changes 

 ```sudo systemctl restart mysql```

 - I enabled Firewall

 ```sudo ufw enable```

 - Allow mysql connection from the subnet range

```sudo ufw allow from 192.168.1.0/24 to any port 3306```


-I installed GIT ON THE mysql server  to allow importation of sql dump file .

```sudo apt install git```

-I cloned the required github repository
```git clone https://github.com/dinulhaque/tooling.git /tmp/tooling```

-I imported sql dump file

```mysql -u webaccess  -h 192.168.1.120 -p tooling  < /tmp/tooling/tooling-db.sql```



 **Prepare the Web Servers**

 -I installed NFS client

 ```sudo apt install nfs-common```

 -I mounted /var/www/ and target the NFS server’s export for apps



 - Install apache

 ```sudo apt install apache2```

 - I backed up the contents of `/var/www` directory as follows:

 Create a backup directory with `sudo mkdir /varwww.bak`
 
 Copy the contents of `/var/www` into the backup directory with `sudo cp -R /var/www/* /varwww.bak`

I mounted /var/www/ and targeted the NFS server’s export for apps with `sudo mount 192.168.1.142:/mnt/html /var/www`



I backed up the Apache log files before mounting

I created the back up directory for log files with `sudo mkdir /varlogapache2.bak`

I backed up the contents of the log folder for Apache with `sudo cp -R /var/log/apache2/* /varlogapache2.bak`

 -I located the log folder for Apache and mount it, targeting the NFS server’s export for logs
 `sudo mount 192.168.1.142:/mnt/logs /var/log/apache2`


-I Forked a tooling source code from  my Github account

I install git on the webservers
`sudo apt install git`
I cloned the repository to the webservers
`git clone https://github.com/darey-io/tooling.git`

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/clone.jpg)




**Deployment of the tooling website’s code to the webserver**

- I checked if the tooling website have been cloned succesfully
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/check.jpg)

- I copied the contents of the `html` folder inside the `tooling` directory(`tooling/html`) in to the Apache 
`/var/www` directory as shown
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/copy.jpg)


- I checked the contents of `/var/www/html` to verify if the contents of `tooling/html` have been copied as shown:
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/verif.jpg) 


- I installed `php` on the webserver
`sudo apt install php libapache2-mod-php php-mysql`


- I changed the page loading sequence and Apache defaults landing page by making `login.php` first in the queue as follows:
`sudo nano /etc/apache2/mods-enabled/dir.conf`
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/order.jpg)

- I restarted Apache with `sudo systemctl restart apache2`


- The tooling website code should now be serving the domain name. I tested this by navigating to `http://192.168.1.130/login.php` as shown

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/testing.jpg)



- Hence i have implemented a tooling website solution and will be done using AWS EC2 instances later


 Credits:

 [DevOps with Darey](www.darey.io)
 
 [Install Git on Ubuntu](https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-20-04)
 
 [Install the Apache Web Server on Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04)