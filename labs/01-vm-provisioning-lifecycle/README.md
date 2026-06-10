# VM Provisioning and Post-Build Validation

## Objective

Provision a Red Hat-based Linux virtual machine in a vSphere environment, place the workload into an appropriate resource pool, create rollback checkpoints, complete baseline operating system configuration, and validate the system after deployment.

## Environment

* VMware vSphere-based virtualization
* Red Hat-based Linux virtual machines
* Resource pool-based workload organization
* VM snapshot checkpoints
* Kickstart-assisted operating system installation
* Static network configuration
* Local administrative user configuration
* NFS-backed storage validation
* Asset inventory tracking

## Work Performed

* Created a resource pool for Linux lab workloads.
* Migrated an existing virtual machine into the resource pool.
* Provisioned a new Linux virtual machine with assigned CPU, memory, disk, and network resources.
* Used thin-provisioned storage for the virtual disk.
* Booted the Linux installer with Kickstart parameters.
* Applied baseline hostname and network configuration.
* Created a local administrative user with sudo access.
* Created VM snapshots at key lifecycle points to support rollback before major configuration changes.
* Validated local name resolution after provisioning.
* Verified network connectivity and system reachability.
* Confirmed NFS mount readiness where required.
* Collected VM identity metadata for inventory tracking.
* Recorded the provisioned system in an asset inventory.
* Validated CPU, memory, disk, network, operating system, and service readiness after build.

## Commands and Configuration

Example Kickstart boot parameters:

```text
inst.ks=http://<kickstart_server>/ks/rhel9-server.cfg ip=<server_ip>::<gateway_ip>:<netmask>::<interface>:none nameserver=<dns_server>
```

Set the system hostname:

```bash
sudo hostnamectl set-hostname <hostname>.example.lab
hostnamectl
```

Review network interface state:

```bash
ip addr show
```

Review default route:

```bash
ip route
```

Test gateway connectivity:

```bash
ping -c 4 <gateway_ip>
```

Test DNS or local name resolution:

```bash
getent hosts <hostname>.example.lab
```

Create a local administrative user:

```bash
sudo useradd <admin_user>
sudo passwd <admin_user>
sudo usermod -aG wheel <admin_user>
```

Validate sudo group membership:

```bash
id <admin_user>
```

Review CPU and memory allocation from the guest:

```bash
lscpu
free -h
```

Review disk layout:

```bash
lsblk
df -hT
```

Validate operating system release:

```bash
cat /etc/os-release
```

Check NFS client utilities where shared storage is required:

```bash
rpm -q nfs-utils
```

Validate mounted filesystems:

```bash
df -hT
mount | grep nfs
```

Snapshot checkpoints used during the VM lifecycle:

```text
pre-change-checkpoint
post-install-baseline
rollback-point
```

Collect VM identity metadata for inventory tracking:

```bash
sudo dmidecode -s system-serial-number
hostnamectl
ip addr show
```

## Validation

Confirm the hostname is set correctly:

```bash
hostnamectl
```

Expected sanitized result:

```text
Static hostname: <hostname>.example.lab
Operating System: Red Hat-based Linux
```

Confirm network routing is present:

```bash
ip route
```

Expected sanitized result:

```text
default via <gateway_ip> dev <interface>
```

Confirm gateway connectivity:

```bash
ping -c 4 <gateway_ip>
```

Expected sanitized result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Confirm local name resolution:

```bash
getent hosts <hostname>.example.lab
```

Expected sanitized result:

```text
<server_ip> <hostname>.example.lab <hostname>
```

Confirm administrative user sudo eligibility:

```bash
id <admin_user>
```

Expected sanitized result:

```text
uid=<uid>(<admin_user>) gid=<gid>(<admin_user>) groups=<gid>(<admin_user>),10(wheel)
```

Confirm disk and filesystem visibility:

```bash
lsblk
df -hT
```

Expected sanitized result:

```text
Configured virtual disk and mounted filesystems visible from the guest operating system.
```

Confirm NFS-backed storage readiness where applicable:

```bash
df -hT | grep nfs
```

Expected sanitized result:

```text
<nfs_server>:/<export_path> nfs4 <size> <used> <avail> <use%> <mount_point>
```

Confirm snapshot checkpoints were created:

```text
Snapshot present: pre-change-checkpoint
Snapshot present: post-install-baseline
Snapshot present: rollback-point
```

Confirm asset inventory fields were recorded:

```text
Hostname: <hostname>.example.lab
IP address: <server_ip>
CPU: <vcpu_count>
Memory: <memory_size>
Disk: <disk_size>
Operating system: Red Hat-based Linux
Serial: <vm_serial>
```

## Outcome

A Red Hat-based Linux virtual machine was provisioned in vSphere, placed into the appropriate resource pool, deployed with baseline operating system settings, protected with snapshot checkpoints, and validated after build. Post-build checks confirmed hostname configuration, network reachability, administrative access, disk visibility, optional NFS readiness, snapshot availability, and inventory metadata collection.

## Lessons Learned

* VM provisioning should include both infrastructure placement and guest operating system validation.
* Resource pools help organize workloads and align virtual machines with capacity planning.
* Kickstart-assisted installation improves consistency during repeated Linux builds.
* Snapshots provide rollback points during provisioning and should be created before major configuration changes.
* Post-build validation should include hostname, network, storage, user access, and OS checks.
* Asset inventory should be updated after provisioning so systems can be tracked consistently.
* A provisioned VM should not be considered complete until connectivity, access, storage, snapshots, and inventory data are verified.
