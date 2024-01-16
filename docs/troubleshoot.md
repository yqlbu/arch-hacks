# Troubleshooting

<!-- vim-markdown-toc GFM -->

* [Boot into existing partitions](#boot-into-existing-partitions)
  * [Open encrypted device (Optional)](#open-encrypted-device-optional)
  * [Mount partitions](#mount-partitions)
  * [Change root](#change-root)
  * [Inspect boot configuration](#inspect-boot-configuration)
  * [Regenerate initramfs](#regenerate-initramfs)
* [Restore from timeshift](#restore-from-timeshift)

<!-- vim-markdown-toc -->

## Boot into existing partitions

> [!NOTE]
> Must boot from Arch Live CD ISO

### Open encrypted device (Optional)

```bash
# open partition
cryptsetup luksOpen /dev/nvme0n1p2 root
# enter passphrase
# verify partitions
lsblk
```

### Mount partitions

```bash
# mount root partition
mount /dev/mapper/root /mnt -o subvol=@
# mount home partition
mount /dev/mapper/root /mnt/home -o subvol=@home
# mount boot partition
mount /dev/nvme0n1p1 /mnt/boot

# verify
lsblk
```

### Change root

```bash
arch-chroot /mnt
```

### Inspect boot configuration

```bash
ls /boot
```

configuration profiles are located in `/boot/loader` and `/boot/loader/entries`

### Regenerate initramfs

```bash
mkinitcpio -p linux
```

## Restore from timeshift

Reference: <https://forum.endeavouros.com/t/system-not-bootable-after-restore-timeshift-on-btrfs/24184/18>

After chrooting into the partition, do the following:

```bash
journalctl -b -0
sudo timeshift --restore
```

Then

```bash
exit
sudo umount -l /mnt
sudo reboot
```
