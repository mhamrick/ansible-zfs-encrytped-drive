#Creating an encrypted ZFS data set that automatically unlocks and #mounts on boot

 

#Creating the Zpool

 

sudo bash

#Root privileges are required.

 

cd /etc

#Use /etc, so the drivekey file can be read at boot.

 

touch drivekey.txt

#Create the drivekey file.

 

dd if=/dev/urandom bs=32 count=1 of=drivekey.txt

#Populate the drivekey file.

#Backup key file (whole file, not text) see:
#ZFS load key issue



 

apt install zfsutils-linux

#Install ZFS tools.

#CHOOSE 1 of the options: 

zpool create DATA sda sdb #no redundancy 

zpool create DATA mirror sda sdb #mirrored

zpool create DATA mirror sda sdb cache nvme(partname)

#

# If SSD available, create zfs cache

#YMMV

#This process is system dependent. In this case, two drives (sda #and sdb) are being imported to the zpool. I used lsblk and #fdisk. DATA refers to the name of the zpool, you may name it #something else, if you’d like.

 

#Creating an encrypted ZFS dataset


zfs create -o encryption=on -o keyformat=raw -o keylocation=file:///etc/drivekey.txt DATA/edata


