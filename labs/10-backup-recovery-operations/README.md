# Backup Repository Preparation and Recovery Readiness

## Objective

Prepare Linux backup storage using an NFS-backed repository, validate backup-agent readiness on client systems, confirm write access to the backup target, and review recovery steps needed when service configuration is missing or unavailable.

## Environment

* Red Hat-based Linux virtual machines
* NFS-backed backup storage
* Linux backup agent
* systemd-managed services
* NFS server and client utilities

## Work Performed

* Prepared a centralized backup storage directory on a Linux server.
* Installed and verified NFS utilities required for backup storage sharing.
* Configured an NFS export for approved Linux backup clients.
* Enabled and restarted NFS services.
* Reloaded NFS export configuration with `exportfs`.
* Created a backup repository directory for Linux client backups.
* Reviewed and corrected ownership and permissions for backup access.
* Installed a Linux backup agent on client systems.
* Verified the backup agent command-line utility was available.
* Tested write access to the backup storage path.
* Reviewed recovery readiness after service configuration loss.
* Recreated required service configuration where restore data was not available.

## Commands and Configuration

Install NFS utilities:

```bash
sudo dnf install -y nfs-utils
```

Create a backup storage directory:

```bash
sudo mkdir -p /srv/backups/linux
sudo chown -R <backup_user>:<backup_group> /srv/backups
sudo chmod 750 /srv/backups
```

Configure an NFS export for approved backup clients:

```bash
sudo vi /etc/exports
```

Example sanitized export:

```exports
/srv/backups <client_ip>(rw,sync)
```

Reload NFS exports:

```bash
sudo exportfs -ra
sudo exportfs -v
```

Enable and restart NFS services:

```bash
sudo systemctl enable --now nfs-server
sudo systemctl restart nfs-server
sudo systemctl status nfs-server --no-pager
```

Install a Linux backup agent on a client system:

```bash
sudo dnf install -y <backup_agent_package>
```

Verify the backup agent CLI is available:

```bash
<backup_agent_cli> version
```

Create a local mount point for backup repository validation:

```bash
sudo mkdir -p /mnt/backup-repo
```

Mount the backup repository from a Linux client:

```bash
sudo mount -t nfs4 <backup_server>:/srv/backups /mnt/backup-repo
```

Test write access to the backup repository:

```bash
touch /mnt/backup-repo/linux/backup-write-test
ls -l /mnt/backup-repo/linux/backup-write-test
rm -f /mnt/backup-repo/linux/backup-write-test
```

Review backup storage mount state:

```bash
df -hT | grep nfs
mount | grep backup
```

## Validation

Confirm NFS utilities are installed:

```bash
rpm -q nfs-utils
```

Expected sanitized result:

```text
nfs-utils-<version>.el9.x86_64
```

Confirm the NFS server is active:

```bash
systemctl is-active nfs-server
```

Expected sanitized result:

```text
active
```

Confirm the backup export is active:

```bash
sudo exportfs -v
```

Expected sanitized result:

```text
/srv/backups <client_ip>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

Confirm the client can mount the backup repository:

```bash
df -hT | grep /mnt/backup-repo
```

Expected sanitized result:

```text
<backup_server>:/srv/backups nfs4 <size> <used> <avail> <use%> /mnt/backup-repo
```

Confirm write access to the backup location:

```bash
touch /mnt/backup-repo/linux/backup-write-test
ls -l /mnt/backup-repo/linux/backup-write-test
```

Expected sanitized result:

```text
-rw-r--r--. 1 <backup_user> <backup_group> 0 <date> backup-write-test
```

Confirm the backup agent command is available:

```bash
<backup_agent_cli> version
```

Expected sanitized result:

```text
<backup_agent_name> <version>
```

## Outcome

A Linux backup repository was prepared using NFS-backed storage and validated from client systems. NFS exports were configured, services were enabled, backup-agent readiness was confirmed, and write access to the repository was tested. Recovery readiness was reviewed by identifying required service configuration and recreating missing configuration where restore data was unavailable.

## Lessons Learned

* Backup storage should be validated before relying on backup jobs or recovery workflows.
* NFS-backed backup repositories require correct exports, service state, ownership, permissions, and client mount validation.
* Backup-agent installation should be verified with both package checks and CLI availability.
* Write-access testing confirms that clients can actually use the backup repository.
* Recovery readiness includes knowing which service configuration files are required to restore operations.
* Backup and recovery workflows should be documented clearly enough to support repeatable restore validation.
