# Service Recovery and Performance Troubleshooting

## Objective

Investigate service-impact and performance-related issues on RHEL-based systems, restore required NFS and Apache configuration, validate network connectivity, and confirm service health through system checks and monitoring review.

## Environment

* Red Hat-based Linux virtual machines
* NFS server and client configuration
* Apache HTTP Server
* systemd-managed services
* Network interface and routing validation
* Monitoring platform for service and CPU visibility

## Work Performed

* Reviewed an operational script after service-impact and performance reports.
* Identified missing or unavailable NFS and Apache configuration components.
* Recreated required NFS export configuration using sanitized export paths.
* Reloaded NFS exports and restarted NFS services.
* Restored Apache service readiness where configuration was missing.
* Enabled and restarted Apache after recovery steps.
* Validated network interface state, routing, and gateway connectivity.
* Confirmed NFS and Apache service status with systemd.
* Verified web service response using `curl`.
* Reviewed monitoring data to confirm CPU and service state returned to expected behavior.

## Commands and Configuration

Review system resource usage during investigation:

```bash
uptime
free -h
df -hT
```

Check active network interfaces:

```bash
ip addr show
```

Review routing configuration:

```bash
ip route
```

Test gateway or upstream connectivity:

```bash
ping -c 4 <gateway_ip>
```

Create a sanitized NFS export configuration:

```bash
sudo vi /etc/exports
```

Example sanitized export:

```exports
/srv/shared <client_ip>(rw,sync)
```

Reload NFS exports:

```bash
sudo exportfs -ra
sudo exportfs -v
```

Restart and validate NFS services:

```bash
sudo systemctl restart nfs-server
sudo systemctl status nfs-server --no-pager
```

Install or restore Apache package availability where required:

```bash
sudo dnf install -y httpd
```

Enable and restart Apache:

```bash
sudo systemctl enable --now httpd
sudo systemctl restart httpd
sudo systemctl status httpd --no-pager
```

Create basic web validation content if the document root is empty:

```bash
echo "Service recovery validation page" | sudo tee /var/www/html/index.html
```

Validate local Apache response:

```bash
curl http://localhost/
```

Validate remote Apache response:

```bash
curl http://<web_server>/
```

## Validation

Confirm NFS exports are active:

```bash
sudo exportfs -v
```

Expected sanitized result:

```text
/srv/shared <client_ip>(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,root_squash,no_all_squash)
```

Confirm NFS service is active:

```bash
systemctl is-active nfs-server
```

Expected sanitized result:

```text
active
```

Confirm Apache service is active:

```bash
systemctl is-active httpd
```

Expected sanitized result:

```text
active
```

Confirm Apache is enabled at boot:

```bash
systemctl is-enabled httpd
```

Expected sanitized result:

```text
enabled
```

Confirm local web response:

```bash
curl http://localhost/
```

Expected sanitized result:

```text
Service recovery validation page
```

Confirm network route is present:

```bash
ip route
```

Expected sanitized result:

```text
default via <gateway_ip> dev <interface>
```

Confirm monitoring reflects recovered service state:

```text
Host state: UP
Service state: OK
CPU utilization returned to expected range
```

## Outcome

Service-impact and performance-related issues were reviewed and remediated by restoring required NFS and Apache configuration, validating network connectivity, restarting affected services, and confirming recovery through systemd, NFS export checks, HTTP response testing, and monitoring review. The recovery process restored expected service availability and provided a repeatable validation path for similar incidents.

## Lessons Learned

* Troubleshooting should start with impact review, recent changes, and current service state.
* Operational scripts should be reviewed carefully when service configuration or application files are unexpectedly missing.
* NFS recovery requires export configuration, service health, and client access validation.
* Apache recovery should include package availability, service state, document-root validation, and HTTP response testing.
* Network checks should include interface state, routes, gateway reachability, and service-level testing.
* Monitoring data is most useful when paired with direct system validation from the affected host.
