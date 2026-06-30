---
title: Dasar-Dasar Instalasi Arch Linux
published: 2025-06-15
description: 'Panduan langkah demi langkah instalasi Arch Linux untuk pemula'
image: "https://upload.wikimedia.org/wikipedia/commons/a/af/Tux.png"
tags: [Linux, Arch, Tutorial]
category: 'Linux'
draft: false
---

## Apa Itu Arch Linux?

Arch Linux adalah distribusi Linux *rolling release* yang terkenal dengan filosofi **KISS** (*Keep It Simple, Stupid*). Berbeda dengan distro seperti Ubuntu atau Fedora, Arch memberikan kendali penuh kepada pengguna sejak awal instalasi.

## Prasyarat

Sebelum memulai, pastikan kamu memiliki:

- Koneksi internet yang stabil
- USB flash drive (minimal 2GB)
- Komputer dengan arsitektur x86_64 (64-bit)
- Cadangan data penting (instalasi akan memformat硬盘)

## 1. Unduh dan Buat Media Instalasi

Pertama, unduh ISO Arch Linux dari situs resmi:

```bash
# Download ISO
wget https://archlinux.org/releng/releases/2025.06.01/torrent/archlinux-2025.06.01-x86_64.iso.torrent
```

Buat bootable USB menggunakan `dd` (di Linux):

```bash
lsblk
sudo dd if=archlinux-2025.06.01-x86_64.iso of=/dev/sdX bs=4M status=progress
sudo sync
```

> Ganti `/dev/sdX` dengan device USB kamu. **Hati-hati**, pastikan device yang benar!

## 2. Boot dari USB

Masukkan USB, boot ulang, dan masuk ke BIOS/UEFI (biasanya tekan `F2`, `F12`, `Del`, atau `Esc`). Atur boot order ke USB pertama, simpan, dan restart.

## 3. Atur Layout Keyboard

Secara default layout keyboard adalah US QWERTY. Untuk keyboard Indonesia:

```bash
loadkeys id
```

## 4. Verifikasi Boot Mode

```bash
ls /sys/firmware/efi/efivars
```

Jika perintah di atas menampilkan file, kamu boot dalam mode **UEFI**. Jika tidak, kamu boot dalam mode **BIOS/Legacy**.

## 5. Koneksi Internet

### Wi-Fi (menggunakan `iwctl`)

```bash
iwctl

# Di dalam iwctl:
device list                 # Lihat daftar perangkat Wi-Fi
station wlan0 scan         # Scan jaringan
station wlan0 get-networks # Lihat hasil scan
station wlan0 connect "NamaWiFi" # Masukkan password jika diminta
exit
```

### Ethernet

Biasanya langsung terhubung. Cek koneksi:

```bash
ping -c 3 archlinux.org
```

## 6. Partisi Disk

Gunakan `fdisk` atau `gdisk` untuk partisi. Contoh partisi untuk UEFI:

```bash
fdisk /dev/nvme0n1
```

Buat partisi:

| Partisi | Ukuran     | Tipe          | Kode  |
|---------|------------|---------------|-------|
| ESP     | 1GB        | EFI System    | ef00  |
| Root    | 40-80GB    | Linux x86-64  | 8300  |
| Home    | Sisa       | Linux x86-64  | 8300  |

## 7. Format Partisi

```bash
# Format EFI partition
mkfs.fat -F32 /dev/nvme0n1p1

# Format root partition (btrfs atau ext4)
mkfs.btrfs -f /dev/nvme0n1p2

# Format home partition
mkfs.ext4 /dev/nvme0n1p3
```

## 8. Mount Partisi

```bash
# Mount root
mount /dev/nvme0n1p2 /mnt

# Buat dan mount direktori untuk EFI (UEFI)
mount --mkdir /dev/nvme0n1p1 /mnt/boot

# Mount home
mount --mkdir /dev/nvme0n1p3 /mnt/home
```

## 9. Install Base System

```bash
pacstrap -K /mnt base base-devel linux linux-firmware vim sudo networkmanager
```

## 10. Generate fstab

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Cek hasilnya:

```bash
cat /mnt/etc/fstab
```

## 11. Chroot ke Sistem Baru

```bash
arch-chroot /mnt
```

## 12. Konfigurasi Waktu

```bash
ln -sf /usr/share/zoneinfo/Asia/Jakarta /etc/localtime
hwclock --systohc
```

## 13. Lokalisasi

Edit `/etc/locale.gen` dan uncomment:

```
id_ID.UTF-8 UTF-8
en_US.UTF-8 UTF-8
```

Generate locale:

```bash
locale-gen
```

Buat `/etc/locale.conf`:

```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Buat `/etc/vconsole.conf`:

```bash
echo "KEYMAP=id" > /etc/vconsole.conf
```

## 14. Hostname

```bash
echo "archlinux" > /etc/hostname
```

Edit `/etc/hosts`:

```bash
cat > /etc/hosts << EOF
127.0.0.1   localhost
::1         localhost
127.0.1.1   archlinux.localdomain archlinux
EOF
```

## 15. Password Root dan User Baru

```bash
# Set password root
passwd

# Buat user baru
useradd -m -G wheel,audio,video,storage -s /bin/bash namauser
passwd namauser
```

Aktifkan sudo dengan edit `/etc/sudoers`:

```bash
EDITOR=vim visudo
```

Uncomment baris: `%wheel ALL=(ALL:ALL) ALL`

## 16. Install Bootloader (GRUB)

### Untuk UEFI:

```bash
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

### Untuk BIOS/Legacy:

```bash
pacman -S grub
grub-install --target=i386-pc /dev/nvme0n1
grub-mkconfig -o /boot/grub/grub.cfg
```

## 17. Network Manager

Aktifkan NetworkManager agar koneksi internet otomatis:

```bash
systemctl enable NetworkManager
```

## 18. Reboot

```bash
exit
umount -R /mnt
reboot
```

Setelah reboot, cabut USB dan kamu akan masuk ke sistem Arch Linux baru!

:::tip[Selanjutnya?]
Setelah berhasil boot, install **display manager**, **desktop environment** (seperti KDE, GNOME, atau Hyprland), dan aplikasi favoritmu.
:::

:::warning
Instalasi Arch Linux membutuhkan ketelitian. Pastikan membaca setiap perintah sebelum menjalankannya. Jika ragu, cek [Arch Wiki](https://wiki.archlinux.org/) yang merupakan dokumentasi terbaik di dunia Linux.
:::

## Referensi

- [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
- [Arch Wiki](https://wiki.archlinux.org/)
