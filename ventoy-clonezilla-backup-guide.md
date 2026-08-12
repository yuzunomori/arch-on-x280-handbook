# 📦 Ventoy × Clonezilla Backup Guide

## 💻 The Setup

This guide is my go-to for backing up my Lenovo ThinkPad X280. Instead of cloning the whole disk, we’re only backing up the EFI and Btrfs partitions — it’s faster, cleaner, and keeps your data organized. I use a 120GB Hikvision SSD in an ORICO enclosure, but any external drive works.

> ⚠️ **Scope & Limitations:** This guide is strictly for full-system recovery on the same SSD (`/dev/nvme0n1`). Because we save and restore the exact partition geometry and file system UUIDs, you won’t need to mess with resizing or re-configuring bootloaders after a restore — you’ll be back exactly where you started. If you are migrating to a different disk, you'll need additional steps.

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
12. Wait until you see your backup drive listed on the screen then press `Ctrl + C` to exit the device scan.
13. Select your target partition to store the backup.
14. Select **no-fsck** to skip file system checking.
15. Select the **Clonezilla image repository** (press `Tab` twice to highlight `<Done>`, then `Enter`).
16. Review target source then press `Enter` to continue.
17. Clonezilla will ask about time synchronization; skip it if you have no internet connection.
18. Select **Beginner** mode.
19. Select **saveparts** for partitions backup.
20. Name your backup `backup-yyyy-mm-dd-short-description` without extension.
21. Select partitions you want to back up. By default, select both EFI (`nvme0n1p1`) and Btrfs (`nvme0n1p2`) partitions.
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
32. When prompted, select **cmd** to drop into the shell.
33. Backup your disk layout so it can be restored later:
    ```
    # Find your backup drive
    lsblk
    
    # Mount backup drive
    sudo mkdir -p /mnt/backup
    sudo mount /dev/[backup_partition] /mnt/backup
    
    # Dump partition table
    sudo sfdisk -d /dev/nvme0n1 > dump.sfdisk
    
    # Copy partition table dump file to backup drive
    sudo cp dump.sfdisk /mnt/backup/backup-yyyy-mm-dd-short-description.sfdisk
    
    # Flush buffers
    sync
    
    # Safely unmount the backup drive
    sudo umount /mnt/backup
    ```
34. Exit the Clonezilla shell and reboot the system:
    ```
    sudo reboot
    ```
35. Done! Now you can safely remove your backup drive.
36. For experienced users, here are compact commands you can run separately after the Clonezilla backup. This is optional and not part of the main guide — beginners may safely skip it.

    > ⚠️ Remember to edit the variables before executing:
    > - `BACKUP_NAME` → set to today’s date with short description (e.g., `backup-2026-08-12-fresh-arch-install`)
    > - `SOURCE_DEV` → your source disk (e.g., `/dev/nvme0n1`)
    > - `BACKUP_DEV` → your backup drive (e.g., `/dev/sdb1`)

    ```
    # Define variables
    BACKUP_NAME=""
    SOURCE_DEV=""
    BACKUP_DEV=""
    
    # Run Clonezilla backup (confirm when prompted)
    sudo /usr/sbin/ocs-sr -q2 -c -j2 -edio -z9p -i 4096 -sfsck -sgoc -p choose saveparts "$BACKUP_NAME" ${SOURCE_DEV}p1 ${SOURCE_DEV}p2
    
    # After finishing, drop into the command line and run these commands to mount the backup drive
    sudo mkdir -p /mnt/backup
    sudo mount $BACKUP_DEV /mnt/backup
    
    # Dump partition table layout
    sudo sfdisk -d $SOURCE_DEV | sudo tee /mnt/backup/${BACKUP_NAME}.sfdisk > /dev/null

    # Flush buffers
    sync
    
    # Safely unmount and verify
    sudo umount /mnt/backup
    mount | grep /mnt/backup
    
    # Reboot manually
    sudo reboot
    ```

---

## 🔄 Restore from Backup

> **Scope:** This guide is designed for restoring to the same physical SSD (including wiped or corrupted partitions).

