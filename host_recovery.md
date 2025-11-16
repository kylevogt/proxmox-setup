## Create Backup of important files to PBS

Manually create a one off backup of the proxmox host:
```
proxmox-backup-client backup host-configs.pxar:/etc/pve network-config.pxar:/etc/network --backup-id pve-host-configs --repository pve-nodes@pbs@192.168.1.171:backups
```

### Configuring automated backups

Create `/root/backup-proxmox-configs.sh` and fill in the password

```
#!/bin/bash
#
# Proxmox Host Config Backup Script

export PBS_REPOSITORY=pve-nodes@pbs@192.168.1.171:backups
export PBS_PASSWORD=''

DATE=$(date +%F)

proxmox-backup-client backup \
  host-configs.pxar:/etc/pve \
  network-config.pxar:/etc/network \
  --backup-id pve-host-configs
```

Make that shit executable
```
chmod +x /root/backup-proxmox-configs.sh
```

Open up crontab
```
crontab -e
```

Add line to the crontab (this will run every hour on the 12th minute)
```
12 * * * * /root/backup-proxmox-configs.sh >> /var/log/proxmox-config-backup.log 2>&1
```

## Restore cirtical files from PBS

### Setup PBS client

Configure PBS server/user we're connecting to/with

```
export PBS_REPOSITORY=pve-nodes@pbs@192.168.1.171:backups
```

Login to the PBS server with will request the password and save it
```
proxmox-backup-client login
```

Get a list of which snapshots exist, you'll have to update all future commands to reference the relevant snapshot you want to use
```
proxmox-backup-client snapshot list
```

Pull down files from the host backup
```
proxmox-backup-client restore host/host-configs/2025-11-07T21:21:30Z host-configs.pxar /tmp/restore/pve

proxmox-backup-client restore host/host-configs/2025-11-07T21:21:30Z network-config.pxar /tmp/restore/network 
```

Overwrite all of /etc/network.

Note: This is only safe if you're restoring to the same hardware and networking hardware has not changed. If you're moving to a new machine you'll have to reconfigure networking manually
according to the new hardware. You can reference these files to find the name of bridge you were using so LXCs/VMs pick up the correct interface when restored
```
cp -a /tmp/restore/network/* /etc/network
```

Overwrite /etc/pve/storage.cfg

Note: Similiar to networking stuff, only safe to do this if your storage configuration hasn't changed which is unlikely to be the case if you're restoring your host.

In my case, I was moving from a direct install on a single disk to an install on a ZFS mirror so I had to remove the previous local-lvm storage entry with the new local-zfs that was created during install.
```
cp /tmp/restore/pve/storage.cfg /etc/pve
```

Overwrite /etc/pve/jobs.cfg

This will reconfigure backup jobs you had setup
```
cp /tmp/restore/pve/jobs.cfg /etc/pve
```

Copy over password/credential files for storage

Note: I first attempted copying the entire priv/* folder and this broke things. Unless there's some reason you need everything,
just copy over priv/storage which just contains password files. If you MUST copy over the other priv items I suspect things would have
still worked fine if I had also copied over /tmp/restore/pve/authkey.pub
```
cp /tmp/restore/pve/priv/storage/* /etc/pve/priv/storage
```

Copy over VMs

Note: When I did this I started with just TrueNAS since that's a dependency for some others, then copied over the last few individually
```
cp -a /tmp/restore/pve/qemu-server/* /etc/pve/qemu-server
```

Copy over LXCs
```
cp -a /tmp/restore/pve/lxc/* /etc/pve/lxc
```
