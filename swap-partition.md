
lsblk

sudo mkswap --
sudo swapon --

sudo blkid --

sudo nano /etc/fstab
UUID=<uuid>  none  swap  sw  0  0

sudo sysctl vm.swappiness=10
sudo nano /etc/sysctl.d/99-swappiness.conf
vm.swappiness=10

sudo nano /etc/default/grub
resume=UUID=
sudo update-grub

sudo apt install initramfs-tools
sudo nano /etc/initramfs-tools/conf.d/resume
resume=UUID=
sudo update-initramfs -u -k all

lsinitramfs /boot/initrd.img-$(uname -r) | grep resume
