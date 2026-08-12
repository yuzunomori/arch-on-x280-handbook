# 📦 Ventoy × Clonezilla Backup Guide

## 💻 The Setup

This guide uses a Btrfs filesystem and backs up only the EFI and Btrfs partitions instead of cloning the entire disk. A single external SSD prepared with Ventoy serves double duty — booting Clonezilla and Arch Linux ISOs, while also storing backup images and partition table dumps. The SSD is a Hikvision C100 120GB inside an ORICO enclosure with a USB 3.0 Micro‑B cable. Your filesystem, partition layout, or enclosure may differ, but the workflow remains the same.

---

## 💿 Prepare Backup Drive

1. Download **Ventoy** for Windows:
  ```
  https://www.ventoy.net/en/download.html
  ```
2. Plug in your backup drive.
3. Run **Ventoy2Disk.exe** to flash Ventoy onto the drive.
4. Download **Clonezilla Live** ISO file (AMD64):
  ```
  https://clonezilla.org/downloads/download.php?branch=stable
  ```
5. Download latest **Arch Linux** ISO file:
  ```
  https://archlinux.org/download/
  ```
6. Move ISO files into an `ISOs/` folder on the backup drive.
    ```
    📁 Ventoy Drive Root/
    └── 📁 ISOs/
        ├── clonezilla-live-3.3.3-15-amd64.iso
        └── archlinux-2026.08.01-x86_64.iso
    
    # After creating the backup image and saving the partition dump, the drive will look like this:
    📁 Ventoy Drive Root/
    ├── 📁 ISOs/                                    # Contains your bootable ISO files
    │   ├── clonezilla-live-3.3.3-15-amd64.iso
    │   └── archlinux-2026.08.01-x86_64.iso
    ├── 📁 backup-260808-fresh-arch-install.img/    # Renamed from Clonezilla’s auto folder
    └── 📄 backup-260808-fresh-arch-install.sfdisk  # Partition dump file, aligned to same convention
    ```
7. Safely eject and remove the Ventoy backup drive.

---

## 💾 Save Backup to Drive
Partition table dump + Clonezilla `saveparts`.

---

## 🔄 Restore from Backup
Recover `sfdisk` + Clonezilla `restoreparts`.

---

## ⚙️ Maintenance
Update ISOs, verify backups, organize files.

---

## 🤝 Credits & Contributions
