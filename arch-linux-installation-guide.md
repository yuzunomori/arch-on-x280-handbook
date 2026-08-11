# 🏛️ Arch Linux Installation Guide

## 💻 The Setup

This guide runs on a **Lenovo ThinkPad X280** (Intel Core i5‑8350U, 8GB RAM, 256GB NVMe SSD, UEFI) with x86_64 architecture, NVMe storage (`/dev/nvme0n1`), Btrfs filesystem, GRUB bootloader, Intel microcode, Secure Boot turned off, and no swap configured.

---

## 💿 Prepare ISO Image

1. Download latest image file:
    ```
    https://archlinux.org/download/
    ```
2. Flash image file to USB flash drive.
3. Turn off **Secure Boot** for easy setup.
4. Insert USB flash drive into the laptop and power it on.
5. Enter **Boot Menu**, then boot from USB flash drive.
6. Select **Arch Linux install medium (x86_64, UEFI)**.

---

## 🌐 Setup Environment

1. Adjust terminal font size:
    ```
    setfont ter-120b
    ```
2. Connect to Wi-Fi (Usually `wlan0`):
    ```
    iwctl
    device list
    station [device_name] scan
    station [device_name] get-networks
    station [device_name] connect [network_name]
    exit
    ```
3. Verify internet connection:
    ```
    # Check valid IP address
    ip addr show

    # Check internet access
    ping archlinux.org -c 1
    ```
4. Set temporary root password:
    ```
    passwd
    ```
5. Check system clock is synchronized:
    ```
    timedatectl status
    ```

---

## 💽 Partition, Format and Mount

1. Manage disk partitions:
    ```
    # List all disks
    fdisk -l

    # Modify target disk partition:
    cfdisk [disk_name, "/dev/nvme0n1"]
    ```
2. Delete all partitions until you see only **Free space** row left.
3. Create boot partition:
    ```
    Size: 1G
    Type: EFI System
    ```
4. Create filesystem partition:
    ```
    Size: (allocate the rest)
    Type: Linux filesystem
    ```
5. Apply write partition table.
6. Verify partitions:
    ```
    fdisk -l
    ```
7. Clean up signature of each partition:
    ```
    wipefs -a [partition_name, "/dev/nvme0n1p1"]
    wipefs -a [partition_name, "/dev/nvme0n1p2"]
    ```
8. Format boot partition:
    ```
    mkfs.fat -F 32 [partition_name, "/dev/nvme0n1p1"]
    ```
9. Format root partition:
    ```
    mkfs.btrfs [partition_name, "/dev/nvme0n1p2"]
    ```
10. Prepare `Btrfs` subvolumes:
    ```
    # Temporary mount
    mount [partition_name, "/dev/nvme0n1p2"] /mnt

    # Create subvolumes
    btrfs subvolume create /mnt/@
    btrfs subvolume create /mnt/@home

    # Recheck created subvolumes
    btrfs subvolume list /mnt

    # Unmount
    umount /mnt

    # Mount root subvolume
    mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ [partition_name, "/dev/nvme0n1p2"] /mnt

    # Create mount points for home
    mkdir -p /mnt/{boot,home}

    # Mount home subvolume
    mount -o noatime,compress=zstd,ssd,discard=async,subvol=@home [partition_name, "/dev/nvme0n1p2"] /mnt/home

    # Mount EFI
    mount [partition_name, "/dev/nvme0n1p1"] /mnt/boot

    # Verify
    mount | grep btrfs
    ```

---

## 📦 Install Base System

1. Update mirror list (Adjust the country code to your location):
    ```
    # Scan for mirror list
    reflector --latest 5 --country TH --protocol https --sort rate --save /etc/pacman.d/mirrorlist

    # Verify mirrorlist file
    cat /etc/pacman.d/mirrorlist
    ```
2. Install essential packages:
    ```
    pacstrap -K /mnt base linux linux-firmware networkmanager vim base-devel intel-ucode btrfs-progs mesa vulkan-intel intel-media-driver
    ```
