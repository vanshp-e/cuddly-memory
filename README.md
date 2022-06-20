# Cuddly-Memory

![](res/drawing.svg)

## Nvidia container/docker problem with ubuntu21.10/pop21.10

Error:
```
Running hook #0:: error running hook: exit status 1, stdout: , stderr: nvidia-container-cli: container error: cgroup subsystem devices not found: unknown.
```

Ref: https://github.com/NVIDIA/nvidia-docker/issues/1447#issuecomment-989024636
```
# Add experimental repo to /etc/apt/sources.list.d/libnvidia-container.list
# deb https://nvidia.github.io/libnvidia-container/experimental/ubuntu18.04/$(ARCH)
sudo editor /etc/apt/sources.list.d/libnvidia-container.list
```

For pop:

Ref: https://gist.github.com/kuang-da/2796a792ced96deaf466fdfb7651aa2e

```
# Assume you already got /etc/apt/sources.list.d/libnvidia-container.list setup
# you also need to change some priority for apt
# create a file under /etc/apt/preferences.d or modify existing one add
Package: *
Pin: origin nvidia.github.io
Pin-Priority: 1002 # Greater priority than pop own repo
```

For both: upgrade the packages.

```
sudo apt update
sudo apt upgrade
```

## Btrfs cause disk to be completely stalled (even for SSD)

Probably related to btrfs quota, you can disable it, but will lose some features.

```
btrfs quota disable /
```

If you use `timeshift` you also need to disable `Settings > Users > Enable BTRFS qgroups` too, since `timeshift` seems to switch it back on later.

## Restore from btrfs snapshot broke docker

Docker use btrfs as storage driver when the backing filesystem is btrfs by default. This mean docker will use btrfs subvolumes for stuffs. When we restore from snapshot at `/` all the subvolumes that was created by docker will be gone (from docker perspective, those subvolumes still exist in btrfs filesystems). To save ourself from headache, we can switch to overlay2 which use overlayfs instead.

Add this to `/etc/docker/daemon.json`.

```
{
    "storage-driver": "overlay2",
    ...
}
```

# Alder lake cpu cause pulseaudio to crash in kernel below 5.16

Seems to be a problem with driver or something.

Add `i915.force_probe` to `GRUB_CMDLINE_LINUX_DEFAULT` at `/etc/default/grub`

*You can do something like this to get the number and set it `sudo dmesg | grep 'not support' -A4`. `i915.force_probe=$SOME_NUMBER`*

# SSH connection stuck (the remote server killing the connection)

Use `Enter`, `~`, `.` in a sequence to disconnect the connection.

# `/dev/shm` is not big enough even though I got a lot of ram

Use remount to remount `/dev/shm`, `sudo mount -o remount,size=4G /dev/shm`.

# Upgrade to Ubuntu 22.04 make system unbootable

This is the following error:

```
error: out of memory.

Press any key to continue...
```

Get a linux bootable usb. It's a bug take a look at this reply https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1842320/comments/44.

However, do this https://bugs.launchpad.net/ubuntu/+source/linux/+bug/1842320/comments/41 step before running `update-initramfs -u -k all`.

```bash
sudo su -
mkdir root
mkdir boot
mkdir efi
cryptsetup luksOpen /dev/nvme0n1p3 nvme0n1p3_crypt #open encrypted root partition
mount /dev/mapper/vgubuntu-root ./root #change mapper to your root partition
mount /dev/nvme01p1 ./efi #change /dev/nvme01p1 to ur own efi partition
mount /dev/nvme01p2 ./boot #change /dev/nvme01p2 to ur own boot partition
mount -B ./boot ./root/boot
mount -B ./efi ./root/boot/efi
mount -B ./dev ./root/boot/dev
mount -B ./dev/pts ./root/boot/dev/pts
mount -B ./proc ./root/proc
mount -B ./sys ./root/sys
chroot ./root
# edit /usr/sbin/mkinitramfs first replace zstd) compress="zstd -q -19 -T0" ;;
# with zstd) compress="zstd -q -1 -T0" ;;
update-initramfs -u -k all
update-grub
exit
reboot
```

After that you should be able to boot into system.