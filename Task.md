### Web Solution With WordPress
As a DevOps engineer you will most probably encounter PHP-based solutions, In 2021, it is the dominant web programming language used by more websites than any other programming language.
In this project we are going to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress.
WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).
The project consist of two(2) parts:
1. Configure storage subsystem for Web and Database servers based on Linux OS. The purpose of this part is to know how to work with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part is to deploy web and DB tiers of Web Solution

# Three-tier Architecture 
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture. Which are:
1. Presentation Layer (PL): This is the user interface such as the client server or browser on your laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver
3. Data Access or Management Layer (DAL): This is the layer for computer data storage and data access. Database Server or File System Server such as FTP server, or NFS Server
In this project, we will have hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as *gdisk* and *LVM* respectively.

# prerequisite for this project
1. A Laptop or PC to serve as a client
2. An EC2 Linux Server as a web server (This is where you will install WordPress)
3. An EC2 Linux server as a database (DB) server
4. We also going to use *RedHat* OS for this Project
   
# Step By Step Procedure:
1. Spin up an EC2 instance on AWS using the *RedHat* OS one named *Web Server* and the second named *DB Server* ![reference image](/Pictures/pic1.PNG)
2. Create  6 volumes in the same AZ as your instances of 10GiB. ![reference image](/Pictures/pic2.PNG)
3. Attach the Three(3) volumes one by one to each of the instances 
4. You can start with any instance of your choice though I started with the *Web Server* instance
5. Open up your Linux terminal to beging configuration ![reference image](/Pictures/pic3.PNG)
6. Use *lsblk* command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with *ls /dev/* and make sure you see all 3 newly created block devices there - their names will likely be *xvdb, xvdc, xvdd*. and you should see this ![reference image](/Pictures/pic5.PNG) and ![reference image](/Pictures/pic4.PNG)
7. Use *df -h* command to see all mounts and free space on your server ![reference image](/Pictures/pic6.PNG)
8. Use gdisk utility to create a single partition on each of the 3 disks follow the prompt in the picture ![reference image](/Pictures/pic7.PNG) n= stands for new partition, p= stands for view partition, w= stands for write partition mean confirm all the changes you made. Follow same procedure for the remaining disks.
9. Use *lsblk* utility to view the newly configured partition on each of the 3 disks. ![reference image](/Pictures/pic8.PNG)
10. Install *lvm2* package using sudo yum install lvm2. ![reference image](/Pictures/pic9.PNG)
11. Run sudo *lvmdiskscan* command to check for available partitions and Use *pvcreate* utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM. ![reference image](/Pictures/pic10.PNG)
12. Verify that your Physical volume has been created successfully by running *sudo pvs* ![reference image](/Pictures/pic11.PNG) 
13. Use *vgcreate* utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg and Verify that your VG has been created successfully by running *sudo vgs* ![reference image](/Pictures/pic12.PNG)
14. Use *lvcreate* utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size and Verify that your Logical Volume has been created successfully by running *sudo lvs* ![reference image](/Pictures/pic13.PNG)
15. Verify the entire setup ![reference image](/Pictures/pic14.PNG) and ![reference image](/Pictures/pic15.PNG)
16. Use *mkfs.ext4* to format the logical volumes with ext4 filesystem ![reference image](/Pictures/pic16.PNG)
17. Create /var/www/html directory to store website files, Create /home/recovery/logs to store backup of log data, Mount /var/www/html on apps-lv logical volume then finally Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system) you should see this ![reference image](/Pictures/pic17.PNG)
18. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why the above step was very important) then Restore log files back into /var/log directory ![reference image](/Pictures/pic18.PNG)
19. Update /etc/fstab file so that the mount configuration will persist after restart of the server but we need the UUID (Universerly Unique Identifier) to do then and to get the UUID use *sudo blkid* ![reference image](/Pictures/pic19.PNG)
20. sudo *vi /etc/fstab* then  Update */etc/fstab* in this format using your own UUID and rememeber to remove the leading and ending quotes.![reference image](/Pictures/pic20.PNG)
21. Test the configuration and reload the daemon using *sudo mount -a* and *sudo systemctl daemon-reload* then Verify your setup by running *df -h*, output must look like this ![reference image](/Pictures/pic21.PNG) and this ![reference image](/Pictures/pic22.PNG)
22. Now repeat the same for your Database Server but instead of *apps-lv* create *db-lv* and mount it to */db* directory instead of */var/www/html/*.

# Part Two 1. (Installing WordPress on Web Server EC2)
1. Update the repository *sudo yum -y update* ![reference image](/Pictures/pic26.PNG)
2. Install wget, Apache and it's dependencies *sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json* ![reference image](/Pictures/pic27.PNG)
3. Start Apache *sudo systemctl enable httpd* and  *sudo systemctl start httpd*
# 2. Now install PHP and it's dependencies
1. *sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm*  ![reference image](/Pictures/pic28.PNG)
2. *sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm* ![reference image](/Pictures/pic29.PNG)
3. *sudo yum module list php* ![reference image](/Pictures/pic30.PNG)
4. *sudo yum module reset php* ![reference image](/Pictures/pic31.PNG)
5. *sudo yum module enable php:remi-7.4*
6. *sudo yum install php php-opcache php-gd php-curl php-mysqlnd*
7. *sudo systemctl start php-fpm*
8. *sudo systemctl enable php-fpm*
9. *setsebool -P httpd_execmem 1*
10. *sudo systemctl restart httpd*

# 3. Download wordpress and copy wordpress to */var/www/html*
1. *mkdir wordpress* *cd wordpress* *sudo wget http://wordpress.org/latest.tar.gz* ![reference image](/Pictures/pic32.PNG) 
2. Then we unzip *sudo tar -xzvf latest.tar.gz* ![reference image](/Pictures/pic33.PNG)
3. then remove the zip file *sudo rm -rf latest.tar.gz* ![reference image](/Pictures/pic34.PNG)
4. then copy *cp wordpress/wp-config-sample.php wordpress/wp-config.php*
5. move the wordpress file to /var/www/html/ the remove the *wp-config.php* file *cp -R wordpress /var/www/html/* and *rm wp-config.php*

# 4. Configure Security Enhanced Linux(SELinux) Policies
SELinux is a mandatory access control mechanism, a higher level of access control than Linux's discretionary access control. SELinux acts under the least-privilege model. SELinux only grants access if the administrator writes a specific policy to do so.
1. *sudo chown -R apache:apache /var/www/html/wordpress*
2. *sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R*
3. *sudo setsebool -P httpd_can_network_connect=1*

# 5. Install MySQL on your DB Server EC2
1. *sudo yum update* and *sudo yum install mysql-server* ![reference image](/Pictures/pic35.PNG), ![reference image](/Pictures/pic36.PNG)
2. Verify that the service is up and running by using *sudo systemctl status mysqld*, if it is not running, restart the service and enable it so it will be running even after reboot using *sudo systemctl restart mysqld* and  *sudo systemctl enable mysqld* 
3.  Configure DB to work with WordPress using the following commands ![reference image](/Pictures/pic38.PNG) and you should see this ![reference image](/Pictures/pic39.PNG)
4.  Configure WordPress to connect to remote database.
5.  Make sure to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule configuration specify source as /32 ![reference image](/Pictures/pic40.PNG)
6.  Install MySQL client and test that you can connect from your Web Server to your DB server by using *mysql-client*
7.  install mysql *sudo yum install mysql*
8.  connect from your web server to your DB server *sudo mysql -u admin -p -h <DB-Server-Private-IP-address>* and if you can see this ![reference image](/Pictures/pic41.PNG) then Enable TCP port 80 in Inbound Rules configuration for your Web Server EC2 (enable from everywhere 0.0.0.0/0 or from your workstation's IP) ![reference image](/Pictures/pic42.PNG)
9.  Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/ ![reference image](/Pictures/pic46.PNG)
10. Fill out your DB credentials ![reference image](/Pictures/pic47.PNG)
11. If you see this ![reference image](/Pictures/pic48.PNG) then it means your WordPress has successfully connected to your remote MySQL database
12. After you login in you should see this ![reference image](/Pictures/pic45.PNG) and ![reference image](/Pictures/pic44.PNG)


## It Was A Really A Long Ride and I hope You Enjoyed It. Don't Hesitate To Ask Me Any Question Where Ever I Get Lost! THANK YOU
