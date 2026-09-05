# openSUSE Tumbleweed Post-Install

Personal post-installation checklist for a fresh **openSUSE Tumbleweed + KDE Plasma** installation.

---

## 1. Upgrade System

Perform a full distribution upgrade immediately after installation.

```bash
sudo zypper ref
sudo zypper dup
```

Reboot if the update included important system components such as the kernel, Mesa, KDE Plasma, or systemd.

```bash
systemctl reboot
```

---

## 2. Create First-Install Snapshot

After completing the initial system configuration, create a snapshot that can be used as a reference point.

### Using Snapper

Check the available configurations:

```bash
sudo snapper list-configs
```

Create a snapshot:

```bash
sudo snapper create --description "First Install"
```

Verify it:

```bash
sudo snapper list
```

> **Note:** A Btrfs/Snapper snapshot is not a backup. It protects against filesystem changes and makes rollback easier, but it does not protect against disk failure.

---

## 3. Install zram

Install the zram service:

```bash
sudo zypper in systemd-zram-service
```

Enable and start it:

```bash
sudo systemctl enable --now zramswap.service
```

Verify that zram is active:

```bash
swapon --show
```

You can also check:

```bash
zramctl
```

> **Note:** Do not manually configure zram if another zram management mechanism is already active. Only one mechanism should manage the zram swap configuration.

---

## 4. Change Hostname

Using YaST:

**YaST → Network → Hostname/DNS**

Set the desired hostname.

Alternatively, from the terminal:

```bash
sudo hostnamectl set-hostname <hostname>
```

Verify:

```bash
hostnamectl
```

---

## 5. Configure Firewall Zone for Docker

Check the current firewall zones:

```bash
sudo firewall-cmd --get-active-zones
```

Check which zone is assigned to Docker:

```bash
sudo firewall-cmd --get-zone-of-interface=docker0
```

If you intentionally want `docker0` associated with the **Home** zone:

```bash
sudo firewall-cmd --zone=home --change-interface=docker0
```

Make the change permanent:

```bash
sudo firewall-cmd --permanent --zone=home --change-interface=docker0
sudo firewall-cmd --reload
```

> **Warning:** Docker manages its own networking and firewall rules. Changing the zone can affect container connectivity and network exposure. Only do this if you have a specific reason for using the Home zone.

---

## 6. Disable Discover Notifications

If KDE Discover notifications become annoying:

**System Settings → Notifications → Application-specific settings**

Select **Discover** and disable the notifications you do not want.

> This is optional. Disabling Discover notifications does not disable system updates.

---

## 7. Add Printer

For general printer support, install CUPS:

```bash
sudo zypper in cups
```

Enable and start it:

```bash
sudo systemctl enable --now cups.service
```

For HP printers, install HPLIP:

```bash
sudo zypper in hplip hplip-hpijs
```

Add the user to the `lp` group if required:

```bash
sudo usermod -aG lp "$USER"
```

Log out and log back in for the group membership to take effect.

Check the printer service:

```bash
systemctl status cups
```

Printers can then be configured through:

**System Settings → Printers**

> HPLIP is primarily useful for HP printers. It is not necessary for every printer.

---

## 8. Set Up Flathub

Check the configured Flatpak remotes:

```bash
flatpak remotes
```

If Flathub is already configured and working, there is normally no need to remove and recreate it.

For a personal workstation, configure Flathub for the current user:

```bash
flatpak --user remote-add --if-not-exists flathub \
    https://dl.flathub.org/repo/flathub.flatpakrepo
```

Verify:

```bash
flatpak remotes --user
```

Applications can then be installed for the current user:

```bash
flatpak --user install flathub <application>
```

> **Note:** User-level Flatpak installations are preferred for this personal workstation because they avoid unnecessary system-wide changes. A system-wide Flathub installation is appropriate if multiple users need access to the same applications.

---

## 9. Install Cockpit

Install Cockpit:

```bash
sudo zypper in cockpit
```

Enable the Cockpit socket:

```bash
sudo systemctl enable --now cockpit.socket
```

Verify:

```bash
systemctl status cockpit.socket
```

Cockpit is normally accessed through:

```text
https://localhost:9090
```

---

## 10. Install OPI

Install OPI:

```bash
sudo zypper in opi
```

Install multimedia codecs:

```bash
opi codecs
```

Follow the prompts and select the appropriate repositories/packages when requested.

---

## 11. Firmware Updates

Check the devices supported by `fwupd`:

```bash
fwupdmgr get-devices
```

Check for available firmware updates:

```bash
fwupdmgr get-updates
```

If updates are available:

```bash
sudo fwupdmgr update
```

Check the configured firmware remotes:

```bash
fwupdmgr get-remotes
```

> Firmware updates are optional, but worth checking after a fresh installation, especially on laptops.

---

## 12. Btrfs and Snapper Maintenance

### Check Btrfs usage

Periodically check the filesystem:

```bash
sudo btrfs filesystem usage /
```

### Check Snapper configuration

```bash
sudo snapper get-config
```

Pay particular attention to:

```text
NUMBER_LIMIT
NUMBER_LIMIT_IMPORTANT
TIMELINE_CREATE
TIMELINE_CLEANUP
```

The default Tumbleweed/Snapper configuration is generally sufficient, so avoid changing these values without a specific reason.

### Check snapshots

```bash
sudo snapper list
```

### Run a Btrfs scrub

Occasionally run a scrub to verify filesystem data and metadata:

```bash
sudo btrfs scrub start -Bd /
```

Check the result:

```bash
sudo btrfs scrub status /
```

> **Note:** A scrub verifies the filesystem and can detect data corruption. It is different from a backup.
>
> **Do not routinely run `btrfs balance`.** A regular desktop installation generally does not need weekly or monthly balancing. Only use it when there is a specific reason.

---

## 13. Final System Health Check

After completing the post-installation configuration, perform a basic system check.

### Check openSUSE version

```bash
cat /etc/os-release
```

### Check kernel

```bash
uname -r
```

### Check disk usage

```bash
df -h
```

### Check Btrfs

```bash
sudo btrfs filesystem usage /
```

### Check Snapper snapshots

```bash
sudo snapper list
```

### Check zram/swap

```bash
swapon --show
zramctl
```

### Check failed systemd services

```bash
systemctl --failed
```

### Check high-priority errors from the current boot

```bash
sudo journalctl -p 3 -b
```

---
