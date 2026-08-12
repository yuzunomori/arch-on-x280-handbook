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
    ├── 📁 backup-260808-fresh-arch-install/        # Renamed from Clonezilla’s auto folder
    └── 📄 backup-260808-fresh-arch-install.sfdisk  # Partition dump file, aligned to same convention
    ```
7. Safely eject and remove the Ventoy backup drive.

---

## 💾 Save Backup to Drive

1. Turn off **Secure Boot** before proceeding.
2. Insert backup drive into the laptop and power it on.
3. Enter Boot Menu, then boot from backup drive.
4. From Ventoy boot menu, select **Clonezilla Live ISO**, then select **Boot in normal mode**.
5. From Clonezilla boot menu, select **Clonezilla live (VGA, 800x600 & To RAM)** and wait.
6. When language selection is prompted, remove your backup drive.
7. Select your preferred language and keyboard layout.
8. Select **Start Clonezilla**.
9. Select **device-image** option.
10. Select **local_dev** option.
11. Insert backup drive back into the laptop and press `Enter`.
12. Wait until you see your backup drive listed on the screen then press `Ctrl + C` to exit.
13. Select your target partition to store the backup.
14. Select **no-fsck** to skip file system checking.
15. Select the **Clonezilla image repository** (press `Tab` twice to highlight `<Done>`, then `Enter`).
16. Review target source then press `Enter` to continue.
17. Clonezilla will ask about time synchronization; skip it if you have no internet connection.
18. Select **Beginner** mode.
19. Select **saveparts** for partitions backup.
20. Name your backup `backup-yyyy-mm-dd-short-description` without extension.
21. Select partitions you want to back up. Default is select both `nvme0n1p1` and `nvme0n1p2`.
22. Select **-z9p** compression.
23. Select **-sfsck** to skip filesystem check.
24. Select **Yes, check the saved image** to verify that backup is restorable.
25. Select **-sgoc** to skip image encryption process.
26. Select **No, do not copy log files to a Clonezilla Live USB drive.**
27. Select **-p** to manually choose option after backup is finished.
28. Clonezilla will prompt with direct command to use next time, press `Enter`.
29. Review source partitions then confirm by typing `y` then `Enter`.
30. Wait for Clonezilla to complete the backup.
31. After everything is done, press `Enter`.
32. When prompted, select **cmd** to enter command line prompt.
33. Backup your disk layout so it can be restored later:
    ```
    # Find your backup drive
    lsblk
    
    # Mount backup drive
    sudo mkdir -p /mnt/backup
    sudo mount /dev/[device_name] /mnt/backup
    
    # Dump partition table
    sudo sfdisk -d /dev/nvme0n1 > dump.sfdisk
    
    # Copy partition table dump file to backup drive
    sudo cp dump.sfdisk /mnt/backup/backup-yyyy-mm-dd-short-description.sfdisk

    # Safely unmount the backup drive
    sudo umount /mnt/backup
    ```
34. Exit the Clonezilla shell and reboot the system:
    ```
    sudo reboot
    ```
35. Now you can freely remove your backup drive.

---

## 🔄 Restore from Backup
Recover `sfdisk` + Clonezilla `restoreparts`.

---

## ⚙️ Maintenance
Update ISOs, verify backups, organize files.

---

## 🤝 Credits & Contributions
