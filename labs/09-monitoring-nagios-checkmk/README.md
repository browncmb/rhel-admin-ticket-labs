# Monitoring Agent Deployment and Service Health Validation

## Objective

Configure Linux monitoring access, deploy monitoring agents, onboard managed systems into Nagios and CheckMK, perform service discovery, and validate host and service health for operational visibility.

## Environment

* Red Hat-based Linux virtual machines
* Nagios monitoring server
* CheckMK monitoring platform
* Linux managed nodes
* NRPE and Nagios plugins
* CheckMK agent
* systemd-managed services
* Sanitized hostnames, IP addresses, usernames, monitoring URLs, and service output

## Work Performed

* Configured authorized monitoring-console access for administrative review.
* Installed NRPE and Nagios plugins on Linux managed nodes.
* Configured NRPE to allow monitoring checks from the monitoring server.
* Enabled and started the NRPE service.
* Added Linux hosts to the monitoring platform.
* Validated CheckMK agent socket status on managed nodes.
* Performed service discovery for monitored Linux hosts.
* Activated monitoring changes after host onboarding.
* Reviewed system health checks for CPU, memory, filesystems, uptime, and Apache HTTP.
* Validated service-state changes after a controlled service check.
* Confirmed monitored hosts reported expected health status after configuration.

## Commands and Configuration

Install NRPE and Nagios plugins on a managed node:

```bash
sudo dnf install -y nrpe nagios-plugins-all
```

Update the NRPE allowed hosts configuration:

```bash
sudo vi /etc/nagios/nrpe.cfg
```

Example sanitized NRPE configuration:

```conf
allowed_hosts=127.0.0.1,<monitoring_server_ip>
```

Enable and start NRPE:

```bash
sudo systemctl enable --now nrpe
sudo systemctl status nrpe --no-pager
```

Validate that NRPE is listening:

```bash
ss -tulnp | grep nrpe
```

Example Nagios authorization settings:

```conf
authorized_for_all_hosts=<monitor_admin>
authorized_for_all_services=<monitor_admin>
```

Validate CheckMK agent socket status:

```bash
systemctl status check-mk-agent.socket --no-pager
```

Check whether the CheckMK agent is listening:

```bash
ss -tulnp | grep 6556
```

Example CheckMK host onboarding workflow:

```text
Add host: <managed_node>
Assign IP address: <managed_node_ip>
Run service discovery
Accept discovered services
Activate changes
```

Example monitored service categories:

```text
CPU load
Memory usage
Filesystem usage
System uptime
Apache HTTP service
Network reachability
```

## Validation

Confirm NRPE is active:

```bash
systemctl is-active nrpe
```

Expected sanitized result:

```text
active
```

Confirm NRPE starts at boot:

```bash
systemctl is-enabled nrpe
```

Expected sanitized result:

```text
enabled
```

Confirm CheckMK agent socket is active:

```bash
systemctl is-active check-mk-agent.socket
```

Expected sanitized result:

```text
active
```

Confirm the CheckMK agent port is listening:

```bash
ss -tulnp | grep 6556
```

Expected sanitized result:

```text
LISTEN 0 4096 0.0.0.0:6556 0.0.0.0:*
```

Confirm host onboarding and service discovery:

```text
Host added: <managed_node>
Service discovery completed
Changes activated
```

Expected sanitized monitoring status:

```text
Host state: UP
Service state: OK
Critical Linux services reporting expected status
```

Validate service-state visibility after a controlled service check:

```bash
sudo systemctl status httpd --no-pager
```

Expected sanitized monitoring result:

```text
Apache HTTP service visible in monitoring platform
Service state reflects current system status
```

## Outcome

Linux monitoring was configured and validated across managed systems using Nagios and CheckMK. Monitoring agents were installed, allowed monitoring sources were configured, services were enabled, hosts were onboarded, service discovery was completed, and key health checks were validated. The resulting monitoring setup provided visibility into host availability, system resources, and service status.

## Lessons Learned

* Monitoring should include both host availability and service-level health checks.
* Agent configuration must allow checks only from approved monitoring systems.
* Service discovery helps confirm what the monitoring platform can observe on each host.
* Monitoring changes should be activated and validated after host onboarding.
* systemd status checks are useful for confirming monitoring agent availability.
* Controlled service-state validation helps confirm that monitoring reflects real system conditions.
