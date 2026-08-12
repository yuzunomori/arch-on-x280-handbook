# 📦 Ventoy × Clonezilla Backup Guide

## 💻 The Setup

This guide uses a Btrfs filesystem and backs up only the EFI and Btrfs partitions instead of cloning the entire disk. A single external SSD prepared with Ventoy serves double duty — booting Clonezilla and Arch Linux ISOs, while also storing backup images and partition table dumps. The SSD is a Hikvision C100 120GB inside an ORICO enclosure with a USB 3.0 Micro‑B cable. Your filesystem, partition layout, or enclosure may differ, but the workflow remains the same.

---

## 💿 Prepare Backup Drive
Includes Ventoy install, ISO downloads, and folder structure.

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
