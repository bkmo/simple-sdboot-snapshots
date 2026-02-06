# Simple SD-Boot uki snapshot creation and automatic boot entries in sd-boot menu for Arch Linux.
Boot directly into a previous system state if an update breaks your system.
Snapshots are set to r/w  and will boot without an overlay filesystem. (keep it simple).
**Writable snapshots if modified will no longer reflect their original state.
Read Only snapshots can be configured via sdboot-snaps.conf and require the sd-volatile mkinitcpio hook added.

 BTRFS Snapshot UKI Manager for systemd-boot (Secure Boot Compatible):
   - creates signed Unified Kernel Images (UKIs) for BTRFS snapshots.
   - these UKIs are self-contained and work with Secure Boot when signed.
   - sd-boot menu is automatically populated with snapshot boot entries.
   - CPU Micrcode can be added to the init via config if not using mkinitcpio hook. 

requirements:
   - BTRFS with snapper (snapshots in /.snapshots/N/snapshot format)
   - systemd-boot installed, ESP mounted to /efi (can be changed in /etc/sdboot-snaps.conf)
   - sbctl for Secure Boot signing (or unsigned if SB disabled)
   - systemd-ukify for UKI generation

## Installation

```bash
git clone https://github.com/bkmo/simple-sdboot-snapshots
cd simple-sdboot-snapshots
makepkg -srci
sudo systemctl enable --now snapper-boot-entries.path
sudo pacman -S snap-pac --needed  ##recommended for pre post pacman snapshots
```
Edit /etc/sdboot-snaps.conf to change default options. READONLY,MICROCODE,NOTIFICATION,SNAPSHOTS,EFI
## How It Works

1. **Snapper** creates BTRFS snapshots in `/.snapshots/`
2. **manage-snapshot-ukis** generates signed UKIs for each snapshot (defaults to last 7 (configurable in /etc/sdboot-snaps.conf)
3. **Systemd Service** auto-refreshes UKIs after snapshot updates/deletions
4. **systemd-boot** displays snapshot entries in the boot menu
5. **Desktop notifications** alert when snapshot UKIs are created (can be disabled via .conf)
6. Designed to work well with the snap-pac pacman hook package. (recommended)

## Boot Menu

Depending on your boot order, you will enter your boot menu and see entries like:

```
Arch Linux (linux)                       ← Current system
Arch Linux (linux-lts)                   ← LTS variant
Snapshot #11 [linux] (2025-12-08 14:30)  ← Bootable snapshot
Snapshot #10 [linux] (2025-12-07 10:15)  ← Older bootable snapshot
```

## Booting Into a Snapshot

1. Turn on/reboot your system
2. Select a snapshot entry from the boot menu
3. System boots into that snapshot - fully functional, writable
4. Verify you are in the right snapshot (see below)
5. If all good, make it permanent (see below)

## Verifying You're in a Snapshot

You should get a notification you are booted into a snapshot, if not:
```bash
# Check current root subvolume
btrfs subvolume show /
# Snapshot: Name: @snapshots/10/snapshot
# Normal:   Name: @

# Quick check via kernel cmdline
cat /proc/cmdline | grep -o 'subvol=[^ ]*'
# Snapshot: subvol=@snapshots/10/snapshot
# Normal:   subvol=@
```

## Making a Snapshot Permanent (Rollback)

```bash
# Option 1: Install btrfs-assistant via pacman. (recommended)
sudo pacman -S btrfs-assistant

# Option 2: Use snapper-rollback AUR package
snapper list ## find snapshot number to restore 
sudo snapper-rollback <snapshot_number>

# Option 3: Use snapper rollback (for OpenSuse layout only)
snapper list ## find snapshot number to restore 
snapper rollback <snapshot_number>

# Option 4: Manual approach
# 1. Note snapshot number: cat /proc/cmdline
# 2. Replace current root
sudo btrfs subvolume delete /@
sudo btrfs subvolume snapshot /.snapshots/<N>/snapshot /@
```

## Automatic UKI Refresh

Systemd service regenerates snapshot UKIs when:

- snapshots are created/deleted
- Packages upgraded/installed/removed (requires snap-pac)
- Snapshot UKIs will appear in the systemd-boot menu automatically.
To boot: Hold Space during boot to access systemd-boot menu

Look for entries like:
  Snapshot #N [kernel] (date)

Log: `/var/log/snapshot-uki-refresh.log`

## Manual Snapshot Management

```bash
snapper -c root create -d "pre upgrade"  # Create snapshot
snapper -c root list                     # List snapshots
manage-snapshot-ukis refresh             # Generate bootable UKIs (last 7)
manage-snapshot-ukis refresh 10          # Generate more if space allows
manage-snapshot-ukis space               # Check EFI partition space
manage-snapshot-ukis list                # List bootable snapshot UKIs
manage-snapshot-ukis cleanup             # Remove all snapshot UKIs
```

## Desktop Notifications

| Event           | Message                                      |
| --------------- | -------------------------------------------- |
| UKIs created    | "Created N bootable snapshot entries"        |
| Creation failed | "Failed to create bootable snapshot entries" |

How to disable Notifications: `set NOTIFICATIONS=false in /etc/sdboot-snaps.conf`

**Prerequisites for bootable snapshots:**

- Snapper must be enabled with a root config.
- BTRFS subvolumes setup the "Arch" way. @, @home, @snapshots. (OpenSuse layout also valid)
- EFI/ESP (2GB suggested) mounted to /efi (can be configured via /etc/sdboot-snaps.conf)
- Read Only configurations need the sd-volatile mkinitcpio hook added for an overlay fs.
- systemd-ukify package must be installed.
- If secureboot is enabled, sbctl is needed to sign the uki's (sbctl file database auto-cleaned)

