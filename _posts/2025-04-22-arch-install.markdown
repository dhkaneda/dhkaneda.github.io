---
layout: post
title:  "Arch Guided Install: LVM on LUKS"
date:   2025-04-21 08:14:04 -0700
categories: arch
---

## Why Arch?

[2025-04-22: Coming Back to Linux with Arch](./2025-04-22-linux.md)

## Why another guide?

At first, I thought I did not need this guide for myself simply because the Arch Wiki is so well written and I got used to navigating it after installing Arch more than ten times over the years. Each time I installed Arch, I learned something new. Sometimes, it was because I made a mistake and got stuck. Other times, I was informed of a better configuration or a new package that improved my system either in terms of utility or security. This guided install is a culmination of all the reasons WHY I installed certain things, in which order, and with what configuration. If this guide helps you, then great! It at least serves as a reference for myself to save time in the future.

Before I begin, it might be helpful to know which machines I am targetting with this guide.

| Device                         | Price                    | Processor        | RAM    | Storage    |
|---------------------------------|--------------------------|------------------|--------|------------|
| **Lenovo ThinkPad x250**        | ~ $80 USD from Craigslist | Intel i5-5300U   | 8 GB   | 256 GB SSD |
| **HP EliteDesk 800 G2** (2x)    | ~ $80 USD each from eBay  | Intel i5-6500T   | 16 GB  | 512 GB SSD |

## LVM on LUKS

I am particularly interested in using **LVM on LUKS** for my Arch Linux installation. This allows me to encrypt my entire disk while also providing the flexibility of logical volume management. I'd like to practice system configurations similar to what I may see in enterprise settings and was made aware the importance of encryption and flexibility by devs like [Mischa van den Burg](https://www.youtube.com/watch?v=qboMuv9vSpQ). I was also recently made aware of other filesystem options like **Btrfs** and **ZFS**, and I am also very interested in using those in the future. At this time, my focus is on working with Kubernetes and need to get up and running as quickly as possible. I'll be revisiting this guide in the future to explore those options.

---

## Booting into the Arch Linux Installation Medium

### Create an installation medium on a USB Drive

Pick a USB Drive (8–128 GB):

- Your mileage may vary, but choose a reputable brand for fewer issues and hardware QA variance (SanDisk, Kingston, etc.).
- Size: At least 4 GB (Arch ISO is ~1 GB).

### ⚠️ Problem: Larger USB Drives

USB drives larger than 32 GB are quite common these days. I had somehow lost all of my sub-64 GB drives over the years and did not wait to wait for a new one to arrive. I only had a 64 GB drive that I had lying around.

I read that it's recommended to use a USB stick 32 GB or smaller because many systems—especially older ones—expect bootable USBs to use the FAT32 file system.

FAT32 has a maximum volume size of 32 GB (officially on Windows), and larger drives may be formatted as exFAT or NTFS, which are not always bootable or recognized by all firmware.

### Solution: Use Ventoy

**Instead of using Rufus, Etcher, or dd to flash the ISO.*

**Ventoy** handles large drives (64 GB, 128 GB+) without compatibility issues. Even if you're going to use a 32 GB USB drive, Ventoy is a great tool for creating bootable USB drives.

<details>
<summary> Why This Bypasses the 32 GB Limitation </summary>

1. No reliance on FAT32 for the bootable ISO itself
    - Ventoy handles booting via its own partition and tools, so the USB’s size or file system type doesn't block bootability. Even if the main partition is exFAT or NTFS, Ventoy can still boot the ISOs.

1. UEFI/BIOS compatibility built-in
    - Ventoy is designed to work with both legacy BIOS and modern UEFI systems, removing guesswork about formatting or partitioning.

1. No need to format for each new ISO
    Just drag and drop ISO files onto the USB. Want Arch, Ubuntu, and Kali on the same stick? Done.

</details>

### Ventoy Installation and using the Arch Linux ISO

