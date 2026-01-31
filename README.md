# Simple SD-Boot uki snapshot creation and automatic boot entries in sd-boot menu

Boot directly into a previous system state if an update breaks your system.

## How It Works

1. **Snapper** creates BTRFS snapshots in `/.snapshots/`
2. **manage-snapshot-ukis** generates signed UKIs for each snapshot (defaults to 7)
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
# Option 1: Use snapper-rollback AUR package (recommended)
snapper-rollback <snapshot_number>

# Option 2: Manual approach
# 1. Note snapshot number: cat /proc/cmdline
# 2. Reboot into normal system
# 3. Replace current root
sudo btrfs subvolume delete /@
sudo btrfs subvolume snapshot /.snapshots/<N>/snapshot /@
```

## Automatic UKI Refresh

Systemd service regenerates snapshot UKIs when:

- snapshots are created/deleted
- Kernel packages upgraded
- Microcode updated
- mkinitcpio presets change

Log: `/var/log/snapshot-uki-refresh.log`

## Manual Snapshot Management

```bash
snapper -c root create -d "Before upgrade"  # Create snapshot
snapper -c root list                        # List snapshots
manage-snapshot-ukis refresh                # Generate bootable UKIs (last 7)
manage-snapshot-ukis refresh 10             # Generate more if space allows
manage-snapshot-ukis space                  # Check EFI partition space
manage-snapshot-ukis list                   # List bootable snapshot UKIs
manage-snapshot-ukis cleanup                # Remove all snapshot UKIs
```

## Desktop Notifications

| Event           | Message                                      |
| --------------- | -------------------------------------------- |
| UKIs created    | "Created N bootable snapshot entries"        |
| Creation failed | "Failed to create bootable snapshot entries" |

How to disable Notifications: `set NOTIFICATIONS=false in /etc/sdboot-snaps.conf`

**Prerequisites for bootable snapshots:**

- Snapper must be enabled with a root config.
- BTRFS subvolumes setup the "Arch" way. @, @home, @snapshots.
- EFI/ESP (2GB suggested) mounted to /efi with sd-boot installed to /efi.
- Boot cmdline must be read-write.
- Ukify package must be installed.
- If secureboot is enabled, sbctl is needed to sign the uki's (sbctl file database auto-cleaned)

