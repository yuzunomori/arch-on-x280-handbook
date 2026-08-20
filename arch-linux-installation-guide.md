# 🏛️ Arch Linux Installation Guide

## 💻 The Setup

This guide runs on a **Lenovo ThinkPad X280** (Intel Core i5‑8350U, 8GB RAM, 256GB NVMe SSD, UEFI) with x86_64 architecture, Btrfs filesystem, GRUB bootloader, Intel microcode, Secure Boot turned off, and no swap configured.

---

## 💿 Prepare ISO Image

1. Download the latest ISO image file from the [official Arch Linux website](https://archlinux.org/download/).
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
2. Connect to Wi-Fi:
    ```
    # Launch wireless control
    iwctl

    # List available devices and note your device name (e.g., `wlan0`)
    device list

    # Scan, show available networks, and connect to Wi-Fi
    station wlan0 scan
    station wlan0 get-networks
    station wlan0 connect "<WIFI_SSID>"

    # Exit wireless control
    exit
    ```
3. Verify internet connection:
    ```
    # Check valid IP address
    ip addr show

    # Check internet access
    ping archlinux.org -c 1
    ```
4. Set a temporary installer root password:
    ```
    passwd
    ```
5. Check system clock is synchronized:
    ```
    # System clock synchronized should return "yes"
    timedatectl status

    # If not, run:
    timedatectl set-ntp true
    ```

---

## 💽 Partition, Format and Mount

1. Manage disk partitions:
    ```
    # List all disks and note your disk name (e.g., `/dev/nvme0n1`)
    fdisk -l

    # Modify target disk partition:
    cfdisk /dev/nvme0n1
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
5. Write the partition table to disk and exit `cfdisk`.
6. Verify and note each partition name (e.g `/dev/nvme0n1p1` and `/dev/nvme0n1p2`):
    ```
    fdisk -l
    ```
7. Wipe filesystem signatures on each partition:
    ```
    wipefs -a /dev/nvme0n1p1
    wipefs -a /dev/nvme0n1p2
    ```
8. Format boot partition:
    ```
    mkfs.fat -F 32 /dev/nvme0n1p1
    ```
9. Format root partition:
    ```
    mkfs.btrfs /dev/nvme0n1p2
    ```
10. Prepare `Btrfs` subvolumes:
    ```
    # Temporary mount
    mount /dev/nvme0n1p2 /mnt

    # Create root subvolume
    btrfs subvolume create /mnt/@

    # Recheck created subvolume
    btrfs subvolume list /mnt

    # Unmount
    umount /mnt

    # Mount root subvolume
    mount -o noatime,compress=zstd,ssd,discard=async,subvol=@ /dev/nvme0n1p2 /mnt

    # Create boot mount point
    mkdir -p /mnt/boot

    # Mount boot partition
    mount /dev/nvme0n1p1 /mnt/boot

    # Verify partitions
    mount | grep boot
    mount | grep btrfs
    ```

---

## 📦 Install Base System

1. Update mirror list:
    ```
    # Scan for mirror list (adjust country codes as needed)
    reflector --latest 10 --country TH,SG --protocol https --sort rate --save /etc/pacman.d/mirrorlist

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

1. Change root into the newly installed system:
    ```
    arch-chroot /mnt
    ```
2. Set laptop timezone:
    ```
    ln -sf /usr/share/zoneinfo/Asia/Bangkok /etc/localtime
    hwclock --systohc
    ```
3. Configure locale by uncommenting at least one locale you plan to use (e.g. `en_US.UTF-8 UTF-8`):
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
    echo "x280" > /etc/hostname
    cat /etc/hostname
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
    useradd -m -G wheel your-username-here
    passwd your-username-here
    ```
9. Make members of group wheel execute any command:
    ```
    # Uncomment on %wheel ALL=(ALL:ALL) ALL
    visudo
    ```
10. Install bootloader and utilities:
    ```
    pacman -S grub efibootmgr git reflector pacman-contrib sof-firmware pipewire pipewire-pulse pipewire-alsa wireplumber tlp tlp-rdw acpid brightnessctl smartmontools
    ```
11. Enable battery and power event services:
    ```
    systemctl enable tlp
    systemctl enable acpid
    ```
12. Ensure early microcode loading is applied:
    ```
    mkinitcpio -P
    ```
13. Setup bootloader:
    ```
    grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
    grub-mkconfig -o /boot/grub/grub.cfg
    ```
14. Exit chroot, unmount partitions, and reboot:
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
    # You can skip these view-only commands...
    nmcli device
    nmcli device wifi list

    # Connect to Wi‑Fi
    nmcli device wifi connect "<WIFI_SSID>" password "<WIFI_PASS>"
    
    # Verify connection
    ip addr show
    ping archlinux.org -c 1
    ```
3. Enable network time synchronization:
    ```
    # Activate systemd-timesyncd for network time
    sudo timedatectl set-ntp true

    # Verify synchronization state
    timedatectl status

    # Show detailed timesync information
    timedatectl timesync-status

    # Check time sync service status
    systemctl status systemd-timesyncd --no-pager
    ```
4. System verification:
    ```
    # Boot verification
    sudo systemctl --failed
    
    # Verify Btrfs volumes
    findmnt /
    
    # Verify subvolumes
    sudo btrfs subvolume list /

    # Check disk usage before moving on
    df -h

    # Check enabled services
    sudo systemctl list-unit-files --state=enabled

    # Check system logs for error entries (optional)
    # Note: It's normal to see some error lines here.
    sudo journalctl -p 3 -xb
    ```
5. Update the system:
    ```
    sudo pacman -Syu
    ```
6. Backup current package list:
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
