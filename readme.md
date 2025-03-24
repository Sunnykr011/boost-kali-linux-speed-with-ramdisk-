Usage Guide
1. First Run Setup
bash
Copy
chmod +x ramdisk_setup.sh ramdisk_save.sh
sudo ./ramdisk_setup.sh
2. Normal Workflow
Work in ~/ramdisk_workspace for maximum speed

Create manual backups:

bash
Copy
sudo ./ramdisk_save.sh create
List available backups:

bash
Copy
sudo ./ramdisk_save.sh list
Restore a backup:

bash
Copy
sudo ./ramdisk_save.sh restore 20240101_120000
3. Automatic Backups (Recommended)
Add to crontab (sudo crontab -e):

bash
Copy
# Backup every 30 minutes
*/30 * * * * /path/to/ramdisk_save.sh create
Key Features
Safety-First Design:

No security reductions (noexec, nosuid flags)

Doesn't modify existing Kali tools/configs

Minimum memory check prevents system instability

Optimized Performance:

Noatime mount option reduces disk writes

Proper filesystem limits for bug bounty tools

Workspace symlink for easy access

Reliable Backup System:

Atomic rsync operations

Backup rotation to prevent disk filling

Simple restore process

Transparent Logging:

All operations logged to /var/log/ramdisk.log

Clear error messages for troubleshooting

These scripts work together to provide a complete RAMDisk solution while maintaining Kali Linux's security posture and all existing functionality.
