Here's the **simplest one-command solution** to make your RAMDisk setup fully automatic and persistent across reboots in Kali Linux:

### **1. Combined Automatic Setup (Paste this single command):**

```bash
sudo bash -c 'cat > /usr/local/bin/ramdisk-manager <<"EOF"
#!/bin/bash
# Auto RAMDisk Manager for Kali Linux (Persistent)

# Config
RAMDISK_SIZE="70%"
MOUNT_POINT="/mnt/ramdisk"
WORKSPACE="$MOUNT_POINT/workspace"
BACKUP_DIR="/opt/ramdisk_backups"
MAX_BACKUPS=5
LOG_FILE="/var/log/ramdisk.log"

# Functions
log() { echo "[$(date "+%Y-%m-%d %H:%M:%S")] $1" | tee -a "$LOG_FILE"; }

ramdisk_setup() {
    mkdir -p "$MOUNT_POINT" "$BACKUP_DIR"
    mount -t tmpfs -o "size=$RAMDISK_SIZE,noatime,nodev,nosuid" tmpfs "$MOUNT_POINT"
    mkdir -p "$WORKSPACE" && chmod 700 "$WORKSPACE"
    ln -sf "$WORKSPACE" ~/ramdisk_workspace
    log "RAMDisk initialized"
}

auto_backup() {
    rsync -aq --delete "$WORKSPACE/" "$BACKUP_DIR/latest/"
    log "Auto-backup completed"
}

auto_restore() {
    if [ -d "$BACKUP_DIR/latest" ]; then
        rsync -aq --delete "$BACKUP_DIR/latest/" "$WORKSPACE/"
        log "Auto-restore completed"
    fi
}

case "$1" in
    install)
        # Install systemd services
        cat > /etc/systemd/system/ramdisk.service <<EOL
[Unit]
Description=RAMDisk Manager
After=network.target

[Service]
ExecStart=/usr/local/bin/ramdisk-manager startup
Restart=no
Type=oneshot

[Install]
WantedBy=multi-user.target
EOL

        cat > /etc/systemd/system/ramdisk-shutdown.service <<EOL
[Unit]
Description=RAMDisk Backup on Shutdown
DefaultDependencies=no
Before=shutdown.target reboot.target halt.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/ramdisk-manager backup

[Install]
WantedBy=halt.target reboot.target shutdown.target
EOL

        systemctl enable ramdisk.service ramdisk-shutdown.service
        log "Services installed - RAMDisk will auto-start on boot"
        ;;
    startup)
        ramdisk_setup
        auto_restore
        ;;
    backup)
        auto_backup
        ;;
    *)
        echo "Usage: sudo ramdisk-manager install"
        ;;
esac
EOF

chmod +x /usr/local/bin/ramdisk-manager
/usr/local/bin/ramdisk-manager install'
```

### **2. Usage:**
After running the command above:
- **RAMDisk auto-creates** on every boot
- **Auto-backups** happen on shutdown/reboot
- **Auto-restores** happen on startup
- Access your fast workspace at: `~/ramdisk_workspace`

### **Key Features:**
1. **Fully Automatic** - No manual intervention needed after setup
2. **Persistent Across Reboots** - Backups restore automatically
3. **Clean Implementation** - Uses systemd for reliability
4. **Logging** - Check `/var/log/ramdisk.log` for status

### **To Uninstall:**
```bash
sudo systemctl disable ramdisk.service ramdisk-shutdown.service
sudo rm /etc/systemd/system/ramdisk* /usr/local/bin/ramdisk-manager
sudo umount /mnt/ramdisk 2>/dev/null
```

This maintains all the functionality of your original scripts while making it completely automatic with a single setup command. The RAMDisk will now persist seamlessly across reboots.
