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
