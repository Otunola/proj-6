# proj-6   WEB SOLUTION WITH WORDPRESS
The project is comprised of 2 parts, 1 Configure storage subsystem for Web and Database servers based on Linux OS, 2, Install WordPress and connect it to a remote MySQL database server
# Launch 2 ec2 instances one as web server and the other as database
we create 3 EBS volumes with 10g each and attach it to the web-server ec2 instance as shown below
![Screen Shot 2022-09-24 at 1 55 09 PM](https://user-images.githubusercontent.com/112595648/192099133-5e52c3e5-9472-4473-9eb9-7627e81e7744.png)
use the command : lsblk 
to confirm the devices are attached and it is seen as shwon
<img width="1134" alt="Screen Shot 2022-09-24 at 2 00 47 PM" src="https://user-images.githubusercontent.com/112595648/192099361-bcbf5139-9715-4b86-85cd-01e0e7530846.png">
# Now create a logical volume from the EBS attached
the first thing we do is create a partition in each of the physical disks, then will switich to LVM (Logical volume management. from thre we creaate physical volumes, from the phisical volumes we shall create volume groups, then from the volume groups we create Logical volumes.
Logical volumes are we webservers abd databse servers work with
Use gdisk utility to create a single partition on each of the 3 disks
with the comand : sudo gdisk /dev/xvdf
as shown below
<img width="578" alt="Screen Shot 2022-09-24 at 2 46 36 PM" src="https://user-images.githubusercontent.com/112595648/192101440-b40772c8-44fe-41a2-888f-8d7880e43c4d.png">
same was done for xvdg and xvdh with commands
sudo gdisk /dev/xvdg

sudo gdisk /dev/xvdh
# Use lsblk utility to view the newly configured partition on each of the 3 disks.
command : lsblk
<img width="597" alt="Screen Shot 2022-09-24 at 2 58 00 PM" src="https://user-images.githubusercontent.com/112595648/192101923-8c2f6c79-64a1-40e6-8fbb-f05c56074a67.png">

then we need to Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.
now scan for the available partitions with the commad : sudo lvmdiskscan
<img width="581" alt="Screen Shot 2022-09-24 at 3 01 16 PM" src="https://user-images.githubusercontent.com/112595648/192102088-9abb17e4-5e0d-43e4-8dca-22846a597614.png">
as seen above, the 3 partitions are present.
# Create Physical Volumes to be used as LVM
we use the command pvcreate as on each of the partitions as shown below
sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1
![image](https://user-images.githubusercontent.com/112595648/192102314-3f86f5b2-6ba5-4695-a5a1-c04d2fdd4f7d.png)
# Verify the physical volumes are created with command 
: sudo pvs
as shown
<img width="1403" alt="Screen Shot 2022-09-24 at 3 09 02 PM" src="https://user-images.githubusercontent.com/112595648/192102498-55ddb9a1-32a7-4875-90db-615b8530f046.png">
# Add all the 3 Physical Volumes to create a volume group
This can be achieved with vgcreate utility. let name the volume group webdata-vg
: sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
# NOw create Logical Volumes from the created volume groups
from the manual, we create 2 logical volumes of 14g each with the commad as shown
: 
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
# verify the entire setup
with command : lsblk
<img width="1376" alt="Screen Shot 2022-09-24 at 3 21 51 PM" src="https://user-images.githubusercontent.com/112595648/192103077-81f82395-7f0f-4b84-80fe-2df080e4c8a3.png">
: sudo vgdisplay -v 
<img width="1103" alt="Screen Shot 2022-09-24 at 3 22 51 PM" src="https://user-images.githubusercontent.com/112595648/192103115-771b844a-0044-4ee3-a1b3-a3b515cf425c.png">
#  Format the logical volumes with ext4 filesystem
this is achieved with the command 
:
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv

# Create /var/www/html directory to store website files
this is achieved with command : sudo mkdir -p /var/www/html
#  Create /home/recovery/logs to store backup of log data
This is achieved with the command : sudo mkdir -p /home/recovery/logs
# Mount /var/www/html on apps-lv logical volume
This is achieved the command : sudo mount /dev/webdata-vg/apps-lv /var/www/html/
Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs
with the command : sudo rsync -av /var/log/. /home/recovery/logs/
# Mount /var/log on logs-lv logical volume.
this is achieved with the command : sudo mount /dev/webdata-vg/logs-lv /var/log
# Restore log files back into /var/log directory
sudo rsync -av /home/recovery/logs/. /var/log
# Check the mount
: df -h

<img width="1107" alt="Screen Shot 2022-09-24 at 10 37 52 PM" src="https://user-images.githubusercontent.com/112595648/192119588-8afb4574-7710-48e8-b269-79cf22089814.png">
# Update /etc/fstab file so that even after restart of the server, the configuration and mount will stay
This is done by updating the UUID of the lological volumes shown below in the /etc/fstab as shown
run the command : sudo blkid
<img width="1031" alt="Screen Shot 2022-09-24 at 11 08 24 PM" src="https://user-images.githubusercontent.com/112595648/192120293-7c25f0ed-abf9-42ff-9d96-716fb66b7ffb.png">
then copy the UUID of the two logical volumes and update them on the /etc/fstab as shown
: run vi /etc/fstab
<img width="1093" alt="Screen Shot 2022-09-24 at 11 10 03 PM" src="https://user-images.githubusercontent.com/112595648/192120323-19a8efd9-a5ed-400c-9a36-cbb0b3a493d8.png">

Will test the coniguration with commnd
sudo mount -a
which should show no error
df -h
<img width="998" alt="Screen Shot 2022-09-24 at 11 18 43 PM" src="https://user-images.githubusercontent.com/112595648/192120548-355383b6-4959-4ac9-a9c1-bc7bd202b297.png">

# Prepare the database server
repeats steps done on the web-server, by creating the 3 addtional volumes, creating physical volumes, ceating volume groups and creating logical volumes. but this time on the db-server only one logical volume is created and allocated a 20g.
logical volume named db-lv
Now we creat a mount point with command : sudo mkdir /db
# MAke a file system before mounting
this is achieved with the comand: sudo mkfs.ext4 /dev/vg-database/db-lv
# now mount with the command below
: sudo mount /dev/vg-database/db-lv /db
confirm the mount with comand : df -h
<img width="1019" alt="Screen Shot 2022-09-24 at 11 53 21 PM" src="https://user-images.githubusercontent.com/112595648/192121320-bb6de76f-dd5c-48ff-8550-647b8ddd98d0.png">

make the mount persistent by putting the UUID in the /etc/fstab as shown below
<img width="1119" alt="Screen Shot 2022-09-24 at 11 56 44 PM" src="https://user-images.githubusercontent.com/112595648/192121398-39dd73ff-f3c3-43ce-874a-e6e9bec7fa05.png">

<img width="462" alt="Screen Shot 2022-09-25 at 12 02 07 AM" src="https://user-images.githubusercontent.com/112595648/192121531-531470cf-2fe3-4ce1-8f2c-acf0b89effd4.png">

test the config with command : sudo mount -a
# We now need to install wordpress on the EC2 server
first we update the repository with command : sudo yum -y update
2. install twget, apache and all its dependancies 
3. : sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
4. start apache with commands
5. 
sudo systemctl enable httpd
sudo systemctl start httpd
INstall php and all its dependancies
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php
sudo yum module reset php
sudo yum module enable php:remi-7.4
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
setsebool -P httpd_execmem 1
