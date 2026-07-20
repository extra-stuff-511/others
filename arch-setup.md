
# Network

iwctl station wlan0 connect <ssid>
<ssid password>

# Disk Partitioning

lsblk

gdisk /dev/disk
n > +1G > ef00 = efi Partition
n > +16G > 8200 = swap partition
n > remaining space > 8300 = root

# Partition Formatting

mkfs.fat -F32 /dev/efi-part
mkswap /dev/swap-part
cryptsetup luksFormat /dev/root-part
cryptsetup luksOpen /dev/root-part main
mkfs.btrfs /dev/mapper/main (root-part)

lsblk

# Btrfs subvolume creation

mount /dev/mapper/main /mnt
cd /mnt
btrfs subvolume create @
btrfs subvolume create @home
btrfs subvolume create @cache
btrfs subvolume create @log
btrfs subvolume create @libvirt
btrfs subvolumme create @snapshots
cd
pwd
umount /mnt

# Mounting

mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@ /dev/mapper/main /mnt
mkdir -p /mnt/home
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@home /dev/mapper/main /mnt/home
mkdir -p /mnt/var/cache
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@cache /dev/mapper/main /mnt/var/cache
mkdir -p /mnt/var/log
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@log /dev/mapper/main /mnt/var/log
mkdir -p /mnt/var/lib/libvirt
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@libvirt /dev/mapper/main /mnt/var/lib/libvirt
mkdir -p /mnt/.snapshots
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@snapshots /dev/mapper/main /mnt/.snapshots
mkdir -p /mnt/boot
mount /dev/efi-part /mnt/boot
swapon /dev/swap-part

lsblk

# Pacstrapping

pacstrap /mnt 
kernel - base linux-lts/zen linux-lts/zen-headers linux-firmware 
essentials - sudo nvim nano base-devel efibootmgr fastfetch git grub networkmanager ufw syncthing timeshift gvfs gvfs-mtp xdg-user-dirs upower(cosmic)
sound - pipewire pipewire-pulse pipewire-jack alsa-utils
bluetooth - bluez bluez-utils

# Fstab Generation

genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

# Chrooting

arch-chroot /mnt

# Timezone setup

ln -sf /usr/share/zoneinfo/continent/city /etc/localtime
hwclock --systohc

# Locale Setup

nvim /etc/locale.gen
locale-gen
echo "LANG=en_GB.UTF-8" >> /etc/locale.conf
echo "hostname" >> /etc/hostname

# User Setup

passwd
useradd -m -g users -G wheel user
Passwd user
echo "user ALL=(ALL) ALL" >> /etc/sudoers.d/user

# Mkinitcpio configuration

nvim /etc/mkinitcpio.conf
mkinitcpio -P

# Additional Packages

sudo pacman -S "additional packages"
terminal - alacritty, kitty, foot, ghostty
graphical desktop - gnome, cosmic, xfce4 xfce4-goodies, cinnamon
for plasma - plasma-desktop plasma-pa plasma-nm power-profiles-daemon powerdevil bluedevil isoimagewriter filelight gwenview ark dolphin spectacle haruna elisa filelight
display manager - ly, sddm, gdm, cosmic-greeter, lightdm

# Grub Setup

grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
blkid /dev/root >> /etc/default/grub
blkid /dev/swap >> /etc/default/grub
"cryptdevice=UUID=uuid:main root=/dev/mapper/main resume=UUID=uuid nvidia-drm.modeset=1"
grub-mkconfig -o /boot/grub/grub.cfg

# Enabling Services

systemctl enable "display-manager"
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable ufw
systemctl enable syncthing@user

# Rebooting

exit
reboot

# AUR Helper Setup

sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/paru.git
cd paru.git
makepkg -sri
cd
rm -rf paru .cargo
