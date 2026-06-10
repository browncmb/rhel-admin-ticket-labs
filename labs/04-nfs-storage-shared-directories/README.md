# NFS Storage and Shared Directory Validation

## Objective

Configure and validate persistent NFS-backed storage for Linux clients, including standardized mount points, NFS client utilities, `/etc/fstab` entries, ownership correction, and shared directory write testing.

## Environment

* Red Hat-based Linux virtual machines
* NFS server
* Linux client systems
* NFSv4 client mounts
* systemd-managed mount handling
* Sanitized hostnames, IP addresses, usernames, paths, and storage targets

## Work Performed

* Created standardized local mount points for NFS-backed directories.
* Verified NFS client utilities were installed on Linux client systems.
* Added persistent NFS mount entries to `/etc/fstab`.
* Reloaded systemd after updating filesystem mount configuration.
* Mounted configured NFS filesystems using `mount -a`.
* Validated NFS mount type, source, and mount location.
* Reviewed ownership and permissions on NFS-backed directories.
* Corrected ownership on a shared user/application directory.
* Created a shared directory for operational file storage.
* Verified write access by creating and removing a test file.

## Commands or Configuration

Install NFS client utilities:

```bash
sudo dnf install -y nfs-utils
```

Create standardized NFS mount points:

```bash
sudo mkdir -p /nfs/incoming/home
sudo mkdir -p /nfs/incoming/vhosts
sudo mkdir -p /nfs/incoming/scripts
```

Example sanitized `/etc/fstab` entries:

```fstab
<nfs_server>:/srv/nfs/home    /nfs/incoming/home    nfs4    defaults,_netdev    0 0
<nfs_server>:/srv/nfs/vhosts  /nfs/incoming/vhosts  nfs4    defaults,_netdev    0 0
<nfs_server>:/srv/nfs/scripts /nfs/incoming/scripts nfs4    defaults,_netdev    0 0
```

Reload systemd after updating `/etc/fstab`:

```bash
sudo systemctl daemon-reload
```

Mount all filesystems defined in `/etc/fstab`:

```bash
sudo mount -a
```

Review mounted NFS filesystems:

```bash
df -hT | grep nfs
mount | grep nfs
```

Correct ownership on an NFS-backed directory:

```bash
sudo chown -R <admin_user>:<admin_group> /nfs/incoming/home/<admin_user>
```

Create and validate a shared operational directory:

```bash
sudo mkdir -p /nfs/incoming/scripts/shared
sudo chown -R <admin_user>:<admin_group> /nfs/incoming/scripts/shared
```

Test write access:

```bash
touch /nfs/incoming/scripts/shared/testfile
ls -l /nfs/incoming/scripts/shared/testfile
rm -f /nfs/incoming/scripts/shared/testfile
```

## Validation

Confirm NFS client packages are installed:

```bash
rpm -q nfs-utils
```

Expected sanitized result:

```text
nfs-utils-<version>.el9.x86_64
```

Confirm NFS filesystems are mounted:

```bash
df -hT | grep nfs
```

Expected sanitized result:

```text
<nfs_server>:/srv/nfs/home     nfs4  <size>  <used>  <avail>  <use%>  /nfs/incoming/home
<nfs_server>:/srv/nfs/vhosts   nfs4  <size>  <used>  <avail>  <use%>  /nfs/incoming/vhosts
<nfs_server>:/srv/nfs/scripts  nfs4  <size>  <used>  <avail>  <use%>  /nfs/incoming/scripts
```

Confirm `/etc/fstab` entries mount cleanly:

```bash
sudo mount -a
echo $?
```

Expected sanitized result:

```text
0
```

Confirm write access to the shared directory:

```bash
touch /nfs/incoming/scripts/shared/testfile
ls -l /nfs/incoming/scripts/shared/testfile
```

Expected sanitized result:

```text
-rw-r--r--. 1 <admin_user> <admin_group> 0 <date> testfile
```

## Outcome

NFS-backed directories were configured for persistent client access and validated through package verification, `/etc/fstab` configuration, systemd reload, mount testing, filesystem inspection, ownership correction, and write-access validation. The resulting configuration provided repeatable access to shared storage across Linux client systems.

## Lessons Learned

* Persistent NFS mounts should be defined carefully in `/etc/fstab` and validated with `mount -a`.
* The `_netdev` mount option helps identify network-backed filesystems during boot.
* `systemctl daemon-reload` should be run after filesystem unit changes are introduced through `/etc/fstab`.
* NFS validation should include mount state, filesystem type, ownership, and write access.
* Ownership and permissions are common causes of NFS access issues even when the mount itself is healthy.
* Shared storage changes should be tested with simple file creation and cleanup before being considered complete.
