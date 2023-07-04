# Project7-Tooling

# Devops_Tooling_website_Solution
Set of tools that will help a DevOps team in day to day activities in managing, developing, testing, deploying and monitoring different project.
This is very essential to the team of developers

## **ARCHTECTURAL DESIGN**

![artectureimage](https://user-images.githubusercontent.com/77943759/224522484-5943e969-42ab-4acf-963c-0d4f4a976a2f.png)


## **NFS SERVER**

Create a new **Redhat 8 ec2 linux instance** and save the keypair as a .pem file for connection for Linux/Windows Terminal

![ec2instance](https://user-images.githubusercontent.com/77943759/224520612-6b13769a-642c-49e1-98ac-6393edd7aa9e.png)

Create 3 volumes of 10G each

![creatvol](https://user-images.githubusercontent.com/77943759/224521152-3f8c4ced-a40f-40a7-aa5f-7be0f320140f.png)

Attach the volumes to the instance one after another

![volattach](https://user-images.githubusercontent.com/77943759/224521179-d23c8a5a-9907-4a4a-99fe-f5d967fac236.png)


Open the linux/windows terminal and checked the attached disks. Run:

`lsblk`

![lsblk](https://user-images.githubusercontent.com/77943759/224521225-600ebfd0-433f-4af2-bf46-98bfd26eaab2.png)

Create Partitions on each disk. Run

```
sudo gdisk /dev/xvdg
sudo gdisk /dev/xvdf
sudo gdisk /dev/xvdh
```
Type n, to create new partition, enter 1 to create 1 partition, p to see the partition details and w to write the created partition. Select yes to finish

![sudogdisk](https://user-images.githubusercontent.com/77943759/224521362-01ee744d-b106-4194-a6e4-2fa73d83709b.png)


Check the Partitions created

`lsblk`

![lsblk2](https://user-images.githubusercontent.com/77943759/224521332-e683b559-98e7-44b8-81e7-a96cb6456eb8.png)

Intall lvm2

`sudo yum install lvm2`

![sudolvm](https://user-images.githubusercontent.com/77943759/224521656-c91eac45-e385-4a15-a504-5a79ef5e1fb3.png)


Check the available partitions. Run

`sudo lvmdiskscan`

![scandiklvm](https://user-images.githubusercontent.com/77943759/224521584-a29874a2-90fe-4bd5-8e46-d4a12d5780d2.png)

Create Physical volume by marking 3 of the partitioned disks with pvcreate

```
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
```
![pvcreate](https://user-images.githubusercontent.com/77943759/224521747-f8b496be-02ef-4d9e-986f-bd2f802cc85d.png)

Verify the created physical volumes

`sudo pvs`

![sudopvs](https://user-images.githubusercontent.com/77943759/224521787-b5d9ff91-75ea-49d6-a4b5-c19f127ee66c.png)

Create a volume group to contain all 3 of the created physical volumes with vgcreate. In this case the name is nfs-vg

`sudo vgcreate nfs-vg /dev/xvdf1 /dev/xvdg1 /dev/xvdh1`

![vgnfs](https://user-images.githubusercontent.com/77943759/224522159-09b3b945-97cc-4b06-9891-8131540b2eea.png)


Verify the created volume group

`sudo vgs`

![sudovgs](https://user-images.githubusercontent.com/77943759/224522171-99dec4fd-8435-4d1c-b878-b8e48464d59a.png)

Create 3 Logical Volumes. lv-opt lv-apps, and lv-logs using the lvcreate utility. Allocate 9G each to them. The lv-apps will store website data, lv-gos will store web logs.

```
sudo lvcreate -n lv-apps -L 9G nfs-vg
sudo lvcreate -n lv-logs -L 9G nfs-vg
sudo lvcreate -n lv-opt -L 9G nfs-vg
```

![lvcreate](https://user-images.githubusercontent.com/77943759/224522281-8fdbac71-a35e-4fe8-affe-3d366e1de2d0.png)

Confirm the logical volumes

`sudo lvs`

![sudolvs](https://user-images.githubusercontent.com/77943759/224522392-aa13bf6d-f174-416b-a8e6-a9381364d7b9.png)

Verify the entire set-up

`lsblk`

![lsblk3](https://user-images.githubusercontent.com/77943759/224522437-8e2672fe-a136-4944-b978-46968f6b7325.png)

Format the disk logical volumes with mfks.xfs

```
sudo mkfs -t xfs /dev/nfs-vg/lv-apps
sudo mkfs -t xfs /dev/nfs-vg/lv-logs
sudo mkfs -t xfs /dev/nfs-vg/lv-opt
```
Create /mnt/apps directory to store files
`sudo mkdir -p /mnt/apps` for website files
`sudo mkdir -p /mnt/logs` for log files
`sudo mkdir -p /mnt/opt` to be used for jedkins in the next project
Mount lv-apps of /mnt/apps; lv-logs on /mnt/log and lv-opt on /mnt/opt

```
sudo mount /dev/nfs-vg/lv-apps /mnt/apps
sudo mount /dev/nfs-vg/lv-logs /mnt/logs
sudo mount /dev/nfs-vg/lv-opt /mnt/opt
```

![mkdirmount](https://user-images.githubusercontent.com/77943759/224875899-040afda0-058b-45ee-81bd-a495f4bc3a07.png)

Update /etc/fstab file

Run

`sudo blkid`

![blkid](https://user-images.githubusercontent.com/77943759/224877314-a564da11-3d94-4447-a015-c3a11cb26026.png)

coppy the mount ids and update:

`sudo vi /etc/fstab`

![fstabedit](https://user-images.githubusercontent.com/77943759/224877704-b0dfd171-0df1-4c68-a44b-eb1813c78778.png)


Test the configuration and reload the daemon

```
sudo mount -a
sudo systemctl daemon-reload
```

Install nfs server, configure and make sure it starts on system reboot

```
sudo yum -y update
sudo yum install nfs-utils -y
sudo systemctl start nfs-server.service
sudo systemctl enable nfs-server.service
sudo systemctl status nfs-server.service
```

![utils](https://user-images.githubusercontent.com/77943759/224878999-0904d9b1-ae87-4744-91c4-7ae2d21e5f6f.png)

![enablenfs](https://user-images.githubusercontent.com/77943759/224879373-bc69422b-c8a4-4b62-8233-e35399443277.png)

set up permission that will allow our Web servers to read, write and execute files on NFS:

```
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt
sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service
```
![own775](https://user-images.githubusercontent.com/77943759/224882245-f1ac09bb-4af7-4b7c-8be4-2a03dfafb962.png)

Check your subnet cidr – open your EC2 details in AWS web console and locate ‘Networking’ tab and open a Subnet link:

![Screenshot from 2023-03-14 03-56-10](https://user-images.githubusercontent.com/77943759/224881295-3b81e8cd-197b-4b08-ac66-e2b55d3c33b1.png)

Configure access to NFS for clients within the same subnet using the subnet cidr we got above

```
sudo vi /etc/exports

/mnt/apps <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/logs <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)
/mnt/opt <Subnet-CIDR>(rw,sync,no_all_squash,no_root_squash)

Esc + :wq!

sudo exportfs -arv
```

![etcedit](https://user-images.githubusercontent.com/77943759/224882144-f04764fa-06e1-4cbf-b159-cfd7a4ab59cf.png)

![exportfs](https://user-images.githubusercontent.com/77943759/224882420-cabaab64-b0d9-4e34-a523-65371f7b941d.png)

Check which port is used by NFS and open its port in security group setting

`rpcinfo -p | grep nfs`

![portcheck](https://user-images.githubusercontent.com/77943759/224882769-dcffe856-3b6a-48c7-a20f-4c2472822be5.png)

Open port TCP 2049 inbound security group and also, In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049

![tcpudpport](https://user-images.githubusercontent.com/77943759/224883015-48163513-c2d5-4eea-bae0-c2a670441a22.png)

## **CONFIGURE DATABASE**

Configure ec2 instance type ubuntu and do these:

Install MySQL server

`sudo apt install mysql-server`

Create a database and name it tooling

Create a database user and name it webaccess

Grant permission to webaccess user on tooling database to do anything only from the webservers subnet cidr

![realDB](https://user-images.githubusercontent.com/77943759/224884374-fe7362a1-50a3-4be3-82ae-824d005a3995.png)

Dont forget to edit bind address for database to 0.0.0.0 so our DB can be accessible for our servers

`sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf`

![bind address](https://user-images.githubusercontent.com/77943759/224891332-310a978c-dace-4b74-ac32-7e9f4438398f.png)



## **WEBSERVERS**

Spin up a new REHL 8 ec2 instance in the same subnet as the nfs server

Install NFS client

`sudo yum install nfs-utils nfs4-acl-tools -y`

Mount /var/www/ and target the NFS server’s export for apps

```
sudo mkdir /var/www
sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www
```
![mkdirndmount](https://user-images.githubusercontent.com/77943759/224885839-add4846d-ad37-441e-a930-7c1a6eb6d9fe.png)

Use df -h to confirm the mount

![mountconfirm](https://user-images.githubusercontent.com/77943759/224886274-b51e4165-ec49-4ad9-b72e-f8f1df6a7a94.png)

Edit /etc/fstab file

`sudo vi /etc/fstab`

Add the following to the file setting

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

![fstabwww](https://user-images.githubusercontent.com/77943759/224886812-8727819a-c474-46ca-a4e4-5cf7b76a6657.png)

Install Remi’s repository, Apache and PHP

```
sudo yum install httpd -y

sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm

sudo yum install http://rpms.remirepo.net/enterprise/remi-release-9.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

Sudo setsebool -P httpd_execmem 1
```

Repeat steps 1-5 for another 2 Web Servers.

Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. If you see the same files – it means NFS is mounted correctly.

![varwwwconfirm](https://user-images.githubusercontent.com/77943759/224887769-c3bd12c9-04d9-41eb-8908-86c7c0a28552.png)

In our webserver /var/www directory. Run

`sudo touch text.txt`

![touchtxt](https://user-images.githubusercontent.com/77943759/224888236-d4054a4e-8eb5-4516-9914-082bd71b0e72.png)

![touchconfirm](https://user-images.githubusercontent.com/77943759/224888293-7f07245f-9d38-4356-af09-381bc14f7322.png)

We can see the text.txt file created inside our nfs server /mnt/apps directory. So they are communicating perfectly.

Create directory for apache log and mount it on the /mnt/logs directory of our NFS server

```
sudo mkdir -p /var/log/httpd

sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/logs /var/log/httpd
```
![logsdirndmount](https://user-images.githubusercontent.com/77943759/224889232-a591f369-3916-4c56-8881-469d962db2f4.png)

Edit the /etc/fstab file so that it persists even after reboot

`sudo vi /etc/fstab`

![fstabupdate](https://user-images.githubusercontent.com/77943759/224889369-e6a31fbc-bc5d-4bbd-a0fd-30ec1b5a9fc8.png)

Fork the tooling source code from [Darey.io Github Account](https://github.com/darey-io/tooling) to your Github account.

Download git

`sudo yum install git`

Clone the repository you forked the project into

`git clone <repository link>`

Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to /var/www/html

```
cd tooling

sudo cp -r html/* /var/www/html/
```

Open TCP port 80 on the Web Server

If you encounter 403 Error:

check permissions on /var/www/html folder
Disable SELinux sudo setenforce 0
To make this change permanent, open selinux config file and set SELINUX=disabled then restrt httpd.

`sudo vi /etc/sysconfig/selinux`

Update the website’s configuration to connect to the database

`sudo vi /var/www/html/functions.php`

![newphpconfig](https://user-images.githubusercontent.com/77943759/224892277-bf9ced13-f3c3-477b-87c6-05e12f7254f8.png)


Apply tooling-db.sql script to your database using this command:

`mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

Now we can open our web browser and enter

`<webserver-public-ip>`

![loginpage](https://user-images.githubusercontent.com/77943759/225014270-73677c67-5571-4c3f-b2d2-efaa4cfd0d5c.png)


![Screenshot from 2023-03-14 14-18-32](https://user-images.githubusercontent.com/77943759/225014338-68d07cc2-58bb-4c81-9f31-df502ee83db3.png)

We have just successfully implemented a web solution for a DevOps team using LAMP stack with remote Database and NFS servers.