1. Turn off **Secure Boot** before proceeding.
2. Insert backup drive into the laptop and power it on.
3. Enter Boot Menu, then boot from backup drive.
4. From Ventoy boot menu, select **Clonezilla Live ISO**, then select **Boot in normal mode**.
5. From Clonezilla boot menu, select **Clonezilla live (VGA, 800x600 & To RAM)** and wait.
6. When language selection is prompted, remove your backup drive.
7. Select your preferred language and keyboard layout.
8. Select **Enter_shell** to enter the command line.
9. Run these commands to restore your exact partition boundaries:
    ```
    # Mount your backup drive
    sudo mkdir -p /mnt/backup
    sudo mount /dev/[backup_partition] /mnt/backup
    
    # Free target disk locks
    sudo swapoff -a 2>/dev/null
    sudo umount /dev/nvme0n1p* 2>/dev/null
    
    # Restore partition table (replace filename with your actual saved .sfdisk file)
    sudo sfdisk /dev/nvme0n1 < /mnt/backup/backup-yyyy-mm-dd-short-description.sfdisk

    # Flush buffers
    sync
    
    # Clean up before entering Clonezilla wizard
    sudo umount /mnt/backup
    exit
    ```
10. Select **Start Clonezilla**.
11. Select **device-image** option.
12. Select **local_dev** option.
13. Insert backup drive back into the laptop and press `Enter`.
14. Wait until you see your backup drive listed on the screen then press `Ctrl + C` to exit the device scan.
15. Select your backup drive partition where the backup image is saved.
16. Select **no-fsck** to skip file system checking.
17. Select the **Clonezilla image repository** (press `Tab` twice to highlight `<Done>`, then `Enter`).
18. Review target source then press `Enter` to continue.
19. Clonezilla will ask about time synchronization; skip it if you have no internet connection.
20. Select **Beginner** mode.
21. Select **restoreparts** for partitions restore.
22. Select your image file to restore.
23. Select partitions to be restored. By default, select both EFI (`nvme0n1p1`) and Btrfs (`nvme0n1p2`) partitions.
24. Select **Yes, check the saved image** to verify that backup is restorable.
25. Select **No, do not copy log files to a Clonezilla Live USB drive.**
26. Select **-p** to manually choose option after restoration is finished.
27. Clonezilla will prompt with direct command to use next time, press `Enter`.
28. Clonezilla will warn you about existing data loss, review it then confirm by typing `y` then `Enter`.
29. Clonezilla will ask you again, confirm by typing `y` then `Enter`.
30. Wait for Clonezilla to complete the restoration.
31. After everything is done, press `Enter`.
32. When prompted, select **reboot** to restart your device.
33. Done! Now you can safely remove your backup drive.
34. For experienced users, here are compact commands you can run separately in command line to restore. This is optional and not part of the main guide — beginners may safely skip it.

    > ⚠️ Remember to edit the variables before executing:
    > - `BACKUP_NAME` → set to today’s date with short description (e.g., `backup-2026-08-12-fresh-arch-install`)
    > - `TARGET_DEV` → your target disk (e.g., `/dev/nvme0n1`)
    > - `BACKUP_DEV` → your backup drive partition (e.g., `/dev/sdb1`)

    ```bash
    # Define variables
    BACKUP_NAME=""
    TARGET_DEV=""
    BACKUP_DEV=""

    # Mount backup drive
    sudo mkdir -p /home/partimag
    sudo mount $BACKUP_DEV /home/partimag

    # Reset partition table layout (Always run to ensure geometry matches backup)
    sudo swapoff -a 2>/dev/null
    sudo umount ${TARGET_DEV}p* 2>/dev/null
    sudo sfdisk $TARGET_DEV < /home/partimag/${BACKUP_NAME}.sfdisk

    # Flush buffers
    sync

    # Run Clonezilla partition restore
    sudo /usr/sbin/ocs-sr -e1 auto -e2 -t -r -edio -c -k -p choose restoreparts "$BACKUP_NAME" ${TARGET_DEV}p1 ${TARGET_DEV}p2
    ```

---

## ⚙️ Maintenance
Update ISOs, verify backups, organize files.

---

## 🤝 Credits & Contributions
