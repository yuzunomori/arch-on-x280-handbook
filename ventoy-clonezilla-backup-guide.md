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
    sudo mount [backup_partition, "/dev/sdb1"] /mnt/backup
    
    # Dump partition table
    sudo sfdisk -d /dev/nvme0n1 > dump.sfdisk
    
    # Copy partition table dump file to backup drive
    sudo cp dump.sfdisk /mnt/backup/backup-yyyy-mm-dd-short-description.sfdisk
    
    # Flush buffers
    sync
    
    # Safely unmount
    sudo umount /mnt/backup
    
    # Reboot manually
    sudo reboot
    ```
34. Done! Now you can safely remove your backup drive.

---

## 🔄 Restore from Backup

1. Turn off **Secure Boot** before proceeding.
2. Insert backup drive into the laptop and power it on.
3. Enter Boot Menu, then boot from backup drive.
4. From Ventoy boot menu, select **Clonezilla Live ISO**, then select **Boot in normal mode**.
5. From Clonezilla boot menu, select **Clonezilla live (VGA, 800x600 & To RAM)** and wait.
6. When language selection is prompted, remove your backup drive.
7. Select your preferred language and keyboard layout.
8. Select **Enter_shell** to enter the command line.
9. Before running the Clonezilla wizard, restore your exact partition boundaries:
    ```
    # Find your backup drive
    lsblk
    
    # Mount backup drive to a neutral location
    sudo mkdir -p /mnt/backup
    sudo mount [backup_partition, "/dev/sdb1"] /mnt/backup
    
    # Free target locks
    sudo swapoff -a 2>/dev/null
    sudo umount /dev/nvme0n1p* 2>/dev/null

    # Restore partition table
    sudo sfdisk /dev/nvme0n1 < /mnt/backup/backup-yyyy-mm-dd-short-description.sfdisk

    # Force kernel to re-read the updated partition table
    sudo partprobe /dev/nvme0n1

    # Flush buffers
    sync
    
    # Clean up and return to the wizard
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

---

## 📋 Post-Restore Verification

Once you reboot and log back into your system, run these quick checks to ensure your filesystem, mounts, and hardware state are 100% healthy:

- **Verify Partition Boundaries & UUIDs**  
    Confirm `sfdisk` aligned the partitions correctly and systemd mounted them via the expected UUIDs:
    ```
    # Verify partition UUIDs match /etc/fstab
    lsblk -f
    
    # Confirm root and boot mountpoints are clean
    findmnt -nt btrfs,vfat
    ```
- **Check Btrfs Filesystem Health & Run Scrub**  
    Verify partition capacity is fully recognized and run an active checksum scrub to catch block corruption:
    ```
    # Check mounted subvolume capacity and metadata allocation
    sudo btrfs filesystem usage /
    
    # Run an immediate integrity scrub (reads all blocks against metadata hashes)
    sudo btrfs scrub start -B /
    
    # Check hardware/driver I/O error stats
    sudo btrfs device stats /
    ```
    _(All counters in device stats should be 0. If any are non-zero, log them and reset with `sudo btrfs device stats -z /`)._
- **Check System Logs for Storage & Driver Errors**  
    Scan the journal from the current boot to ensure no NVMe driver, Btrfs metadata, or partition mount warnings occurred:
    ```
    # Check for high-priority kernel or disk errors from current boot
    journalctl -p 3 -b
    ```
- **Verify EFI Boot Entries**
    Ensure your motherboard firmware still recognizes the Arch Linux boot entry:
    ```
    # List UEFI boot entries
    efibootmgr
    ```
    _(If the output is empty or missing `Arch Linux`, you will need to re-install the bootloader using `grub-install` or `bootctl install` from a live USB)._

---

## ⚙️ Maintenance
Update ISOs, verify backups, organize files.

---

## 🤝 Credits & Contributions
