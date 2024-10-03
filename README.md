# proxmox-setup
The steps I took to setup my proxmox cluster

## Node Configurations

### Enable VLAN awareness on networking bridge
Networking > vmbr0 > VLAN Aware: Yes

### Post Install Setup Script
https://tteck.github.io/Proxmox/#proxmox-ve-post-install

The most important bit of this is to disable the enterprise repository and add the no-subscription repository.

Updates > Repositories
- Disable both enterprise repositories
- Add the no-subscription repository

### Gmail Setup
Generally I followed this guide: https://technotim.live/posts/proxmox-alerts/

I did not bother to override the header for sent emails. Proxmox seems to have improved how the header is set since that guide/video.

## Datacenter Configurations

### Connect SMB drive for backups, etc
Storage > Add SMB/CIFS

#### General
```
ID: truenas
Server: 192.168.9.62
Username: proxmox
Password: ********
Share: proxmox
```

```
Nodes: all
Enable: Yes
Content: Disk image, VZDump backup file, container template, ISO Image
Domain:
Subdirectory:
```

#### Backup Retention
```
Keep all backups: No

Keep Last: -
Keep Daily: 5
Keep Monthly: 3

Keep Hourly: -
Keep Weekly 4
Keep Yearly: 2

Maximum Protected: -
```

### Enable Backups
Backup > Add

#### General
```
Node: All
Storage: truenas
Schedule: 21:00
Selection mode: All

Notification mode: Default
Send email to: *email*
Send email: Always
Compression: ZSTD
Mode: Snapshot
Enable: Yes
```

#### Retention
```
Leave this empty and settings from SMB (or any other storage being used) will be the default)
```

#### Node Template
Backup Notes: {{cluster}}, {{guestname}}, {{node}}, {{vmid}}

#### Advanced
I didn't change anything here

## Issues I fixed

### failed to start zfs-import-scan.service error at boot

Because TrueNAS has disks passed through directly to it the PVE Node seems to have been getting confused and the import scan service would fail to start at boot.
`systemctl disable --now zfs-import-scan.service`
