# Ubuntu System Recovery Guide

## Quick Reference for Daemon Boot Failures

This guide helps you recover from daemon failures during Ubuntu boot without reinstalling the system.

---

## System Information

**Firmware Type**: UEFI (EFI v2.5 by INSYDE Corp.)

---

## 1. Accessing Recovery Mode

### Boot to GRUB Menu
1. Power on the system
2. Press and hold **Esc** key (or try F2, F10, F12 depending on manufacturer)
3. GRUB menu should appear

### Enter Recovery Mode
1. From GRUB menu, select **"Advanced options for Ubuntu"**
2. Choose a kernel entry with **(recovery mode)** in the name
3. Select **"root - Drop to root shell prompt"** from recovery menu
4. Remount filesystem as writable:
   ```bash
   mount -o remount,rw /
   ```

---

## 2. Diagnostic Commands

### View Boot Errors
```bash
# Show errors from last boot
journalctl -xb -p err

# View all logs from last boot
journalctl -xb

# Check specific service logs
journalctl -u <service-name>
```

### Check Service Status
```bash
# Check specific daemon
systemctl status <daemon-name>

# List all failed services
systemctl --failed

# List all services
systemctl list-units --type=service
```

---

## 3. Common Fixes

### Disable Problematic Service
```bash
systemctl disable <service-name>
systemctl mask <service-name>  # Completely prevent it from starting
```

### Fix Broken Packages
```bash
apt --fix-broken install
apt update
apt upgrade
```

### Reinstall Specific Package
```bash
apt install --reinstall <package-name>
```

### Check Disk Space
```bash
df -h
```

### Run Filesystem Check
```bash
# From recovery mode or live USB
fsck /dev/sdXY  # Replace with your partition
```

---

## 4. Emergency Boot Options

### Add to Kernel Parameters (GRUB)
Press 'e' on your boot entry in GRUB and add to the kernel line:

- `systemd.unit=rescue.target` - Minimal multi-user mode
- `systemd.unit=emergency.target` - Single-user emergency mode
- `3` - Boot to text mode (no GUI)
- `systemd.mask-tmp=false` - If /tmp issues occur

After editing, press **Ctrl+X** or **F10** to boot with these parameters.

---

## 5. Preventive Measures

### Enable Persistent Logging
```bash
# Ensure logs survive reboots
sudo mkdir -p /var/log/journal
sudo systemd-tmpfiles --create --prefix /var/log/journal
```

### Regular Configuration Backups
```bash
# Backup /etc directory
sudo tar -czf ~/etc-backup-$(date +%F).tar.gz /etc

# Restore if needed
sudo tar -xzf ~/etc-backup-YYYY-MM-DD.tar.gz -C /
```

### System Snapshots with Timeshift
```bash
# Install Timeshift
sudo apt install timeshift

# Configure and create snapshots
sudo timeshift --create --comments "Before system changes"
```

---

## 6. Alternative: UEFI Firmware Settings

Access firmware settings (usually F2 or Del during boot) to:
- Check boot order
- Disable "Fast Boot" if GRUB is being skipped
- Verify Secure Boot settings

---

## 7. Live USB Recovery

When recovery mode isn't enough:

1. Boot from Ubuntu live USB
2. Open terminal
3. Mount your system partition:
   ```bash
   sudo mount /dev/sdXY /mnt  # Replace with your partition
   sudo mount --bind /dev /mnt/dev
   sudo mount --bind /proc /mnt/proc
   sudo mount --bind /sys /mnt/sys
   ```
4. Chroot into your system:
   ```bash
   sudo chroot /mnt
   ```
5. Fix issues from within your actual system
6. Exit and reboot:
   ```bash
   exit
   sudo umount /mnt/dev /mnt/proc /mnt/sys /mnt
   sudo reboot
   ```

---

## 8. Finding Your System Partition

```bash
# List all partitions
sudo fdisk -l

# Show partition details
lsblk -f

# Your root partition is usually the largest ext4 partition
```

---

## Common Daemon Issues

### NetworkManager fails
```bash
systemctl restart NetworkManager
# or
systemctl disable NetworkManager
systemctl enable systemd-networkd
```

### Display Manager fails (no GUI)
```bash
systemctl restart gdm  # or lightdm, sddm depending on your setup
```

### Docker/Snap daemons
```bash
systemctl restart docker
systemctl restart snapd
```

---

## Emergency Contacts

- Ubuntu Forums: https://ubuntuforums.org/
- Ask Ubuntu: https://askubuntu.com/
- System logs location: `/var/log/`

---

## Notes

- Always check `/var/log/syslog` and `journalctl` output for specific error messages
- Google the exact error message for targeted solutions
- When in doubt, boot to recovery mode rather than reinstalling
- Keep a live USB readily available for emergencies

---

**Last Updated**: January 2026
