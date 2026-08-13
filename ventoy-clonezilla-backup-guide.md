# 📦 Ventoy × Clonezilla Backup Guide

## 💻 The Setup

This guide is my go-to for backing up my Lenovo ThinkPad X280. Instead of cloning the whole disk, we’re only backing up the EFI and Btrfs partitions — it’s faster, cleaner, and keeps your data organized. I use a 120GB Hikvision SSD in an ORICO enclosure, but any external drive works.

The parameters in this guide are tailored to a **Lenovo ThinkPad X280**. Depending on your hardware, device node names and file names may vary:

| Variable | Example Value | Description |
| :--- | :--- | :--- |
| **Internal Drive** | `/dev/nvme0n1` | Target NVMe node (`/dev/sda` for SATA). |
| **Backup Drive** | `/dev/sdb1` | External USB partition (verify via `lsblk`). |
| **Backup Name** | `backup-yyyy-mm-dd-short-description` | Follow this custom naming convention. |

> **Note:** Always verify device nodes via `lsblk` before running destructive commands like `sfdisk` or `grub-install`.

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
    ├── 📁 ISOs/                                        # Contains your bootable ISO files
    │   ├── clonezilla-live-3.3.3-15-amd64.iso
    │   └── archlinux-2026.08.01-x86_64.iso
    ├── 📁 backup-yyyy-mm-dd-short-description/         # Renamed from Clonezilla’s auto folder
    └── 📄 backup-yyyy-mm-dd-short-description.sfdisk   # Partition dump file, aligned to same convention
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

    > This dump file is required for restoring to any SSD, ensuring correct partition boundaries and alignment regardless of hardware.

    ```
    # Find your backup drive
    lsblk
    
    # Mount backup drive
    sudo mkdir -p /mnt/backup
    sudo mount /dev/sdb1 /mnt/backup
    
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

    > Restoring the partition table ensures identical boundaries on any SSD. Without this, UUIDs and alignment may mismatch.

    ```
    # Find your backup drive
    lsblk
    
    # Mount backup drive to a neutral location
    sudo mkdir -p /mnt/backup
    sudo mount /dev/sdb1 /mnt/backup
    
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
32. When prompted, select **Reboot**.
33. Done! Now you can safely remove your backup drive.

---

## 📋 Post-Restore Verification

> Note: Run these commands after booting into your restored Arch system. If restoring to a brand-new SSD and the system fails to boot, boot from your Arch ISO and `arch-chroot` into your system to run these commands.

After reboot, confirm your system is healthy and consistent:

- **System Reconfiguration & UUID Verification :** Confirm mount points, update `/etc/fstab` if hardware changed, and refresh boot binaries:
    ```
    # Confirm root and boot mountpoints are clean
    findmnt -nt btrfs,vfat
    
    # Compare partition UUIDs with /etc/fstab
    lsblk -f
    sudo cat /etc/fstab

    # If differ, update /etc/fstab manually to match the new values
    sudo vim /etc/fstab
    
    # Reinstall GRUB bootloader
    sudo grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
    sudo grub-mkconfig -o /boot/grub/grub.cfg
    
    # Regenerate initramfs
    sudo mkinitcpio -P
    
    # Verify EFI boot entries
    sudo efibootmgr
    ```

- **Btrfs Filesystem Health & Scrub :** Verify partition capacity is fully recognized and run an active checksum scrub to catch block corruption:
    ```
    # Check mounted subvolume capacity and metadata allocation
    sudo btrfs filesystem usage /
    
    # Run an immediate integrity scrub
    sudo btrfs scrub start -B /
    
    # Check error stats, expected all counters returns 0
    # If not, run `sudo btrfs device stats -z /` to reset it
    sudo btrfs device stats /
    ```

- **System Logs :** Scan the journal from the current boot to ensure no NVMe driver, Btrfs metadata, or partition mount warnings occurred:
    ```
    # Check for high-priority kernel or disk errors from current boot
    # This should not return any errors, but if it does, investigate it
    sudo journalctl -p 3 -b
    ```

---

## ⚙️ Maintenance

Keep your backup USB reliable, keep backup sizes minimal, and manage your image lifecycle over time.

1. **Pre-Backup OS & Btrfs Cleanup :** Since Clonezilla copies all used filesystem blocks, clean up unnecessary data on Arch before booting into Clonezilla to keep image sizes small:

    > Always run these steps before creating a new backup, whether restoring to the original SSD or migrating to any other SSD.

    ```
    # Keep only the current version of installed packages in pacman cache
    sudo paccache -r
    
    # Trim systemd journal logs older than 14 days
    sudo journalctl --vacuum-time=2w
    
    # Check unallocated space and metadata usage ratio
    sudo btrfs filesystem usage /
    
    # Prevent metadata ENOSPC errors by reclaiming sparse data & metadata chunks
    sudo btrfs balance start -dusage=20 -musage=20 /
    
    # Check for hardware/driver errors and run checksum scrub
    sudo btrfs device stats /
    sudo btrfs scrub start -B /
    
    # Trim NVMe blocks to maintain drive performance
    sudo fstrim -v /
    ```

2. **Image Retention & Storage Management :** When cleaning up space, always delete both the backup folder and its paired partition dump file — the dump file is critical for restoring correct partition alignment on any SSD, not just the original one.

3. **Ventoy & ISO Lifecycle**
    - Upgrade **Ventoy** using `Ventoy2Disk` (or the Linux script) with the Update option (`-u`) — this updates the bootloader on your external drive without touching your ISOs or backup images.
    - Update your **Arch Linux ISO** every few months so you have a modern kernel and up‑to‑date Btrfs/GRUB tools if you ever need to `chroot` or repair boot entries via `efibootmgr`.
    - Keep your **Clonezilla ISO** updated as well, since newer builds improve compatibility with NVMe and EFI systems.

---

## 🤝 Credits & Contributions

This guide came together through hands-on testing on my own machine, refined with a bit of help from AI (Gemini, Copilot, and ChatGPT). I’ve personally run through these steps to make sure everything works, but I’m still actively learning Arch Linux and discovering better ways to do things.

If you spot anything that could be improved, fixed, or made more efficient, I’d love to hear it! Pull requests, feedback, and tips are always welcome to help keep this guide solid.
