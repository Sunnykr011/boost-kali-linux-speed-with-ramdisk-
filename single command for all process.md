sudo bash -c 'cat > /usr/local/bin/ramdisk-manager <<"EOF"
#!/bin/bash
# Auto RAMDisk Manager for Kali Linux (Persistent)

# Config
RAMDISK_SIZE="70%"
MOUNT_POINT="/mnt/ramdisk"
WORKSPACE="$MOUNT_POINT/workspace"
BACKUP_DIR="/opt/ramdisk_backups"
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
