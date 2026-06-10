# Static Networking and Host Resolution Validation

## Objective

Configure static networking and host identity on RHEL-based systems, validate routing and gateway connectivity, maintain local host resolution, and confirm SSH reachability after network changes.

## Environment

* Red Hat-based Linux virtual machines
* NetworkManager
* Static IPv4 configuration
* Local hostname configuration
* `/etc/hosts` local name resolution
* SSH connectivity validation

## Work Performed

* Reviewed assigned network information before configuration.
* Created a static NetworkManager connection with an IP address, prefix, gateway, and DNS servers.
* Activated the static network connection.
* Set the system hostname using `hostnamectl`.
* Updated `/etc/hosts` with local host resolution entries.
* Verified network interface address assignment.
* Reviewed default route and subnet route configuration.
* Tested gateway connectivity.
* Validated local hostname resolution.
* Confirmed SSH reachability after static network configuration.

## Commands and Configuration

Review network interfaces:

```bash
ip addr show
```

Review active NetworkManager connections:

```bash
nmcli connection show
```

Create a static NetworkManager connection:

```bash
sudo nmcli connection add \
  type ethernet \
  ifname <interface> \
  con-name static-<interface> \
  ipv4.addresses <server_ip>/<prefix> \
  ipv4.gateway <gateway_ip> \
  ipv4.dns "<dns_server_1> <dns_server_2>" \
  ipv4.method manual \
  autoconnect yes
```

Activate the static connection:

```bash
sudo nmcli connection up static-<interface>
```

Verify assigned IP address:

```bash
ip addr show <interface>
```

Set the system hostname:

```bash
sudo hostnamectl set-hostname <hostname>.example.lab
hostnamectl
```

Update local host resolution:

```bash
sudo vi /etc/hosts
```

Example sanitized `/etc/hosts` entries:

```text
<app_server_ip> app01.example.lab app01
<web_server_ip> web01.example.lab web01
<nfs_server_ip> nfs01.example.lab nfs01
<bastion_ip> bastion01.example.lab bastion01
<control_node_ip> control01.example.lab control01
```

Review routing configuration:

```bash
ip route
```

Test gateway connectivity:

```bash
ping -c 4 <gateway_ip>
```

Validate local hostname resolution:

```bash
getent hosts <hostname>.example.lab
getent hosts app01.example.lab
getent hosts web01.example.lab
```

Validate SSH reachability:

```bash
ssh <admin_user>@<hostname>.example.lab
```

## Validation

Confirm hostname configuration:

```bash
hostnamectl
```

Expected sanitized result:

```text
Static hostname: <hostname>.example.lab
```

Confirm static IP assignment:

```bash
ip addr show <interface>
```

Expected sanitized result:

```text
inet <server_ip>/<prefix> brd <broadcast_ip> scope global <interface>
```

Confirm default route:

```bash
ip route
```

Expected sanitized result:

```text
default via <gateway_ip> dev <interface>
<private_subnet>/<prefix> dev <interface> proto kernel scope link src <server_ip>
```

Confirm gateway connectivity:

```bash
ping -c 4 <gateway_ip>
```

Expected sanitized result:

```text
4 packets transmitted, 4 received, 0% packet loss
```

Confirm local host resolution:

```bash
getent hosts <hostname>.example.lab
```

Expected sanitized result:

```text
<server_ip> <hostname>.example.lab <hostname>
```

Confirm SSH reachability:

```bash
ssh <admin_user>@<hostname>.example.lab
```

Expected sanitized result:

```text
SSH session established successfully using the configured hostname.
```

## Outcome

Static network configuration was applied and validated on RHEL-based systems. Hostnames were configured, local name resolution was maintained, routing and gateway connectivity were confirmed, and SSH reachability was validated using sanitized host entries. The resulting configuration provided stable post-build network identity and access for managed Linux systems.

## Lessons Learned

* Static network configuration should be validated at the interface, route, gateway, and name-resolution layers.
* NetworkManager provides a consistent way to manage persistent Linux network connections.
* Hostname configuration should align with local name resolution and administrative access patterns.
* Local host resolution should be documented and validated when DNS is not the primary source of name resolution.
* SSH validation confirms that network and name-resolution changes support real administrative access.
* Post-build network checks reduce troubleshooting time before higher-level services are configured.
