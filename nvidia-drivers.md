
# Remove Nouveau

sudo apt purge xserver-xorg-video-nouveau

sudo tee /etc/modprobe.d/blacklist-nouveau.conf > /dev/null <<EOF
blacklist nouveau
options nouveau modeset=0
EOF

sudo update-initramfs -u -k all

grep GRUB_CMDLINE_LINUX /etc/default/grub
sudo update-grub

# Driver Installation

main contrib non-free non-free-firmware

sudo apt install nvidia-detect
nvidia-detect

sudo apt install linux-headers-$(uname -r) build-essential dkms

sudo apt install nvidia-driver firmware-misc-nonfree

ls /etc/modprobe.d/nvidia-blacklists-nouveau.conf
sudo update-initramfs -u

sudo nvim /etc/default/grub
nvidia-drm.modeset=1
sudo update-grub
sudo update-initramfs -u

sudo reboot

