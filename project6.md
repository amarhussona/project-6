# Project 6: Web Solution With WordPress

## Prepare a Web server:

Launch an EC2 instance that will serve as the Web Server. Using Red Hat for this Project.
Create 3 volumes for the Web Server EC2 where each is 10 GiB.

[create volume](./images/01_create_volume.png)

Attach all three volumes one by one to your Web Server EC2 instance

[attach volume](./images/02_attach_volume.png)

Confirm storage on the EC2 dashboard:

[storage](./images/03_storage.png)

Go to the Linux terminal and use `lsblk` to also confirm:

[lsbulk](./images/04_lsblk.png)

Use `df -h` command to see all mounts and free space on your server:

[df -h](./images/04_df_h.png)

Use gdisk utility to create a single partition on each of the 3 disks:

`sudo gdisk /dev/xvdf`

[gdisk](./images/05_create_partition.png)

Note: inserting the n command to create the partition is missing in the documentation

Do the same for xvdg and xvdf

Use `lsblk` utility to view the newly configured partition on each of the 3 disks:

[lsblk confirm](./images/07_lsblk_new.png)

Install lvm2 package using `sudo yum install lvm2`:

[install lvm2](./images/06_install_lvm2.png)
[lvmdiskscan](./images/08_lvmdiskscan.png)

Use pvcreate utility to mark each of 3 disks as physical volumes to be used by LVM using these codes:

`sudo pvcreate /dev/xvdf1`
`sudo pvcreate /dev/xvdg1`
`sudo pvcreate /dev/xvdh1`

[pvcreate](./images/09_pvcreate.png)

Verify that the Physical volume has been created successfully by running `sudo pvs`:

[pvs](./images/10_verify_phy.png)

Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg:

`sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1`

[verify group](./images/11_volumegroup.png)

Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size:

`sudo lvcreate -n apps-lv -L 14G webdata-vg`
`sudo lvcreate -n logs-lv -L 14G webdata-vg`

[verify 2 logical volumes](./images/12_twologicalvolumes.png)

Verify the entire setup:

`sudo vgdisplay -v #view complete setup - VG, PV, and LV`
`sudo lsblk` 

[verify setup](./images/12_verify_logic_volumes.png)

Use mkfs.ext4 to format the logical volumes with ext4 filesystem:

`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`
`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`

[ext4](./images/13_format_to_ext4.png)

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

[back up files](./images/14_backupfiles.png)

[restore](./images/15_restore.png)




