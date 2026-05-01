# Linux Filesystem & Disk Management

> Notes on managing partitions and filesystems on Debian-based systems (Linux Mint / Ubuntu).

## 1. Package management

Use `apt` on Debian-based systems. `yum` / `dnf` are Red Hat only.

```bash
sudo apt update && sudo apt install -y parted
```

## 2. Disk & partition identification

Always identify the correct device before formatting to avoid data loss.

| Command | Purpose |
|---|---|
| `lsblk` | Visual tree of disks and mount points |
| `sudo fdisk -l` | Detailed partition table info |
| `sudo blkid` | Show UUIDs for each partition |

## 3. Creating & refreshing partitions

```bash
sudo partprobe /dev/sdb        # reload partition table without reboot
sudo mkfs.ext4 /dev/sdb1      # format as ext4 (journaled)
sudo mkfs.ext2 /dev/sdb1      # format as ext2 (no journal)
```

> **Note:** No trailing slash on device paths — use `/dev/sdb1`, not `/dev/sdb1/`.

## 4. Tuning the filesystem (tune2fs)

Works on ext2 / ext3 / ext4.

```bash
sudo tune2fs -l /dev/sdb1               # view current settings
sudo tune2fs -m 1 /dev/sdb1             # set reserved space to 1%
sudo tune2fs -L "BackupDrive" /dev/sdb1 # add a label
```

## 5. Mounting drives

```bash
sudo mkdir -p /mnt/storage          # 1. create mount point
sudo mount /dev/sdb1 /mnt/storage   # 2. mount partition
sudo umount /dev/sdb1               # 3. unmount
```

## 6. Common terminal shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+Shift+C` / `Ctrl+Shift+V` | Copy / Paste in terminal |
| `Ctrl+C` | Interrupt current process |
| `Tab` | Auto-complete commands and paths |
| `!!` | Re-run previous command |
| `sudo !!` | Re-run previous command as root |