3. Generate `fstab` and verify:
    ```
    # Generate
    genfstab -U /mnt >> /mnt/etc/fstab

    # Verify
    cat /mnt/etc/fstab
    ```

---

## ⚙️ System Configuration

1. Change root to laptop:
    ```
    arch-chroot /mnt
    ```
2. Set laptop timezone:
    ```
    ln -sf /usr/share/zoneinfo/Asia/Bangkok /etc/localtime
    hwclock --systohc
    ```
3. Configure locale by uncomment at least one locale you plan to use (e.g. en_US.UTF-8 UTF-8):
    ```
    vim /etc/locale.gen
    locale-gen
    ```
4. Create locale config file and verify it:
    ```
    echo "LANG=en_US.UTF-8" > /etc/locale.conf
    cat /etc/locale.conf
    ```
5. Set hostname and verify:
    ```
    echo "[hostname]" > /etc/hostname
    ```
6. Enable networking:
    ```
    systemctl enable NetworkManager
    ```
7. Set root password:
    ```
    passwd
    ```
8. Add new user and set the password:
    ```
    useradd -m -G wheel [username]
    passwd [username]
    ```
9. Make members of group wheel execute any command by uncomment **%wheel ALL=(ALL:ALL) ALL**:
    ```
    visudo
    ```
10. Install bootloader and utilities:
    ```
    pacman -S grub efibootmgr git reflector pacman-contrib sof-firmware pipewire pipewire-pulse pipewire-alsa wireplumber tlp tlp-rdw acpid brightnessctl smartmontools bluez bluez-utils
    ```
11. Enable battery and power event services:
    ```
    systemctl enable tlp
    systemctl enable acpid
    systemctl enable bluetooth
    ```
12. Ensure early microcode loading is applied:
    ```
    mkinitcpio -P
    ```
13. Setup bootloader and verify:
    ```
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg
    efibootmgr -v
    ```
14. Finalize:
    ```
    exit
    umount -R /mnt
    reboot
    ```
15. Remove USB flash drive.

---

## ✨ Initial System Setup

1. Login to the system.
2. Connect to Wi-Fi:
    ```
    # Connect
    nmcli device
    nmcli device wifi list
    nmcli device wifi connect "[wifi_ssid]" password "[wifi_password]"
    
    # Verify
    ip addr show
    ping archlinux.org -c 1
    ```
3. Enable `TRIM` to turn on automatic SSD maintenance:
    ```
    sudo systemctl enable --now fstrim.timer
    sudo systemctl status fstrim.timer
    sudo systemctl list-timers fstrim.timer
    ```
4. System verification:
    ```
    # Boot verification
    sudo systemctl --failed
    
    # Verify Btrfs volumes
    findmnt /
    findmnt /home
    
    # Verify subvolumes
    sudo btrfs subvolume list /

    # Verify journal logs for errors
    sudo journalctl -p 3 -xb

    # Check disk usage before moving on
    df -h

    # Check service
    sudo systemctl list-unit-files --state=enabled
    ```
5. Update the system:
    ```
    sudo pacman -Syu
    ```
6. Save package list:
    ```
    # Create directory
    mkdir -p ~/arch
    
    # Dump package lists
    pacman -Qe > ~/arch/arch-pkglist.txt
    pacman -Qqm > ~/arch/arch-aurlist.txt
    ```
7. Final reboot:
    ```
    reboot
    ```

---

## 🤝 Credits & Contributions  
This guide is adapted and tweaked from **Josean Martinez’s** YouTube video [*The Only Arch Linux Installation Guide You'll Ever Need*](https://www.youtube.com/watch?v=TS1ghG3c3xI), with a bit of help from AI along the way.

I’m still learning my way around Arch, so this file may have rough edges. Contributions, fixes, and tips are welcome — feel free to open a pull request or drop feedback to help polish it.
