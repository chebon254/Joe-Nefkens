# Windows 10 + Ubuntu Dual Boot — Lenovo ThinkPad X1 Carbon

This guide walks you through formatting your ThinkPad X1 Carbon, installing Windows 10 on a 60 GB partition, leaving the rest of the disk empty for Ubuntu, and setting up a boot menu so you can choose which OS to run on every startup.

---

## What You Will End Up With

| Partition | Size | Purpose |
|---|---|---|
| EFI System Partition | 500 MB | Boot files (created by Windows installer) |
| Windows 10 | 60 GB | Your Windows installation |
| Unallocated | ~175 GB | Reserved for Ubuntu (install later) |

---

## What You Need Before You Start

- Lenovo ThinkPad X1 Carbon (any generation)
- Windows 10 ISO — `Win10_22H2_English_x64v1.iso` or newer
- USB drive, 8 GB or larger
- A second device (phone or another PC) to follow this guide while your laptop is being formatted
- **Back up all important files** — this process wipes the entire disk

---

## Step 1 — Create a Bootable Windows 10 USB

### On Linux (current system, before formatting):

```bash
# Find your USB drive device name (look for your USB size, e.g. /dev/sdb)
lsblk

# Write the ISO to the USB (replace /dev/sdX with your USB device — NOT a partition like sdb1)
sudo dd if=/home/kibe/Downloads/Win10_22H2_English_x64v1.iso of=/dev/sdX bs=4M status=progress oflag=sync
```

> **Warning:** Double-check the device name. Writing to the wrong device will wipe that disk.

### On Windows (alternative):
Use **Rufus** — select the ISO, choose GPT partition scheme, UEFI target, and click Start.

---

## Step 2 — Configure ThinkPad BIOS/UEFI

1. Shut down the laptop completely
2. Power on and immediately press **F1** repeatedly to enter BIOS Setup
3. Make the following changes:

### Security tab:
- **Secure Boot** → Set to `Disabled`
  *(Required so Ubuntu can be installed later without issues)*

### Startup tab:
- **UEFI/Legacy Boot** → Set to `UEFI Only`
- **Boot Order** → Move `USB HDD` to the top of the list

4. Press **F10** to Save and Exit

---

## Step 3 — Boot from USB

1. Insert the bootable Windows 10 USB
2. Power on the ThinkPad
3. If it doesn't boot from USB automatically, press **F12** at the ThinkPad logo to open the **Boot Menu**
4. Select your USB drive from the list (usually shown as `USB HDD` or the USB brand name)

---

## Step 4 — Windows 10 Installation

1. Click **Next** on the language/keyboard screen
2. Click **Install now**
3. When asked for a product key, click **"I don't have a product key"** (you can activate later)
4. Select **Windows 10 Pro** or **Windows 10 Home** — your choice
5. Accept the license agreement → click **Next**
6. Choose **"Custom: Install Windows only (advanced)"**

---

## Step 5 — Partition the Disk (Critical Step)

This is where you set up the 60 GB partition for Windows and leave the rest empty for Ubuntu.

> You will see all existing partitions. If there are any old partitions listed, **delete them all** one by one until you see one large block of **Unallocated Space** covering the full disk (e.g. ~238 GB).

### Create the Windows partition:

1. Click on the **Unallocated Space** block
2. Click **New**
3. In the size field, type: `61440` *(that is 60 GB in MB)*
4. Click **Apply**
5. Windows will prompt: *"To ensure that all Windows features work correctly, Windows might create additional partitions."* — Click **OK**
   - This creates a small EFI partition (~500 MB) and the 60 GB Windows partition automatically

### Leave the rest unallocated:

6. You will now see:
   - Partition 1: ~500 MB (EFI — do not touch)
   - Partition 2: ~16 MB (MSR — do not touch)
   - Partition 3: 60 GB (your Windows partition)
   - **Unallocated: ~175 GB** ← Leave this completely alone
7. Click on **Partition 3 (60 GB)** to select it
8. Click **Next** — Windows will install here

---

## Step 6 — Complete Windows Installation

- Windows will copy files, restart several times — this is normal
- **Do not remove the USB** until Windows has fully booted into the setup screen
- Go through the Windows setup (region, keyboard, account, etc.)
- Once on the Windows desktop, remove the USB

---

## Step 7 — Verify Partitions in Windows

After installation, confirm the partition layout:

1. Press `Win + X` → click **Disk Management**
2. You should see:
   - EFI partition (~500 MB)
   - Windows C: drive (~60 GB)
   - **Unallocated space (~175 GB)** — this is reserved for Ubuntu

> Do not format or assign a drive letter to the unallocated space. Leave it as-is for Ubuntu.

---

## Step 8 — Install Ubuntu Later (When Ready)

When you are ready to install Ubuntu:

1. Create a bootable Ubuntu USB (same `dd` method as Step 1, or use Balena Etcher)
2. Boot from the USB → choose **"Install Ubuntu"**
3. On the installation type screen, choose **"Something else"**
4. Select the **unallocated space** and create:
   - Swap partition: `4096 MB` (type: swap)
   - Root partition: remaining space (type: ext4, mount point: `/`)
5. Complete the Ubuntu installation

---

## Step 9 — Set Up Dual Boot Menu (GRUB)

After Ubuntu is installed, it will automatically install **GRUB** — the boot menu that lets you choose your OS on every startup.

### What happens on boot:
- A menu appears for **5–10 seconds** showing:
  - Ubuntu (default)
  - Windows 10
- If you do nothing, it boots Ubuntu automatically
- Use arrow keys to select, press Enter to confirm

### Change the default OS or timeout (optional):

In Ubuntu terminal:
```bash
sudo nano /etc/default/grub
```

Key settings:
```bash
GRUB_DEFAULT=0          # 0 = Ubuntu, change to "Windows 10" or the menu position number
GRUB_TIMEOUT=10         # Seconds to wait before auto-booting (increase if needed)
```

Save and apply:
```bash
sudo update-grub
```

### If Windows does not appear in the GRUB menu:

```bash
sudo os-prober
sudo update-grub
```

---

## Troubleshooting

### ThinkPad boots straight into Windows, skipping the GRUB menu:
- Enter BIOS (F1) → go to **Startup** tab
- Change boot order: move `ubuntu` or `Linux Boot Manager` above `Windows Boot Manager`
- Save with F10

### "No bootable device" error after installation:
- Enter BIOS (F1) → **Startup** → **Boot** tab
- Make sure `Windows Boot Manager` is listed
- If missing, click **Add Boot Option** and browse to `\EFI\Microsoft\Boot\bootmgfw.efi`

### Windows repair loop after Ubuntu installation:
- Boot from Windows USB → **Repair your computer** → **Startup Repair**
- This is rare when Ubuntu installs GRUB correctly

### GRUB rescue prompt on boot:
Run Boot Repair from Ubuntu Live USB:
```bash
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt update && sudo apt install -y boot-repair
boot-repair
```
Click **Recommended repair**.

---

## Quick Reference — ThinkPad Function Keys

| Key | Action |
|---|---|
| F1 | Enter BIOS Setup |
| F12 | Boot Menu (choose boot device) |
| F10 | Save and Exit BIOS |
| Esc | Exit without saving |

---

## Summary

1. Create bootable USB → 2. Disable Secure Boot in BIOS → 3. Boot from USB → 4. Install Windows to 60 GB partition → 5. Leave ~175 GB unallocated → 6. Later install Ubuntu on unallocated space → 7. GRUB dual boot menu appears automatically
