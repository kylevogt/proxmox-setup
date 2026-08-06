### How It Was Setup

Cluster was created from the node `pve` which is my original HP workstation machine.

My old intel gaming PC then joined as `pve-barboa` into the cluster.

To avoid quorum issues since there are only two clusters I decided to setup my Proxmox Backup Server as a corosync QDevice.

### Corosync QDevice setup

On PBS:
```
apt update
apt install corosync-qnetd
systemctl status corosync-qnetd
```

On both PVE nodes:
```
apt update
apt install corosync-qdevice
```

On one of the PVE nodes:
```
pvecm qdevice setup <pbs-ip>
```

From either PVE node to verify expected votes:
```
pvecm status
```

From PBS to verify setup/status:
```
corosync-qnetd-tool -l
```

To remove the QDevice someday, from a cluster node:
```
pvecm qdevice remove
```


