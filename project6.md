# Project 6: Web Solution With WordPress

## Prepare a Web server:

Launch an EC2 instance that will serve as the Web Server. Using Red Hat for this Project.
Create 3 volumes for the Web Server EC2 where each is 10 GiB.

![create volume](./images/01_create_volume.png)

Attach all three volumes one by one to your Web Server EC2 instance

![attach volume](./images/02_attach_volume.png)

Confirm storage on the EC2 dashboard:

![storage](./images/03_storage.png)

Go to the Linux terminal and use `lsblk` to also confirm:

![lsbulk](./images/04_lsblk.png)

Use `df -h` command to see all mounts and free space on your server:

![df -h](./images/04_df_h.png)

Use gdisk utility to create a single partition on each of the 3 disks:

`sudo gdisk /dev/xvdf`

![gdisk](./images/05_create_partition.png)

Note: inserting the n command to create the partition is missing in the documentation

Do the same for xvdg and xvdf

Use `lsblk` utility to view the newly configured partition on each of the 3 disks:

![lsblk confirm](./images/07_lsblk_new.png)

Install lvm2 package using `sudo yum install lvm2`:

![install lvm2](./images/06_install_lvm2.png)
![lvmdiskscan](./images/08_lvmdiskscan.png)

Use pvcreate utility to mark each of 3 disks as physical volumes to be used by LVM using these codes:

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

![pvcreate](./images/09_pvcreate.png)

Verify that the Physical volume has been created successfully by running `sudo pvs`:

![pvs](./images/10_verify_phy.png)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

![verify group](./images/11_volumegroup.png)

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size:

`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

![verify 2 logical volumes](./images/12_twologicalvolumes.png)

Verify the entire setup:

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`
`sudo lsblk` 

![verify setup](./images/12_verify_logic_volumes.png)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

![ext4](./images/13_format_to_ext4.png)

Create /var/www/html directory to store website files:

`sudo mkdir -p /var/www/html`

Create /home/recovery/logs to store backup of log data:

`sudo mkdir -p /home/recovery/logs`

Mount /var/www/html on apps-lv logical volume:

`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`

Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs

`sudo rsync -av /var/log/. /home/recovery/logs/`

Mount /var/log on logs-lv logical volume:

`sudo mount /dev/webdata-vg/logs-lv /var/log`

Restore log files back into /var/log directory:

`sudo rsync -av /home/recovery/logs/. /var/log`

![back up files](./images/14_backupfiles.png)

![restore](./images/15_restore.png)

## Update the `/etc/fstab` file

The UUID of the device will be used to update the /etc/fstab file:

`sudo blkid`

![UUID](./images/16_UUID.png)

`sudo vi /etc/fstab`

Update /etc/fstab in this format using the previous UUID and make sure to remove the leading and ending quotes:

![etc/fstab](./images/17_etc_fstab.png)

Test the configuration and reload the daemon:

`sudo mount -a`
`sudo systemctl daemon-reload`

Verify the setup by running `df -h`:

![verify_setup](./images/18_verify_setup.png)

Now Prepare the Database server:

Launch a second Red Hat EX2 instance and repeat the same steps for the Web Server previously but instead of apps-lv create db-lv and mount it /db directory instead of /var/www/html

Note: when I created the new instance, I had created the volumes first. So to make the same subset, I added the subset east-1a to the new instance to be able to attach the volumes instead of chossing default:

![step2](./images/19_step2_select_subset_1a.png)
![step2attach](./images/20_step2_attach_volumes.png)

Update the repo:

`sudo yum update`

![update](./images/21_yum_update.png)

Now install WordPress on Web Server EC2:

Install wget, Apache and it’s dependencies:

`sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json`

![install wget,apache](./images/22_install_wordpress_on_webserver.png)

Start Apache:

`sudo systemctl enable httpd`
`sudo systemctl start httpd`

![start apache](./images/223_start_apache.png)

Install PHP and it’s depemdencies:

`sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm`
`sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm`
`sudo yum module list php`
`sudo yum module reset php`
`sudo yum module enable php:remi-7.4`
`sudo yum install php php-opcache php-gd php-curl php-mysqlnd`
`sudo systemctl start php-fpm`
`sudo systemctl enable php-fpm`
`setsebool -P httpd_execmem 1`

![install php](./images/24.png)

Retart Apache:

`sudo systemctl restart httpd`

Download wordpress and copy wordpress to var/www/html:

`mkdir wordpress`
`cd   wordpress`
`sudo wget http://wordpress.org/latest.tar.gz`
`sudo tar xzvf latest.tar.gz`
`sudo rm -rf latest.tar.gz`
`cp wordpress/wp-config-sample.php wordpress/wp-config.php`
`cp -R wordpress /var/www/html/`

![download wordpress & cp into dir](./images/25_download_wordpress_and_cp_to_dir.png)

Configure SELinux Policies:

`sudo chown -R apache:apache /var/www/html/wordpress`
`sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R`
`sudo setsebool -P httpd_can_network_connect=1`

Now install MySQL on the DB Server EC2:

Make sure the repo is updated if you haven't already by `sudo yum update`

Install mysql:

`sudo yum install mysql-server`

![install mysql](./images/27_install_mysql.png)

Verify that the service is up and running by using `sudo systemctl status mysqld`

![verify mysql](./images/28_sql_status.png)

Restart the service and enable it so it will be running even after reboot:

![restart apache](./images/29_restart_apache.png)

Configure DB to work with WordPress using these:
```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```
Note: make use it is the correct webserver private IP address

![create db and user](./images/30_create_database_and_user.png)

Next configure WordPress to connect to remote database:

Note to not forget to open MySQL port 3306 on DB Server EC2

Edit inbound rules to allow access to the Database only from the  web server private IP address:

![inbound rules](./images/31_edit_inbound_rules.png)

Next install mysql client and test the connection from web server to database:

`sudo yum install mysql`
`sudo mysql -u myuser -p -h <DB-Server-Private-IP-address>`

Verify if you can successfully execute SHOW DATABASES; command and see a list of existing databases:

![connect and show database](./images/32_connect_to_user.png)

Change permissions and configuration so Apache could use WordPress (NOTE this is no longer viewable in the documentation which slowed down my process)

`sudo chown _R apache:apache /var/www/html/wordpress`
`sudo chcon -t httpd_sys_rw_content_t /var/www/wordpress -R`
`sudo setsebool -P httpd_can_network_connect=1`

Enable TCP port 80 in Inbound Rules configuration for the Web Server EC2:

![inbound rule](./images/33_inbound_rules_for_webserver.png)

Try to access from a browser the link to the WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![wordpress](./images/35_wordpress.png)
![wordpress2](./images/36_wordpress1.png)

Note to use this link to have latest php version:

https://www.tecmint.com/install-lamp-on-centos-8/

Done