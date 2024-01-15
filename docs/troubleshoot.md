# Troubleshooting

<!-- vim-markdown-toc GFM -->

* [Boot to existing partitions](#boot-to-existing-partitions)
  * [Open encrypted device (Optional)](#open-encrypted-device-optional)
  * [Mount partitions](#mount-partitions)
  * [Change root](#change-root)
  * [Inspect boot configuration](#inspect-boot-configuration)
  * [Regenerate initramfs](#regenerate-initramfs)

<!-- vim-markdown-toc -->

## Boot to existing partitions

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
