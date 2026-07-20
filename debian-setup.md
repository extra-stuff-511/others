df -h
umount /target/boot/efi
umount /target/boot
umount /target

mount /dev/mapper/partition_crypt /mnt
cd /mnt
ls
mv @rootfs @
ls
btrfs subvolume create @home
btrfs subvolume create @cache
btrfs subvolume create @log
btrfs subvolume create @libvirt
btrfs subvolume create @snapshots

mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@ /dev/mapper/partition_crypt /target

mkdir -p /target/home
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@home /dev/mapper/partition_crypt /target/home

mkdir -p /target/var/cache
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@cache /dev/mapper/partition_crypt /target/var/cache

mkdir -p /target/var/log
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@log /dev/mapper/partition_crypt /target/var/log

mkdir -p /target/var/lib/libvirt
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@libvirt /dev/mapper/partition_crypt /target/var/lib/libvirt

mkdir -p /target/.snapshots
mount -o noatime,ssd,compress=zstd,space_cache=v2,discard=async,subvol=@cache /dev/mapper/partition_crypt /target/.snapshots

mkdir -p /target/boot/efi
mount /dev/bootpart /target/boot
mount /dev/efipart /target/boot/efi

nano /target/etc/fstab