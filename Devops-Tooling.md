**Devops Tooling Website Solution**


In this Project the set up of NFS shared folders to 3 Apache Apache Web Servers will be demonstrated.

This is caried out to provide a Devops Tooling Website Solution.

The NFS shared folders will store var/www/ and  httpd logs contents to the NFS server and the project is executed in Virtual Box.

The aim of this work is to understand the use of  NFS servers to store logs and shared folders in certain location.

**Network Information**

MySQL Server 192.168.1.132
Web Server1 192.168.1.223
Web Server2 192.168.1.130
Web Server3 192.168.1.294
NFS Server 192.168.1.131




**SETTING UP THE NFS SERVER**


I created physical volume,logical volumes and volume group as follows:
- ```Pvcreate /dev/sdb /dev/sdc /dev/sdd``` --- creating physical volumes
 
- ```lvcreate -l 100%FREE -n lv-apps vg-apps``` – to create logical volumes for app group using all the available space

- ```lvcreate -l 100%FREE -n lv-logs vg-logs``` – to create logical volumes for logs using all the available space
 
- ```lvcreate -l 100%FREE -n lv-opt vg-opt``` – to create logical volumes for opt on the volume group opt using all the available space

- ```Vgcreate vg-apps /dev/sdb``` – to create volume group for apps on the ```/dev/sdb```
 
- ```Vgcreate vg-logs /dev/sdc``` – to create volume group for logs on the ```/dev/sdc```
 
- ``` Vgcreate vg-opt /dev/sdd``` – to create volume group for opt on the ```/dev/sdd```

I created mount points for logs,html and opt on 
```/mnt``` directory for the logical volumes as follows

I created mount points directory for the logical volumes

- Mkdir /mnt/html – making directory for html
- Mkdir /mnt/logs – making directory for logs

I mounted them as shown below

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/mount.jpg)


I formatted the disks with the xfs file system as show below
mkfs.xfs /dev/vg-apps/lv-apps – first make it an xfs filesystem
mkfs.xfs /dev/vg-logs/lv-logs – first make it an xfs filesystem

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/xfs.png)


To ensure that all the mounts  persists after reboot i edit the /etc/fstab config as follows
![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/uuid.jpg)


Then i Check the mount.
```df -h```

![](https://github.com/drazen-dee28/Devops-Tooling-Website-Solution/blob/main/devops_tooling/output.png)



I changed the permision on the following folders as follows:
```chmod a+rwx /mnt/html/```
```chmod a+rwx /mnt/logs/```
```chmod a+rwx /mnt/opt/`

 

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


Then, i have the firewalld service running, I need to allow traffic to the necessary NFS services (mountd, nfs, rpc-bind) via the firewall, then reload the firewall rules to apply the changes, as follows.

firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-source=192.168.1.0/24
firewall-cmd --reload