1. **Download** the latest version of Ventoy from the [official website](https://www.ventoy.net/en/download.html).
1. **Download** the latest Arch Linux ISO from the [Arch Linux website](https://archlinux.org/download/).
1. Use this guide to get started with Ventoy: [Ventoy Quick Start](https://www.ventoy.net/en/doc_start.html).
1. **Drag and drop** it onto the Ventoy USB—no flashing required.

### Booting into Ventoy with Arch ISO – Quick Steps

1. **Plug in your Ventoy USB** (with the Arch ISO copied to it).

2. **Power on the target computer**, (if it boots into Ventoy, skip to step 4, otherwise reboot) and immediately press the BIOS/boot menu key:  
   - Common keys: `F2`, `DEL`, `F12`, `ESC`, or `F10` (depends on the manufacturer).  
   - You may need to **log in to the BIOS** if it's locked.

3. **Enter the Boot Menu** or change **Boot Order** to select your USB drive.

4. **Ventoy boots up**, showing a menu of ISOs—select the Arch Linux ISO and press Enter.

### ⚠️ UEFI & Secure Boot Notes

- **Ventoy works in both UEFI and Legacy BIOS**—no need to enable CSM or Legacy Mode.  
- It also supports **Secure Boot**, but setup requires enrolling a custom key:
  - This is **for advanced users only**—misconfiguration could brick or lock your system.

Once you select the Arch ISO from Ventoy’s menu, it’ll boot into the Arch installer. You’re now ready to begin the installation!

### Ventoy's Normal or GRUB2 mode?

Some systems—especially certain UEFI firmwares—may not boot certain ISOs properly in Ventoy’s default mode. GRUB2 mode can serve as a fallback if:

- You get a blank screen or failed boot in normal mode.
- Your system has strict Secure Boot settings or weird UEFI behavior.

---

## Installing Arch Linux

### Verify Boot Mode

Run the following command to check if you're in **UEFI mode**:

```bash
ls /sys/firmware/efi
```

- If the directory exists, you're in **UEFI mode** (recommended).
- If not, you're in **BIOS mode**, and you may need to tweak your firmware settings.

---

## Partitioning

### Check Existing EFI Partition

1. List your disks and partitions:

    ```bash
    lsblk
    fdisk -l <disk>
    ```

2. Ensure the disk is using **GPT (GUID Partition Table)**. If it's **MBR**, you’ll need to convert it.

### Create EFI Partition

1. If the disk is using **MBR**, convert it to **GPT** using `fdisk`:

 ```bash
 fdisk /dev/sda
 ```

 - Press `g` to create a new **GPT partition table** (**Warning: This will erase all data**).

- Press `w` to save and exit.

2. Create a new **EFI system partition**:

    - Press `n` (new partition).
    - Select partition number **1**.
    - First sector: **Default** (`2048`).
    - Size: **+1G**.
    - Press `t` to change type → **EFI System (1 or ef00)**.
    - Press `w` to write changes.

3. Create the **root partition**:

    - Press `n` (new partition).
    - Select partition number **2**.
    - First sector: **Default**.
    - Last sector: **Default** (use remaining disk space).
    - Partition type: **Linux LVM (44)**.
    - Press `w` to save and exit.

### Use [LVM on LUKS](https://wiki.archlinux.org/title/Dm-crypt/Encrypting_an_entire_system#LVM_on_LUKS)

1. Create the encrypted container. Confirm with `YES` and create password.

 ```bash
 cryptsetup luksFormat /dev/sda2
 ```

1. Open the container

  ```bash
  cryptsetup open /dev/sda2 cryptlvm
  ```

### Prepare the logical volumes

1. Create a physical volume on top of the opened LUKS container:

 ```bash
 pvcreate /dev/mapper/cryptlvm
 ```

1. Create a volume group with a cool name

 ```bash
 vgcreate MyVolGroup /dev/mapper/cryptlvm
 ```

1. Create the logical volumes

 ```bash
 lvcreate -L 4G -n swap MyVolGroup
 lvcreate -L 32G -n root MyVolGroup
 lvcreate -l 100%FREE -n home MyVolGroup
 ```

1. Leave at least 256 MiB free space in the volume group to allow using [e2scrub(8)](https://man.archlinux.org/man/e2scrub.8). After creating the last volume with `-l 100%FREE`, this can be accomplished by reducing its size with `lvreduce -L -256M MyVolGroup/home`.

---

<!-- ### Format the Partitions

1. Format the **EFI partition**:

    ```bash
    mkfs.fat -F32 /dev/sda1
    ```

2. Format the **root partition** as **ext4**:

    ```bash
    mkfs.ext4 /dev/sda2
    ```

### Mount the Partitions

1. Mount the **root partition**:

    ```bash
    mount /dev/sda2 /mnt
    ```

2. Mount the **EFI partition**:

    ```bash
    mount --mkdir /dev/sda1 /mnt/boot
    ```

---

## Install Essential Packages

Run the following command to install the **base system**:

```bash
pacstrap -K /mnt base linux linux-firmware
```

### Generate Filesystem Table

Generate an `fstab` file to automatically mount partitions on boot:

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot into the New System

Enter your newly installed Arch Linux environment:

```bash
arch-chroot /mnt
```

---

## System Configuration

### **Set Time Zone**

```bash
ln -sf /usr/share/zoneinfo/US/Pacific /etc/localtime
```

### **Sync Hardware Clock**

```bash
hwclock --systohc
```

### Configure Locale

1. Edit `/etc/locale.gen` and uncomment:

    ```
    en_US.UTF-8 UTF-8
    ```

2. Generate locales:

    ```bash
    locale-gen
    ```

### Set Hostname

Create the `/etc/hostname` file with your chosen hostname:

```bash
echo "myhostname" > /etc/hostname
```

### Set Root Password

```bash
passwd
```

---

## Install and Configure GRUB

### Install GRUB and Boot Manager

```bash
pacman -S grub efibootmgr
```

### Install GRUB to the EFI System Partition (ESP)

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

### Generate GRUB Configuration

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

---

## Final Steps

1. Exit chroot:

    ```bash
    exit
    ```

2. Unmount partitions:

    ```bash
    umount -R /mnt
    ```

3. Reboot:

    ```bash
    reboot
    ```

## Next Steps

### Network Configuration -->
